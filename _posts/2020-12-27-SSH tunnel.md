---
layout: post
title: SSH隧道
author: tux
date: 2020-12-27
tags: ssh
---

# SSH tunnel

本文译自https://www.ssh.com/ssh/tunneling/

本文解释了SSH隧道也称为ssh端口转发，如何使用它从互联网进入到企业内部网络，如何在防火墙上阻止SSH隧道。SSH隧道是一款强大的工具，但其有可能被滥用。当服务迁移到亚马逊AWS或其他的云计算服务是，控制SSH隧道尤其重要。

## 1 什么是SSH隧道

SSH隧道是一种方法，用于通过SSH加密连接传输网络数据。它可以对传统型应用进行加密，它也可用于实现VPN，以及穿过防火墙访问内网。

SSH是一个基于不可信任网络进行安全远程登录或文件传输的标准。它也提供了通过端口转发的方式对任何给定的应用进行数据加密，这一地基本是通过基于SSH的端口隧道。这意味着应用的数据直接通过SSH的加密连接进行传输，因此其在传输过程中不能被窃听或截获。ssh隧道队原本不支持加密功能的应用提供了加密功能。

![](/assets/img/Securing_applications_with_ssh_tunneling___port_forwarding.png)

上图展示了一个SSH隧道应用的简答总览。通过不可信的网络，ssh客户端与sshserver建立了安全连接。这个SSH连接被加密了，保护数据安全和完整，并认证通信的双方。

SSH连接用于应用连接到应用服务器。使用ssh隧道，应用会访问本地主机上ssh客户端监听的端口。SSH客户端会通过ssh隧道转发应用到服务器的数据。ssh服务器接着会连接实际应用的服务器，通常是与ssh server在相同的机器上或与ssh server在相同的数据中心的机器。应用通信被加密了，而不需要改变应用或终端用户。

## 3 谁使用SSH隧道

任何能登录到服务器的用户都能启用端口转发。这个被内部IT网络人员使用，他们登录到他们的家庭电脑或者云上的服务器，从服务器上转发一个端口回到企业内网，然后连接到工作电脑或其他服务器。

黑客和恶意软件也能使用类似的功能来对内网开后门。它也可以用来隐藏黑客攻击。隧道通常与ssh key和公钥认证一起使用，使过程完全自动化。

## 4 ssh给企业带来的好处

ssh隧道被广泛应用于企业环境，其使用主框架系统作为其应用的后端。在这些环境中，应用本身对安全的支持非常有限。通过使用ssh隧道就能满足SOX, HIPAA, PCI-DSS 以及其他标准，而不需要修改应用。

在许多情况下，这些应用或应用服务器改变代码可能是不现实的或者是改造成本高到无法接受的情况。源代码可能找不到或供应商已经不存在了，产品可能已经过了支持期限，或者是开发团队已经不存在了。添加安全封装，如ssh隧道，对这种应用的加密就提供了低成本可操作的方法。如整个国建范围内的ATM网络使用隧道来加密安全。

## 5 ssh隧道在企业中的数据风险

尽管ssh隧道很有用，但其制造的风险需要企业的IT安全团队进行说明。

## 6 ssh端口转发示例

### 6.1 Local Forwarding本地转发

本地转发用于将客户端上的一个端口转发到服务器上。通常是，ssh客户端在一个配置的端口上监听连接，当其收到一个连接后，就会通过隧道将连接转到ssh服务器。ssh服务器会连接到一个配置好的目的端口，一般不与ssh server在同一个机器上。

本地端口转发的典型应用包含以下几种：

- 通过跳板机进行转发session和文件传输
- 从外部访问内网服务
- 通过互联网连接远程文件共享服务

有些组织会通过单台跳板机来连接所有的ssh连接。跳板机通常是Linux或Unix服务器，通常会带有额外的加固，入侵检测，日志记录等功能。或者是一个商业的跳板机解决方案。

许多跳板机，一旦连接认证完成后就会允许端口转发。这样的端口转发是非常有用的，例如他们可以将自己本地电脑端口转发到企业内网的web服务器上，或转发到内网邮件服务器的IMAP端口，或本地文件服务器445和139端口，或者是打印机或者是版本仓库，或者是内网的任何系统。多数情况下端口被转发到了内网机器的SSH端口。

在openssh中，本地端口转发通过-L选项配置：

```bash
ssh -L 80:intra.example.com:80 gw.example.com
```
上面的示例打开了一个到跳板机gw.example.com服务器的连接，并将所有对本机80端口的连接转发到内网intra.example.com服务器的80端口。

默认情况下，任何人(甚至是不同机器)都能连接到ssh客户端的端口。但这个可以做限制，通过指定绑定地址来限制只有在本地的程序才能访问：

```bash
ssh -L 127.0.0.1:80:intra.example.com:80 gw.example.com
```
openssh客户端配置文件中的LocalForward选项可用于胚子端口转发而不需要在命令行指定

### 6.2 Remote Forwarding远程转发

在openssh中，远程ssh端口转发通过-R选项指定，如：
```
ssh -R 8080:localhost:80 public.example.com
```
这允许任何在远程服务器上的人连接到远程服务器的8080端口。这个连接将被转发到客户端主机，接着客户端或建立一个到localhost的80端口的连接。任何其他主机名或IP地址可替换localhost来指定要连接的主机。

这个例子在给一些外网的人访问内部webserver的情况非常有用。或者是暴露内部的web应用到公网。

默认情况下，openssh仅允许来自server主机连接的远程端口转发。但是服务器端的配置文件sshd_config中`GatewayPorts`选项可用于控制这个。可选的配置如下：

```
GatewayPorts no
```

这阻止来非ssh server主机的端口转发

```
GatewayPorts yes
```
允许任何连接的端口转发。如果服务器是在公网，则互联网上的任何人都能连接到这个端口。

```
GatewayPorts clientspecified
```
上面的配置意思是客户端指定的用于连接到端口的IP地址是被允许的。语法如下：
```
ssh -R 52.194.1.73:8080:localhost:80 host147.aws.example.com
```
这个例子的意思是只有来自IP地址为52.194.1.73到8080端口的连接才被允许。

更多信息以及动态转发的信息请参考下面的网站：

- https://www.tecmint.com/create-ssh-tunneling-port-forwarding-in-linux/
- https://www.linuxbabe.com/firewall/ssh-dynamic-port-forwarding
- https://www.booleanworld.com/guide-ssh-port-forwarding-tunnelling/
- https://netsec.ws/?p=278
- https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/
- https://linuxize.com/post/how-to-setup-ssh-tunneling/
- https://dev.to/__namc/ssh-tunneling---local-remote--dynamic-34fa

