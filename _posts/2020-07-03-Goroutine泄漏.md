---
layout: post
date: 2020-07-03
title: Goroutine泄漏
author: tux
tags: golang
---

### 什么是goroutine泄漏

在Golang中，每一个并发执行的活动称为goroutine。当使用goroutine(独立的活动)和channel(用于goroutine间通信)进行并发编程时，开发人员需要小心的处理goroutine，以防goroutine泄漏。goroutine泄漏基本属于内存泄漏的一种，它会永久占用分配给它的内存，属于一个bug。不同于变量的垃圾回收，泄漏的goroutine不会自动回收。

### 产生原因

- goroutine由于channel的读/写端退出而一直阻塞，导致goroutine一直占用资源，而无法退出

- goroutine进入死循环中，导致资源一直无法释放

- nil channels

### 解决办法

#### 场景一:向无接受者的无缓冲通道发送消息

代码如下：

```go
package main
import (
    "fmt"
    "math/rand"
    "runtime"
    "time"
)
func query() int {
    n := rand.Intn(100)
    time.Sleep(time.Duration(n) * time.Millisecond)
    return n
}
func queryAll() int {
    ch := make(chan int)
    go func() { ch <- query() }()
    go func() { ch <- query() }()
    go func() { ch <- query() }()
    return <-ch
}
func main() {
    for i := 0; i < 4; i++ {
        queryAll()
        fmt.Printf("#goroutines: %d\n", runtime.NumGoroutine())
    }
}
#goroutines: 3
#goroutines: 5
#goroutines: 7
#goroutines: 9
```

上述代码的问题在于，在接受第一个响应的值后，后面响应慢的goroutine仍然会想无缓冲通道中发送值，但此时另一端已经没有接受者了。对于这种情况，如果事先知道goroutine的数量，可以使用容量为goroutine数量的缓冲通道。比如：

```go
ch := make(ch int, 3)
```

或者是只要还有goroutine在运行，就使用一个接受者在通道中接受消息。

另一种可选的办法，是使用一种机制，通过`context`取消其他的请求：

