---
laytou: post
title: Keystore与Truststore
date: 2021-02-04
author: tux
tags: Java
---

# Keystore与Truststore的异同

## 概念

大多数情况下，当我们的应用需要基于SSL/TLS进行通信的时候，我们就要使用keystore和truststore了。通常它们是密码保护的一些文件，与运行的程序在同一个文件系统中。直到Java8，文件的默认格式都是JKS。

但从Java9开始，默认keystone的格式为PKCS12. JKS与PKCS12最大的不同是，JKS是Java特有的格式，而PKCS12是一种标准的，语言中立的存储加密私钥和证书的方式。

## Java Keystore

Keystore存储了私钥，证书，或者只是密码，用于各种加密目的。采用别名的方式存储，便于查找。通常来讲，keystone持属于应用程序的秘钥，秘钥用于验证消息的完整性，以及对消息发送者的认证。

通常当服务需要使用HTTPS的时候，我们就需要使用keystone。在SSL握手过程中，服务会从keystone中查找私钥，并出示私钥对应的公钥证书给客户端。

对应的，如果客户端也想认证自己--mutual authentication双向认证的场景--客户端也会有keystone，并出示自己的公钥证书。

没有默认的keystore,因此如果我们想使用加密通信，就必须设置`javax.net.ssl.keyStore`和`javax.net.ssl.keyStorePassword`，如果我们的keystone不是默认格式，则还需要设置`javax.net.ssl.keyStoreType`。

当然我们也可以使用这些秘钥服务其他应用。私钥用于签名和解密数据，而公钥用于验证和加密数据。加密秘钥也可以完成这些功能。keystone就是存储这些秘钥的地方。

关于keystone的使用参考[interact with the keystore programmatically](https://www.baeldung.com/java-keystore)

## Java Truststore

与keystone相反，keystone存储确认自己身份的证书，truststore存储用于其他身份的证书。在Java中，使用truststore来信任第三方的证书，来认证我们将要通信的另一方。拿前面的例子来说，当客户端基于HTTPS与Java的服务通信时，Java服务首先从keystone中找到私钥对应的公钥和证书出示给客户端。客户端会在truststore中查找关联的证书。如果外部服务出示的证书或CA不在client的truststore中，则出现SSLHandshakeException的异常，并且连接不会建立成功。

Java再带的truststore成为cacerts，位于`$JAVA_HOME/jre/lib/security`，其包含默认信任的CA：

```bash
$ keytool -list -keystore cacerts
Enter keystore password:
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 92 entries

verisignclass2g2ca [jdk], 2018-06-13, trustedCertEntry,
Certificate fingerprint (SHA1): B3:EA:C4:47:76:C9:C8:1C:EA:F2:9D:95:B6:CC:A0:08:1B:67:EC:9D
```
我们看到truststore包含了92个证书，其中之一是verisignclass2gca ，这意味着JVM会自动信任由verisignclass2gca 签署的证书。

我们可以通过设置`javax.net.ssl.trustStore`来改写truststore的默认位置。同样我们也可以设置`javax.net.ssl.trustStorePassword`和`javax.net.ssl.trustStoreType`来设置truststore的密码和类型。

## 结论

简而言之：

- keystone用于存储程序的私钥和证书，出示给server和client，用于验证
- truststore存储CA的证书，证书用于在SSL连接时验证server的证书

参考[SSL guide](https://www.baeldung.com/java-ssl)和[JSSE Reference Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html)来了解更多关于Java加密通信的细节。

## Reference

- https://www.educative.io/edpresso/keystore-vs-truststore
- https://www.baeldung.com/java-keystore-truststore-difference



