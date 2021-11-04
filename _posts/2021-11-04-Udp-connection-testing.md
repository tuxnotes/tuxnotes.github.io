---
layout: post
title: Udp-connection-test
author: tux
date: 2021-11-04
tags: Linux
---

如果是测试TCP的联通性，一般可以使用telnet即可完成。但是如果要测试udp端口的连通性，则可以使用iperf或nc完成。

方法一：iperf

iperf是一款自由的客户端服务端工具，用于验证udp连通性和吞吐

Server setup: Run iperf to listen for UDP traffic on port 33001
```bash
iperf -s -p 33001 -u
```
On the client: Send to the server machine that's listening for UDP on port 33001 (in this case IP 10.0.101.49) at 10 Mbps:

```bash
iperf -c 10.0.101.49 -u -p 33001 -b 10M 
```
方法二： netcat

netcat可以说是网络工具里的瑞士军刀。

On the server run netcat in listen mode (-l) and specify UDP (-u) and the port (33001):
```bash
# nc -l -u 33001
```
On the client specify UDP (-u) and provide the server IP address and port that the server is listening to:
```bash
# nc -u ip_address 33001
```