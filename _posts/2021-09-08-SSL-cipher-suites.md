---
layout: post
title: SSL cipher suite
author: tux
date: 2021-09-08
tags: tls
---

# Cipher suite

一个加密套件是一个四件套，包含四个功能：密钥交换算法、身份验证算法、对称加密算法和信息摘要算法。

![](https://pic3.zhimg.com/80/v2-4d51eb42c28bc686dada0223d7c767ba_720w.jpg)

在早期版本的 Windows中，使用单个字符串配置了 TLS 密码套件和椭圆曲线，如下图所示：

![](https://docs.microsoft.com/zh-cn/windows/win32/secauthn/images/tls-cipher-suite.png)

密码套件为以下每种任务指定一种算法：

- 密钥交换
- 批量加密
- 消息验证

## 密钥交换算法

密钥交换算法保护创建共享密钥所需的信息，这些算法是非对称(公钥算法)算法，数据量相对较小，性能良好。

SSL通信过程(握手结束后)中，双方使用的是堆成加密的方式。由于通信双方以前并不知道彼此的存在，他们也不可能预先存储相同的加密秘钥，那应该怎么做呢？答案是在SSL通信的握手阶段，使用秘钥交换算法使双方使用的秘钥保持一致。

常用的密钥交换算法有RSA、Diffie-Hellman密钥交换、ECDH（Elliptic Curve Diffie-Hellman）、SRP（安全远程密码）、由TLS 1.2支持密钥交换算法PSK（Pre Shared Key）。

## 身份验证算法

身份验证又称“验证”、“鉴权”，是指通过一定的手段，完成对用户身份的确认。常用算法有 RSA、ECDSA、DSS

## 对称加密算法

对称加密（也叫私钥加密），也是上面提到的批量加密算法。用于对客户端和服务器之间交换的消息进行加密。 这些算法是 [*对称的*](https://docs.microsoft.com/zh-cn/windows/desktop/SecGloss/s-gly) ，适用于大量数据。

加密和解密使用相同密钥的加密算法。常用算法有 AES、DES、3DES。

## 信息摘要算法

根据某种运算规则对信息进行某种形式的提取，提出出来的数据就是摘要。主要用于验证信息的完整性。常用算法有MD5、SHA-1等。

hashing是一种密码学技术，数据一旦使用这种技术转换为了其他形式，就不可能再根据hash值还原出原数据。

摘要经私钥加密后就是签名。

## 举例说明

比如加密套件为：`TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA1`

TLS：通信协议

ECDHE：秘钥交换算法

ECDSA：身份验证算法

AES_128_CBC：通信时使用的对称加密算法

SHA1：信息摘要算法

## Reference

- https://zhuanlan.zhihu.com/p/87943928
- https://docs.microsoft.com/zh-cn/windows/win32/secauthn/cipher-suites-in-schannel?redirectedfrom=MSDN
- https://www.wst.space/ssl-part1-ciphersuite-hashing-encryption/



