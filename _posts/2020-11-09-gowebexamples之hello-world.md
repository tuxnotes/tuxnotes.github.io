---
layout: post
title: gowebexamples之hello world
date: 2020-11-09
author: tux
tags: golang
---

### Introduction

Go语言已经内置了一个webserver。标准库中的`net/http`包包含了HTTP协议的所有功能。这包括一个HTTP客户端和一个HTTP server.本例中你会发现使用go创建一个可以在你的浏览器中浏览的webserver是如此的简单。

### Registering a Request Handler注册请求处理器

首选需要创建一个Handler用于接收来自浏览器，HTTP clients或API请求的所有HTTP连接。go中的Handler就是有如下签名的一个函数：

```go
func (w http.ResponseWriter, r *http.Request)
```

这个函数接收两个参数：

- http.ResponseWriter: 将你的`text/html`返回的地方
- http.Request: 其包含了HTTP请求的所有信息，包括如URL或header字段

将请求处理器注册到默认的HTTP Server如同下面的代码一样简单：

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, you've requested: %s\n", r.URL.Path)
})
```

### Listen for HTTP Connections监听HTTP连接

只有请求Handler并不能接受HTTP连接.HTTP server 必须监听一个端口，将连接传给请求Handler。因为80端口大多数情况下是HTTP流量的默认端口，所以webserver可以监听这个端口。

下面的代码在启动go默认的HTTP服务时会监听80端口。你可以打开浏览器，输入`http://localhost/`来查看server处理你的请求。

```go
http.ListenAndServe(":80", nil)
```

### 代码

本节中完整的代码如下：

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, you've requested: %s\n", r.URL.Path)
    })
    http.ListenAndServe(":80", nil)
}
```
>NOTE:某些Linux系统，如debian 10，使用普通用户执行这段代码的时候，可能出现执行后就退出了，这是因为要监听端口，权限的问题，可编译后使用sudo执行
