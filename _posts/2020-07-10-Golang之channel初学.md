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
