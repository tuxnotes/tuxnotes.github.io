---
layout: post
title: Debian-10使用L2TP VPN客户端连接VPN
date: 2020-11-12
author: tux
tags: l2tp VPN
---

### 1 安装相关包

```bash
$ sudo apt-get update
$ sudo apt-get install network-manager-l2tp  network-manager-l2tp-gnome
```

### 2 配置

早期的Debian系统可能还需要安装nm-l2tp

1. Go to Settings -> Network -> VPN. Click the + button.
2. Select Layer 2 Tunneling Protocol (L2TP).
3. Enter anything you like in the Name field.
4. Enter Your VPN Server IP for the Gateway.
5. Enter Your VPN Username for the User name.
6. Right-click the ? in the Password field, select Store the password only for this user.
7. Enter Your VPN Password for the Password.
8. Leave the NT Domain field blank.
9. Click the IPsec Settings... button.
10. Check the Enable IPsec tunnel to L2TP host checkbox.
11. Leave the Gateway ID field blank.
12. Enter Your VPN IPsec PSK for the Pre-shared key.
13. Expand the Advanced section.
14. Enter aes128-sha1-modp2048! for the Phase1 Algorithms.
15. Enter aes128-sha1-modp2048! for the Phase2 Algorithms.
16. Click OK, then click Add to save the VPN connection information.
17. Turn the VPN switch ON.

开始根据给定的资料配置后，开启VPN总是提示activation of network falied。对比上面的步骤发现是少了14-15步。

14-15步的密码也可以分别填入如下内容：

```
Phase1 Algorithms:3des-sha1-modp1024
Phase2 Algorithms:3des-sha1
```
但是一定要注意，在配置完后IPv4和IPv6选项卡的`Use this connection only for resources on this network`一定不要勾选。勾选的话即便用户名和密码等配置没有问题，也是连不通的。如果勾选，路由表如下：

```bash
$ sudo route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.43.1    0.0.0.0         UG    600    0        0 wlp3s0
111.12.83.148   192.168.43.1    255.255.255.255 UGH   600    0        0 wlp3s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 virbr0
192.168.18.1    0.0.0.0         255.255.255.255 UH    50     0        0 ppp0
192.168.43.0    0.0.0.0         255.255.255.0   U     600    0        0 wlp3s0
192.168.43.1    0.0.0.0         255.255.255.255 UH    600    0        0 wlp3s0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr1
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```
如果不勾选的话，路由表如下：
```bash
$ sudo route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         0.0.0.0         0.0.0.0         U     50     0        0 ppp0
0.0.0.0         192.168.43.1    0.0.0.0         UG    600    0        0 wlp3s0
111.12.83.148   192.168.43.1    255.255.255.255 UGH   600    0        0 wlp3s0
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 virbr0
192.168.18.1    0.0.0.0         255.255.255.255 UH    50     0        0 ppp0
192.168.43.0    0.0.0.0         255.255.255.0   U     600    0        0 wlp3s0
192.168.43.1    0.0.0.0         255.255.255.255 UH    600    0        0 wlp3s0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr1
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```
由此可见对路由是有影响的。
总结一下关键步骤如下：

1. 安装VPN插件：`apt install network-manager-l2pt-gnome`
2. 打开GUI界面，填入用户名，密码，gateway.
3. IPsec Settings:勾选Enbale IPsec tunnel to L2PT host;填写Pre-shared key;根据上面的步骤填入加密算法；勾选Enforce UDP encapsulation;点击OK
4. 这样就可以了，PPP Settings不用配置，保留默认就好。


### Reference

1. https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md#linux
2. https://www.tecmint.com/setup-l2tp-ipsec-vpn-client-in-linux/
3. https://my.oschina.net/podjonss/blog/2243307
