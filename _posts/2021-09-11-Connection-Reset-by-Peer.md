---
layout: post
title: Connection Reset by Peer
author: tux
date: 2021-09-11
tags: network
---

# 1 背景

在工作中可能会经常遇到"Connection Reset by Peer"的错误或日志。接下来解释一下这个错误的主要含义，以及发生这种错误可能的原因。

# 2 意义解释

在tcp header中，有一个名为RST的bit位，表示reset.首先来看一下权威著作中的说明，TCP/IP Illustrated, Volume 1, 1st中的第十八章第七节：

>In general, a reset is sent by TCP whenever a segment arrives that doesn't appear correct for the referenced connection. (We use the term "referenced connection" to mean the connection specified by the destination IP address and port number, and the source IP address and port number. This is what RFC 793 calls a socket.)

大概的意思：通常，如果到达的数据段，在referenced connection看来是不正确的，则tcp就会发送reset。referenced connection指的是建立连接的四元组，包含目标地址，目标端口，源地址，源端口。RFC 793称之为socket。

从这里可以看出，发生reset是因为四元组发生了改变，或者说遭到了破坏。

# 3 可能原因

这里参考quora的回答，参考链接：https://www.quora.com/What-exactly-does-connection-reset-by-peer-mean-in-a-computing-parlance

计算机与计算机在互联网上通信，通常会基于某种协议，比如tcp。你的计算机会试图通过网络(经过很多路由器和其他的网络设备)与远端计算机建立一个 virtual “connection”。它并不像用电线将一个点与另一个点连接起来的物理连接，仅是在两台计算机达成一致后，想象中的连接，因为中间存在很多网络设备。

这种情况下，就可能出现下面几种错误的情况：

- 在给定的时间内，网络上找不到为双方建立连接而工作的路由，则会出现“connection timeout”的错误
- 如果对端拒绝基于你使用的协议进行通信(或者说无论什么协议都拒绝)，则关于看到“connection refused”。如果对端与你通信一段时间后决定停止与你通信，则会出现 “connection closed by remote host”
- 如果连接建立是OK的，或者是建立了部分连接，但是对端主机或者双方之间的任何一台网络设备终止了连接(一般都不是通过FIN正常终止的)，则会看到 “connection reset by peer”

这种错误意思是virtual connection已经建立，接着又断了。

It’s the computing or TCP/IP version of slamming the phone down on you.

The remote server sent an “RST packet” to you. This indicates an immediate dropping of the connection (rather than the normal ‘handshake’).

这种场景类似于正在通话的两个人，其中一个人手机没电自动关机，而另一个人认为对方还在，而继续通话。

简而言之TCP中间断开了，而不是通过四次挥手实现的。

What is a TCP Connection Reset by Peer?
An application gets a connection reset by peer error when it has an established TCP connection with a peer across the network, and that peer unexpectedly closes the connection on the far end.

That usually happens when the peer crashes, but can also happen with poorly-written applications or frameworks that don't shut their TCP connections down cleanly.

Similar to a connection refused error, a connection reset by peer error is generated when the operating system receives a TCP reset (RST) from the remote system.

What is a TCP Reset (RST)?
When an unexpected TCP packet arrives at a host, that host usually responds by sending a reset packet back on the same connection. A reset packet is simply one with no payload and with the RST bit set in the TCP header flags.

There are a few circumstances in which a TCP packet might not be expected; the two most common are:

The packet is an initial SYN packet trying to establish a connection to a server port on which no process is listening.

The packet arrives on a TCP connection that was previously established, but the local application already closed its socket or exited and the OS closed the socket.
Other circumstances are possible, but are unlikely outside of malicious behavior such as attempts to hijack a TCP connection.

Connection reset by peer的常见原因：
1）服务器的并发连接数超过了其承载量，服务器会将其中一些连接关闭；
   如果知道实际连接服务器的并发客户数没有超过服务器的承载量，看下有没有网络流量异常。可以使用netstat -an查看网络连接情况。
