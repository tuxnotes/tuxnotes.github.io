---
layout: post
title: golang之URL查询字符串
date: 2021-02-25
author: tux
tags: golang
---

# 1 问题引出

在使用net/http包进行请求的时候，如使用get方法，URL中带查询字符串。此时不能直接使用，需要对查询字符串进行编码，即使用%的编码形式。例如人力可读的URL加查询参数如下所示：

```
http://ip:443/path?title=日志缺失&message=北京 89%,上海 73%&user_id=abc|xiaoming|xiaowang
```
但上述内容不能直接用于http包的get方法，需要使用net/url包进行编码。但上面的内容复制到浏览器中会自动进行转码成百分号编码的形式。如果直接使用上面的内容进行get请求，在我的案例中则遇到了"502 Bad Gateway"的错误，而编码后就正常工作了。

# 2 使用方法

```bash
package main

import (
    "fmt"
    "net/http"
    "net/url"
)

func main() {
    params := url.Values{}
    params.Add("message", "this will be esc@ped!")
    params.Add("author", "golang c@fe >.<")
    fmt.Println("http://example.com/say?"+params.Encode())
    http.Get("http://example.com/say?"+params.Encode()) // 后面的内容省略
}
```
输出
```
http://example.com/say?author=golang+c%40fe+%3E.%3C&message=this+will+be+esc%40ped%21
```

# Reference

- https://golang.cafe/blog/how-to-url-encode-string-in-golang-example.html
- https://www.urlencoder.io/golang/

