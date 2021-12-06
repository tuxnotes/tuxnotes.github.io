---
layout: post
title: Golang concurrency控制
author: tux
date: 2021-12-06
tags: Golang
---

# 1 介绍

Golang吸引人的原因有很多，其中最重要的一点就是在语言层面支持并发。通过简单的`go`关键字就可以将任务丢到后台运行。但是如何有效的控制concurrency，通常有以下三种方式：

- WaitGroup
- channel
- context

# 2 WaitGroup

先来了解一下什么场景下需要使用WaitGroup。假设有两台机器需要同时上传最新的代码，两台机器分别上传完成后，才能执行最后的重启步骤。就是把一个job同时拆成好几份，同时一起做，可以减少很多时间，但是最后需要等到全部做完才能执行下一步。这种场景下就需要使用WaitGroup了。即多个goroutine之间不需要相互交流，但需要所有的goroutine完成后才能进行下一步的处理。示例代码如下
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)

    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {

    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)

        i := i

        go func() {
            defer wg.Done()
            worker(i)
        }()
    }

    wg.Wait()

}
```
# 3 Channel

另一个实际案例就是，我们需要主动通知一个goroutine停止运行。例如，当app启动时，会在后台跑一些监控程序。当整个app需要停止的时候，在停止前需要发送通知给后台运行的监控程序，先将其停止，此时就需要用到Channel来通知。示例如下
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    exit := make(chan bool)
    go func() {
        for {
            select {
            case <-exit:
                fmt.Println("Exit")
                return
            case <-time.After(2 * time.Second):
                fmt.Println("Monitoring")
            }
        }
    }()
    time.Sleep(5 * time.Second)
    fmt.Println("Notify Exit")
    exit <- true //keep main goroutine alive
    time.Sleep(5 * time.Second)
}
```
上面的示例中，后台用了一个goroutine及一个channel来控制。可以想象当后台有无数个goroutine的时候，就需要使用多个channel才能进行控制。同时goroutine中可能又会产生goroutine。这种场景下午就单纯使用channel来控制多级goroutine了。可以通过`context`解决。

# 3 Context

假设有这样一个场景:有一个后台任务A，A任务又产生了B任务，B任务又产生了C任务。假设中途需要停止A任务，而A又必须告诉B及C一起停止，此时使用context是最方便的。
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func foo(ctx context.Context, name string) {
    go bar(ctx, name) // A calls B
    for {
        select {
        case <-ctx.Done():
            fmt.Println(name, "A Exit")
            return
        case <-time.After(1 * time.Second):
            fmt.Println(name, "A do something")
        }
    }
}

func bar(ctx context.Context, name string) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println(name, "B Exit")
            return
        case <-time.After(2 * time.Second):
            fmt.Println(name, "B do something")
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go foo(ctx, "FooBar")
    fmt.Println("client release connection, need to notify A, B exit")
    time.Sleep(5 * time.Second)
    cancel() //mock client exit, and pass the signal, ctx.Done() gets the signal  time.Sleep(3 * time.Second)
    time.Sleep(3 * time.Second)
}
```
可以把context当做一个controller，可以随时控制不确定个数的goroutine。由上往下，主要宣告context.WithCancel后，在任意时间点都可以透过cancel()停止后台服务。

更形象的示例如下：
```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func hello(w http.ResponseWriter, req *http.Request) {

    ctx := req.Context() // net/http包为每个request使用Context方式创建一个context.Context
    fmt.Println("server: hello handler started")
    defer fmt.Println("server: hello handler ended")

    select {
    case <-time.After(10 * time.Second): // 返回客户端之前等待10秒，用于模拟服务在处理请求。在等待处理请求的过程中，注意观察context的Done()通道
        fmt.Fprintf(w, "hello\n")
    case <-ctx.Done():

        err := ctx.Err() // context的Err()方法返回一个error，用于解释Done()通道为什么关闭了
        fmt.Println("server:", err)
        internalError := http.StatusInternalServerError
        http.Error(w, err.Error(), internalError)
    }
}

func main() {

    http.HandleFunc("/hello", hello)
    http.ListenAndServe(":8090", nil)
}
```
将web服务在后台运行。使用curl模拟客户端请求/hello,在请求刚刚开始后，使用Ctrl+c发送取消信号
```bash
$ go run context-in-http-servers.go &


$ curl localhost:8090/hello
server: hello handler started
^C
server: context canceled
server: hello handler ended
```

# Reference

- https://blog.wu-boy.com/2020/08/three-ways-to-manage-concurrency-in-go/
- https://blog.wu-boy.com/2020/01/when-to-use-go-channel-and-goroutine/
- https://blog.wu-boy.com/2020/05/understant-golang-context-in-10-minutes/
- https://gobyexample.com/context