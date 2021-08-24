---
layout: post
title: Harbor配置HTTPS访问
author: tux
date: 2021-08-24
tags: harbor
---

默认情况下，harbor不采用证书加密和认证。因此部署harbor的时候可以采用HTTP连接。但是HTTP的连接方式仅限于测试和开发环境，不能连接到外网的环境。否则可能会遭到中间人攻击。生产环境中要使用HTTPS的方式。并且，如果Notary启用了Content Trust来对镜像签名，则必须使用HTTPS。

你必须创建SSL证书来配置HTTPS。你可以采用可信的第三方CA来签发你的证书，或者使用自签证书。本文将描述如何使用OpenSSL创建CA，已经如何使用CA来签署服务器证书和客户端证书。当然你也可以使用其他CA提供商，如Let's Encypt.

下面的步骤假定你的harbor仓库的hostname是`yourdomain.com`，并且`yourdomain.com`的DNS记录指向运行harbor的主机。

# 1 Generate a Certificate Authority Certificate生成CA证书

在生产环境中，你应该从CA机构获取证书。在开发或测试环境中可以使用自己生成的CA证书。运行下面的命令来生成CA证书。

## 1 生成CA证书的私钥

```bash
openssl genrsa -out ca.key 4096
```

## 2 生成CA证书

采用`-subj`选项中的值来表明你的组织机构。如果你使用FQDN连接到你harbor主机，那么你必须在common name(`CN`)的属性中指定FQDN。

```bash
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
 -key ca.key \
 -out ca.crt
```

# 2 生成服务端证书

证书通常是包含`.crt`和`.key`的文件，比如`yourdomain.com.crt`和`yourdomain.com.key`.

## 1 生成私钥

```bash
openssl genrsa -out yourdomain.com.key 4096
```

## 2 生成证书签名请求CSR

使用`-subj`中的配置来表示你的组织机构。如果使用FQDN连接到harbor主机，则必须在common name(CN)中指定域名，秘钥和CSR文件的文件名使用指定的域名命名，如`yourdomain.com.key`和`yourdomaim.com.csr`

```bash
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
    -key yourdomain.com.key \
    -out yourdomain.com.csr
```

## 3 生成x509 v3扩展文件

无论你是使用FQDN还是使用ip地址来连接到你的harbor主机，则你必须创建这个文件，以便你为harbor生成的证书遵从SAN和x509 v3扩展(Subject Alternative Name (SAN) and x509 v3 extension)的要求。将DNS后面的值替换成你自己的域名。

```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF
```

## 4 使用`v3.ext`文件生成harbor证书

将CSR与CRT文件名中的`yourdomain.com`替换成你自己的harbor hostname。

```bash
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in yourdomain.com.csr \
    -out yourdomain.com.crt
```
# 3 Provide the Certificates to Harbor and Docker证书分发给harbor与docker

在生成`ca.crt`,`yourdomain.com.crt`,`yourdomain.com.key`文件后，你必须将这些文件分发给harbor与docker，并重新配置harbor来使用证书。

## 1 将证书和私钥拷贝到harbor服务器的证书目录

```bash
cp yourdomain.com.crt /data/cert/
cp yourdomain.com.key /data/cert/
```
## 2 将`yourdomain.com.crt`转换为`yourdomain.com.cert`供docker使用

Docker守护进程会将`.crt`文件视为CA证书，`.cert`文件视为客户端证书。
```bash
openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert
```
## 3 将服务器端证书，私钥和CA文件拷贝到harbor主机上docker的证书目录。首先你必须先创建相关的目录

```bash
cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
cp ca.crt /etc/docker/certs.d/yourdomain.com/
```
如果你将Nginx默认的443端口改成了其他端口，则还需要创建目录`/etc/docker/certs.d/yourdomain.com:port`或`/etc/docker/certs.d/harbor_IP:port`

## 4 重启docker

```bash
systemctl restart docker
```
你可能还需要在OS层面信任证书。更多信息参考[Troubleshooting Harbor Installation](https://goharbor.io/docs/1.10/install-config/troubleshoot-installation/#https)

大致步骤如下：
如果你使用的是来自证书发行机构的中间证书，你可以将中间证书合并到你自己的证书中，来创建certificate bundle.运行下面的命令：

```bash
cat intermediate-certificate.pem >> yourdomain.com.crt
```
Docker守护进程运行于某种操作系统上，你可能需要在OS层面上新人证书，比如运行下面的命令：

Ubuntu系统
```bash
cp yourdomain.com.crt /usr/local/share/ca-certificates/yourdomain.com.crt 
update-ca-certificates
```

Red Hat (CentOS etc)
```bash
cp yourdomain.com.crt /etc/pki/ca-trust/source/anchors/yourdomain.com.crt
update-ca-trust
```
下面是使用自定义证书的目录示例
```
/etc/docker/certs.d/
    └── yourdomain.com:port
       ├── yourdomain.com.cert  <-- Server certificate signed by CA
       ├── yourdomain.com.key   <-- Server key signed by CA
       └── ca.crt               <-- Certificate authority that signed the registry certificate
```



