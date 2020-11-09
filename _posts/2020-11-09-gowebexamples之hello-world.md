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