代码源于：[Using contexts to avoid leaking goroutines][https://rakyll.org/leakingctx/]

假设有一个函数，内部起了一个goroutine。这个函数一旦被调用，调用者是无法终止此函数内启动的goroutine的。代码如下：

```go
// gen is a broken generator that will leak a goroutine.
func gen() <-chan int {
	ch := make(chan int)
	go func() {
		var n int
		for {
			ch <- n
			n++
		}
	}()
	return ch
}
```
上面的generator启动了一个带有无限循环的goroutine，但是调用者值消费了前5个，代码如下：

```go
// The call site of gen doesn't have a 
for n := range gen() {
    fmt.Println(n)
    if n == 5 {
        break
    }
}
```
调用者在break后结束，但goroutine还在无限循环，这样的代码就存在goroutine的泄漏。
这里可以通过直接关闭通道，来给goroutine发一个通知，但更好的办法是：使用cancellable context.The generator can select on a context’s Done channel and once the context is done, the internal goroutine can be cancelled.代码如下：

```go
// gen is a generator that can be cancellable by cancelling the ctx.
func gen(ctx context.Context) <-chan int {
	ch := make(chan int)
	go func() {
		var n int
		for {
			select {
			case <-ctx.Done():
				return // avoid leaking of this goroutine when ctx is done.
			case ch <- n:
				n++
			}
		}
	}()
	return ch
}
```
现在调用这可以在消费结束后通知generator，一旦调用cancel函数，内部的goroutine将会返回：
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel() // make sure all paths cancel the context to avoid context leak

for n := range gen(ctx) {
    fmt.Println(n)
    if n == 5 {
        cancel()
        break
    }
}

// ...
```
**使用另一个通道，在主线程结束后进行通知的方法**：

```go
func main() {
	newRandStream := func(done <-chan interface{}) <-chan int {
		randStream := make(chan int)

		go func() {
			defer fmt.Println("newRandStream closure exited.")
			defer close(randStream)

			for {
				select {
				case randStream <- rand.Int():
				case <-done:  // 得到通知，结束自己
					return
				}
			}
		}()

		return randStream
	}


	done := make(chan interface{})
	randStream := newRandStream(done)
	fmt.Println("3 random ints:")

	for i := 1; i <= 3; i++ {
		fmt.Printf("%d: %d\n", i, <-randStream)
	}

    // 通知子协程结束自己
    // done <- struct{}{}
	close(done)
	// Simulate ongoing work
	time.Sleep(1 * time.Second)
}
```
上面的代码中，协程通过一个channel来得到结束的通知，这样它就可以清理现场。防止协程泄露。 通知协程结束的方式，可以是发送一个空的struct，更加简单的方式是直接close channel。

#### 场景二：从没有发送者的无缓冲通道中接受

这个场景与场景一相似，[代码示例][https://www.openmymind.net/Leaking-Goroutines/]

#### 场景三：nil channels

对nil channels进行读写会永久阻塞，造成daedlock:

```go
package main

func main() {
    var ch chan struct{}
    ch <- struct{}{} // 写入
    <-ch // 读取
}
```

这种场景一般发生在没有对通道进行初始化的情况：

```go
package main
import (
    "fmt"
    "runtime"
    "time"
)
func main() {
    var ch chan int
    if false {
        ch = make(chan int, 1)
        ch <- 1
    }
    go func(ch chan int) {
        <-ch
    }(ch)
    c := time.Tick(1 * time.Second)
    for range c {
        fmt.Printf("#goroutines: %d\n", runtime.NumGoroutine())
    }
}
```
上面示例代码的问题在于`if false {`，在大型程序中，很容易忘记不能使用通道的零值，即nil

#### 场景四：无限循环

这种情况下，goroutine的泄漏不是由于错误的使用通道，而是阻塞在I/O操作中，如在没有设置超时的情况下，对API server发送请求。另一种原因是程序单纯的陷入了无限循环。

### 分析方法

#### 1 runtime.NumGoroutine

最简单的方法就是使用runtime.NumGoroutine返回的goroutine数量

#### 2 net/http/pprof

```go
import (
    "log"
    "net/http"
    _ "net/http/pprof"
)
...
log.Println(http.ListenAndServe("localhost:6060", nil))
```
调用 http://localhost:6060/debug/pprof/goroutine?debug=1 ，将会返回带有堆栈跟踪的 goroutine 列表.

#### 3 runtime/pprof

打印当前存在的goroutine栈跟踪信息到标准输出

```go
import (
    "os"
    "runtime/pprof"
)
...
pprof.Lookup("goroutine").WriteTo(os.Stdout, 1)
```

#### 4 gops

```go
import "github.com/google/gops/agent"
...
if err := agent.Start(); err != nil {
    log.Fatal(err)
}
time.Sleep(time.Hour)
> ./bin/gops
12365   gops    (/Users/mlowicki/projects/golang/spec/bin/gops)
12336*  lab     (/Users/mlowicki/projects/golang/spec/bin/lab)
> ./bin/gops vitals -p=12336
goroutines: 14
OS threads: 9
GOMAXPROCS: 4
num CPU: 4
```

#### 5 leaktest

在测试中自动检测泄漏的方法之一。它主要是从测试开始到测试结束，使用`runtime.Stack`得到活跃goroutine的栈跟踪信息。如果在测试结束后，还存在新的goroutine，那就可以将其归类为泄漏。

分析甚至已经在运行的程序的 goroutine 管理，以避免可能会导致内存不足的泄露，这非常重要。因为这种问题通常会在代码在生产运行数日后才出现，这回带来真正的损害，


### Reference

1. https://medium.com/golangspec/goroutine-leak-400063aef468
2. https://hoverzheng.github.io/post/technology-blog/go/goroutine-leak%E5%92%8C%E8%A7%A3%E5%86%B3%E4%B9%8B%E9%81%93/