---
layout: post
date: 2020-08-31
title: OpenSSL CA keyUsage extension的说明
author: tux
tags: OpenSSL keyUsage
---

参考链接：https://superuser.com/questions/738612/openssl-ca-keyusage-extension

任何一个CA证书，无论是根证书还是中间证书，都必须具有`keyCertSign`扩展。同样，如果你想使用CA证书签发废除列表(CRL),你需要添加`cRLSign`扩展。对于CA证书任何其他的`keyUages`扩展都要避免。
openssl接受值的详细信息请参考![文档](https://www.openssl.org/docs/manmaster/man5/x509v3_config.html)

对于终端实体的证书，你可以使用OpenSSL文档中列出的任何其他keyUsages.只要保证没有包含上面提到的CA-extensions。从安全的角度来说，不应该使用多与所需的keyusage扩展。但这并不是严格要求。
需要指出的是，除了一般的keyusage扩展，还有`extendedKeyUsage` (EKU) 扩展。这类扩展不限制于RFC中的预定义值，但理论上可以采用任何你想采用的OID。证书用常用来签发时间戳或OCSP responses。

你不需要使用`nsCertType`扩展，这些扩展以及ns*一类扩展是网景时代之前的，现在已不再使用。任何软件中可能找不到它们了。

唯一强制要求的扩展是`basicConstraints`扩展，设置方法为`basicConstraints=CA:TRUE`,根据具体使用也可设置为`FALSE`.
