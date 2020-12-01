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

