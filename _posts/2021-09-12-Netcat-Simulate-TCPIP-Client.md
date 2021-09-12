---
layout: post
title: 使用netcat模拟tcp/ip客户端
author: tux
date: 2021-09-12
tags: network
---

# 1 背景

在调试网络的过程中，有时需要模拟tcp/ip服务端与客户端，netcat是一款能完成此类工作的强大工具。

# 2 使用

使用语法

```
nc [-options] hostname port[s] [ports]
nc -l -p port [-options] [hostname] [port]
```

参数说明

- l: set the “listen” mode, waits for the incoming connections.
- p: local port.
- u: set the UDP mode.

## 2.1 客户端模拟
### 2.1.1 TCP
首先模拟server
```bash
$ nc -l 2399
```
客户端

```bash
$ nc localhost 2399
```
### 2.1.2 UDP

启动UDP服务
```bash
$ nc -u -l 2399
```
UDP客户端
```bash
$ nc -u localhost 2399
```

### 2.2 用于扫描

有时只是想测试一下端口是否通，而telnet会有交互模式，此时就可以使用nc的-z选项，说明如下：
```
-z           zero-I/O mode [used for scanning]
```

示例如下
```
$ nc -z ip port
```
上面的命令执行后，不管端口通不通都会退出，而没有任何提示，此时可以使用`$?`这个特殊变量来判断上述执行是否成功。


# Reference

- https://ubidots.com/blog/how-to-simulate-a-tcpudp-client-using-netcat/