2）客户端关掉了socket，而服务器还在给客户端发送数据；  这属于正常情况
3）防火牆的问题；
   如果网络连接通过防火牆，而防火牆一般都会有超时的机制，在网络连接长时间不传输数据时，会关闭这个TCP的会话，关闭后在读写，就会导致异常。 如果关闭防火牆，解决了问题，需要重新配置防火牆，或者自己编写程序实现TCP的长连接。实现TCP的长连接，需要自己定义心跳协议，每隔一段时间，发送一次心跳协议，双方维持连接。

The technical explanation for this error is that Samba has tried to read (or write) from a TCP socket but that socket has been closed by the client at the other end with no notification given to Samba. Normally, as part of gracefully closing down a TCP connection, both the client and the server exchange TCP packets and agree to close both ends of their pipe. No error messages are produced in Samba's log files in this case. If something unexpected occurs at the remote end, such as the client being reset or the remote application crashing, then any open TCP connections may not be closed down properly. A condition like this may produce the connection reset by peer message in the log.

A more likely explanation, however, involves the behaviour of modern (Windows 2000 and above) clients when setting up a connection to a Samba server. SMB/CIFS operates on two TCP ports, port 139 and port 445. In the interests of minimising waiting time, Windows opens two connections simultaneously. One to port 139 and another to port 445. When it receives a response to one of these connection requests, the other one is discarded. When Samba tries to read a packet from the discarded connection, it receives the connection reset by peer error, and this is dutifully logged.

Samba could probably cope with this situation more gracefully, and not produce an error which resembles some kind of problem, but in reality is just part of the normal operation of the network.

Error message: Connection reset by peer: socket write error.

This basically means that a network error occurred while the client was receiving data from the server. But what is really happening is that the server actually accepts the connection, processes the request, and sends a reply to the client. However, when the server closes the socket, the client believes that the connection has been terminated abnormally because the socket implementation sends a TCP reset segment telling the client to throw away the data and report an error.

A connection reset by peer message means that the site you are connected to has reset the connection. This is usually caused by a high amount of traffic on the site, but may be caused by a server error as well.



Sometimes this can be solved by this documentation:

WebLogic Server Java Virtual Machine (JVM) Tuning and Garbage Collection (see link below):

http://www-1.ibm.com/support/docview.wss?rs=3214&context=SSLKT6&uid=swg21262003&loc=en_US&cs=utf-8&lang=en

# 工作中遇到此问题的原因

keepalived的健康检查脚本逻辑是看本机是否监听tcp端口，脚本中使用到了`netstat -tln |grep PORT`命令来检查。但是当机器作为网关的时候，netstat命令返回需要等待很长的时间，这导致超时，keepalived将VIP进行了切换，所以出现了connection reset by peer.

# Reference

- https://www.ibm.com/support/pages/connection-reset-peer-socket-write-error-error-message
- https://www.quora.com/What-exactly-does-connection-reset-by-peer-mean-in-a-computing-parlance
- https://stackoverflow.com/questions/1434451/what-does-connection-reset-by-peer-mean
- https://everything2.com/title/Connection+reset+by+peer
- https://testerhome.com/articles/23296
- https://www.pico.net/kb/what-is-a-tcp-connection-reset-by-peer/
- https://developer.aliyun.com/article/238641
- https://www.samba.org/~tpot/articles/connection-reset-by-peer
- https://github.com/envoyproxy/envoy/issues/5022
- https://github.com/envoyproxy/envoy/issues/14201
- https://www.pico.net/kb/what-is-a-tcp-reset-rst/
- https://en.wikipedia.org/wiki/TCP_reset_attack
- https://docs.microsoft.com/en-us/windows/client-management/troubleshoot-tcpip-connectivity
- https://www.geeksforgeeks.org/services-and-segment-structure-in-tcp/
- https://en.wikipedia.org/wiki/Transmission_Control_Protocol
