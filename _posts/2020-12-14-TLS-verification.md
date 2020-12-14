---
layout: post
title: TLS Verification
date: 2020-12-14
author: tux
tags: tls
---

# 1 What is the SSL Certificate Common Name?

参考链接：https://support.dnsimple.com/articles/what-is-common-name/

Common Name(通常也称为CN)代表服务器的server name。只有请求的hostname与证书的common name匹配的时候才会认为证书有效。如果浏览器连接的地址与证书的CN不匹配将会提示警告信息。

在证书中只有一个name的情况下，Common Name由单个主机名组成(如example.com,wwww.example.com)或一个通配符的名称组成(如*.example.com)

## commonName formatCN格式

CN并不是一个URL，它不包含任何的协议(如http://或https://)，端口号或路径名称。例如`https://example.com或example.com/path都是不正确的。这两种情况下，CN应该是`example.com`。

>NOTE:至于选择单域名证书还是通配符证书请参阅[ Choosing the SSL Certificate Common Name](https://support.dnsimple.com/articles/ssl-certificate-names)的内容来帮助你确定合适的证书类型

chrome浏览器当主机名不匹配的时候将出现下图所示的错误：

![](/assets/img/dnsimple-certificate-mismatch-chrome.png)

主机名在Google Safari上不匹配的错误如下;

![](/assets/img/dnsimple-certificate-mismatch-safari.png)

## Common Name VS Subject Alternative Name

common name只能包含一个入口：要么是通配符的CN要么不是通配符的CN。通过SSL 证书在Common Name字段包含一个名称列表是不允许的。

所以引入了Subject Alternative Name extension(也被称为Subject Alternative Name或SAN)来解决这个局限性。SAN允许在SSL证书中发布多个名称。

直接在证书中指定SAN内容的能力取决于CA和特定的产品。大多数CA都有一个历史标记的多域名SSL证书做为一个独立的产品。这类产品通常比标准的单域名证书收费很高。

在技术层面，SAN的引入是为了整合Common Name。因为HTTPS是在2000年引入的(由RFC2818定义)，使用CN字段已经被废弃，因为这被认为是不明确或类型不明确的。

所以CA/Browser Forum已经授权SAN可以包含CN中出现的任何内容，这是的证书server name的匹配验证只需要SAN作为唯一引用，从而提高了效率。现在common name的使用仅存在于过去传统的情况。还有一些讨论要求在浏览器或接口中移除CN的使用。

更多信息请参考：https://support.dnsimple.com/categories/ssl-certificates/

接下来考虑将上述连接中的文章都翻译以便，以供学习之用。

# Reference

- https://en.wikipedia.org/wiki/Transport_Layer_Security
- https://developer.okta.com/books/api-security/tls/certificate-verification/
- https://wiki.openssl.org/index.php/Hostname_validation
- https://u.uniface.info/docs/1000/uniface/networkSupport/TLS/TLS_peerNameVerification.htm
- https://tools.ietf.org/html/rfc6125
- https://support.dnsimple.com/articles/what-is-common-name/
- https://protonmail.com/blog/tls-ssl-certificate/
- https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.5.0/com.ibm.mq.sec.doc/q009940_.htm
- https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html
- https://security.stackexchange.com/questions/139176/details-of-tls-certificate-verification