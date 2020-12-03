---
layout: post
title: gowebexamples之HTTP server
date: 2020-12-01
author: tux
tags: golang
---

原文链接：https://gowebexamples.com/http-server/

## 1 Introduction

在本示例中你将学习如何在golang中创建基本的HTTP server。首先我们讨论一下HTTP server应该具有什么能力。一个基本的HTTP server有几个关键的功能需要考虑：

- 处理动态请求：处理来自用户的请求，如登录账户上传图片
- 静态资源服务：为浏览器提供JavaScript，CSS，和图像，来为用户提供动态体验
- 接受连接：HTTP server必须监听某个端口来接受来自互联网的连接

## 2 Process dynamic request

`net/http`包提供了用于接受请求和动态处理请求的所有工具。我们可以使用`http.HandleFunc`函数注册一个新的handler。它接受的第一个参数是匹配的路径，第二个参数是执行的函数。

```go
http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Welcome to my website!")
})
```
至于动态方面，`http.Request`包含了关于请求及其参数的所有信息。可以通过`r.URL.Query().Get("token")`来读取GET请求的参数，或者通过`r.FormValue("email")`读取POST请求参数(HTML表单字段)。

## 3 静态资源服务

提供如JavaScript，CSS和图像的静态资源服务，可以使用内置的`http.FileServer`，并将其指向一个url路径。为了使文件服务器正确工作，它需要直到服务的文件来自哪里？通常采用如下方式：

```go
fs := http.FileServer(http.Dir("static/"))
/*
type Dir string
    A Dir implements FileSystem using the native file system restricted to a
    specific directory tree.

func FileServer(root FileSystem) Handler
    FileServer returns a handler that serves HTTP requests with the contents of
    the file system rooted at root.

    To use the operating system's file system implementation, use http.Dir:

        http.Handle("/", http.FileServer(http.Dir("/tmp")))
*/
```

一旦文件服务器准备好了，我们就需要将其指向一个url路径，如同动态请求那里一样。需要指出的是，为了使用文件服务正确，我们需要将url路径中的部分去除。通常是共享文件的上级目录的名称。

```go
http.Handle("/static/", http.StripPrefix("/static/", fs))
/*
func StripPrefix(prefix string, h Handler) Handler
    StripPrefix returns a handler that serves HTTP requests by removing the
    given prefix from the request URL's Path and invoking the handler h.
    StripPrefix handles a request for a path that doesn't begin with prefix by
    replying with an HTTP 404 not found error.
*/
```

## 4 接受连接

对于基本的HTTP服务器的最后异步是监听端口来接受来自网络的连接请求。如你所料，golang有一个内置的HTTP server，用于快速启动。一旦启动成功就可以在浏览器中访问了。

```go
http.ListenAndServe(":80", nil)
```

## 5 完整代码

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func (w http.ResponseWriter, r *http.Requests) {
        fmt.Fprintf(w, "Welcome to my website!")
    }) 
    fs := http.FileServer(http.Dir("static/"))
    http.Handle("/static/", http.StripPrefix("/static/", fs))
    http.ListenAndServe(":80", nil)
}
```