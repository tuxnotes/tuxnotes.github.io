---
layout: post
date: 2020-07-10
title: Golang之channel初学
author: tux
tags: golang,channel
---

### 初学golang并发编程之channel

首先需要拜读的就是golang官网中的`effective go`中的内容，链接如下：https://golang.org/doc/effective_go.html#concurrency

#### Sharing by communication

简单讲就是通过在goroutine之间来回传递数据来实现，而数据的传递通过通道。

#### Goroutines

Goroutine在golang中是一个执行的实体，可以是一个函数或者方法。其简化的模型为：一个函数与其他goroutine在相同的地址空间并发执行。这些goroutine可以运行在多个os级别的线程中，
这意味着如果一个发生了阻塞，如等待io，其他的goroutine可以继续执行。带有`go`关键字的函数或方法调用，都会启动一个新的goroutine，并在这个goroutine中执行这个函数或方法。
当调用完成后，goroutine也就退出了。

#### channels

对于channel首先要明确的是，**channel是用于goroutine之间的通信，也就是说channel连接的必须是goroutine，而不是能变量或常量**.
无缓冲通道用于同步，无缓冲通道两端的sender和receiver同生共死：当sender发送时，如果receiver没有准备好，那sender就会被阻塞。
如果receiver根本不会出现，那就出现deadlock。同样，如果sender没准备好，reveiver就会被阻塞。如果根本没有sender同样会方法deadlock.
如果有多个sender向非缓冲通道中发送数据，在某个时刻确定不再向通道发送数据时，要关闭通道，这样receiver会收到通知也退出，否则不通知就
会deadlock。并且还要保证已经发送的goroutine都被接受完毕，否则就会goroutine泄漏。无传统通道的简单示例如下：
```go
package main

import (
  "fmt"
  "math/rand"
  "time"
)

func main() {
  c := make(chan int)
  go func() {
    rand.Seed(time.Now().UnixNano())
    for i := 0; i < rand.Intn(100); i++ {
        c <- i
    }
    close(c) // 当发送有限数据时，一定要记得关闭通道
  }()
  for i := range c {
      fmt.Println(i) // 这里相当于是在main goroutine中接受
  }
  fmt.Println("Done")
}
```
另一种接受的方法是，使用无限循环，代码片段如下：

```go
for {
        if data, ok := <-c; ok {
            fmt.Println(data)
        } else {
            break
        }
    }
```

##### 缓冲通道

缓冲通道可用作信号(semaphore),比如限制通量，限流。缓冲通道有容量限制，在到达容量前，goroutine不会阻塞。当达到容量后，goroutine会阻塞，直到缓冲通道中的内容被消费，通道中有剩余空间的时候，才能继续向通道中发送数据。在下面的例子中，进来的请求会发到`handle`函数进行处理，`handle`函数会先发送数据到缓冲通道中，然后处理请求，请求处理完成后，在从缓冲通道中读取数据，这相当于给下一个consumer发送了一个信号，缓冲通道的容量限制了同时调用handle进行处理的数量。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
  sem <- 1 // Wait active queue to drain
  process(r) // take a long time
  <- sem // Done; enable next request to run
}

func Serve(queue chan *Request) {
    for {
        req := <- queue
        handle(req) // Don't wait handle to finish
    }
}
```
但上面的设计有个问题，`Serve`会为每个进来的请求创建一个新的goroutine，如果请求数量巨大的话，`Serve`会消耗无限的资源。我们可以让`Serve`控制goroutine的创建。下面是一个解决方案，但是有bug，接下来会介绍如何debug:

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```
这里的bug是for循环，循环中的变量在每次迭代中会被复用，因此`req`变量在所有的goroutine中是共享的。这并不是我们想要的，我们需要的是在每个goroutine中的`req`都是唯一的。下面的解决办法是将`req`的值作为参数传给goroutine的闭包。
```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```
这个版本与前一个版本相比，注意闭包的声明和运行的不同，另一种解决办法是：用相同的变量名创建一个新的变量。
```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```
虽然`req := req`这种写法看起来很奇怪，但在go中这种写法是合法的，也是惯用法。

回到`server`的主要问题，另一种管理资源的办法是：从request channel中读取所有的请求，启动固定数量的`handle` goroutine，goroutine的数量控制同时调用`process`的数量。`Server`函数也接受一个通道作为参数，用来通知退出。goroutine启动之后会阻塞住从quit通道接受数据：
```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```
#### channels of channels

Go的重要特性之一是，channel是一等公民，它可以作为变量，也可以作为函数的参数。**A common use of this property is to implement safe, parallel demultiplexing.**
在前面的示例中，`hadnle`函数是一个理想化的处理器，我们没有定义其处理对象的类型。如果处理对象的类型包含一个用于reply的channel，那么每个客户端请求都能提供独立的用于回答的路径。
下面是Request类型的定义：

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```
客户端提供了一个函数，它包含的参数以及一个通道，用于接受回答。
```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```
在服务端，处理器函数只需做如下更改：
```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```
There's clearly a lot more to do to make it realistic, **but this code is a framework for a rate-limited, parallel, non-blocking RPC system, and there's not a mutex in sight**.

