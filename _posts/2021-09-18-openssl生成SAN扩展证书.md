---
layout: post
title: openssl生成SAN扩展证书
author: tux
date: 2021-09-18
tags: PKI
---

# 1 使用OpenSSL为NetScaler应用生成自签SAN证书

参考连接：

- https://support.citrix.com/article/CTX135602
- https://github.com/levitte/openssl/blob/43890d7fc963d8c5ec3084dcf6c6c20b5efcaa7f/doc/man1/req.pod

采用下面的步骤生成具有一个SAN名称的自签SAN扩展证书

1. 本机上创建OpenSSL配置文件，并编辑相关字段。假设配置文件名称为`req.conf`

   ```
   [req]
   distinguished_name = req_distinguished_name
   x509_extensions = v3_req
   prompt = no
   [req_distinguished_name]
   C = US
   ST = VA
   L = SomeCity
   O = MyCompany
   OU = MyDivision
   CN = www.company.com
   [v3_req]
   keyUsage = keyEncipherment, dataEncipherment
   extendedKeyUsage = serverAuth
   subjectAltName = @alt_names
   [alt_names]
   DNS.1 = www.company.net
   DNS.2 = company.com
   DNS.3 = company.net
   ```

   

2. 上传文件到NetScaler应用服务器的/nsconfig/ssl目录

3. 登录netscaler服务器命令行

4. 运行下面的命令创建证书

   ```bash
   cd /nsconfig/ssl
   openssl req -x509 -nodes -days 730 -newkey rsa:2048 -keyout cert.pem -out cert.pem -config req.conf -extensions 'v3_req'
   ```

   

5. 验证证书

   ```bash
   openssl x509 -in cert.pem -noout –text
   Certificate:
   Data:
   Version: 3 (0x2)
   Serial Number:
   ed:90:c5:f0:61:78:25:ab
   Signature Algorithm: md5WithRSAEncryption
   Issuer: C=US, ST=VA, L=SomeCity, O=MyCompany, OU=MyDivision, CN=www.company.com
   Validity
   Not Before: Nov 6 22:21:38 2012 GMT
   Not After : Nov 6 22:21:38 2014 GMT
   Subject: C=US, ST=VA, L=SomeCity, O=MyCompany, OU=MyDivision, CN=www.company.com
   Subject Public Key Info:
   Public Key Algorithm: rsaEncryption
   RSA Public Key: (2048 bit)
   Modulus (2048 bit):
   …
   Exponent: 65537 (0x10001)
   X509v3 extensions:
   X509v3 Key Usage:
   Key Encipherment, Data Encipherment
   X509v3 Extended Key Usage:
   TLS Web Server Authentication
   X509v3 Subject Alternative Name:
   DNS:www.company.net, DNS:company.com, DNS:company.net
   Signature Algorithm: md5WithRSAEncryption …
   ```

   

可以将证书导入客户端的信任根证书。

# 2 命令行方式创建

参考链接：https://security.stackexchange.com/questions/74345/provide-subjectaltname-to-openssl-directly-on-the-command-line

OpenSSL1.1.1在命令行直接提供了SubjectAltName配置的选项，通过在`openssl req`命令中添加`-addext`来配置，示例如下

```bash
openssl req -new -subj "/C=GB/CN=foo" \
                  -addext "subjectAltName = DNS:foo.co.uk" \
                  -addext "certificatePolicies = 1.2.3.4" \
                  -newkey rsa:2048 -keyout key.pem -out req.pem
```

另外的方式

```bash
openssl req -new -sha256 \
    -key domain.key \
    -subj "/C=US/ST=CA/O=Acme, Inc./CN=example.com" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf \
        <(printf "\n[SAN]\nsubjectAltName=DNS:example.com,DNS:www.example.com")) \
    -out domain.csr
```

上面的命令写成一行

```bash
openssl req -new -sha256 -key domain.key -subj "/C=US/ST=CA/O=Acme, Inc./CN=example.com" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:example.com,DNS:www.example.com")) -out domain.csr
```

示例

```bash
user@hostname:~$ openssl req -new -sha256 -key domain.key -subj "/C=US/ST=CA/O=Acme, Inc./CN=example.com" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:example.com,DNS:www.example.com\n")) -out domain.csr
user@hostname:~$ openssl req -in domain.csr -text -noout
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=US, ST=CA, O=Acme, Inc., CN=example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:a8:05:50:86:49:98:c8:05:01:e9:50:18:7f:2f:
                    b4:89:09:29:d1:c1:58:d8:14:bb:58:1d:25:50:11:
                    bb:43:d8:28:03:a5:de:59:49:bb:d2:f7:d3:79:5c:
                    c6:99:2c:98:ff:99:23:8c:df:96:7c:ea:4b:62:2a:
                    a4:c2:84:f5:5d:62:7f:7d:c4:7c:e2:c3:db:e6:58:
                    03:c2:26:9d:02:da:bb:84:d9:11:82:fe:38:12:9b:
                    c7:b6:ff:b2:40:30:38:b1:44:d8:47:1d:43:4a:29:
                    58:6b:49:ec:33:d7:dc:a7:1b:90:05:3a:f5:e6:16:
                    98:08:5d:2d:7e:b4:ea:a2:a4:b1:84:89:f7:f1:c4:
                    67:a6:a1:06:70:dd:4e:6b:0c:f8:b5:9b:bc:3f:06:
                    ee:90:d6:86:29:52:d3:af:f6:d4:2f:c6:cf:4b:5a:
                    b8:cd:01:74:6d:5c:25:a8:02:1c:7c:e8:66:3d:46:
                    07:b1:9d:ef:cc:eb:90:b6:bf:7b:33:e0:5f:b2:9b:
                    e8:b4:12:67:2f:8d:0d:9b:54:9d:95:6e:09:83:cb:
                    f3:5b:1f:31:8e:3b:ca:4e:08:e0:40:c0:60:40:72:
                    dd:0d:3e:99:ec:7c:ac:c4:3c:ba:85:9d:d9:d9:6b:
                    02:2e:bf:a8:a3:02:1d:eb:c8:58:e3:04:b3:a5:f1:
                    67:37
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name: 
                DNS:example.com, DNS:www.example.com
    Signature Algorithm: sha256WithRSAEncryption
         a2:1d:1a:e8:56:43:e7:e5:c7:c1:04:c1:6a:eb:d5:70:92:78:
         06:c1:96:fa:60:e2:5f:3c:95:ee:75:ed:70:52:c1:f0:a7:54:
         d2:9f:4a:2f:52:0f:d4:27:d8:13:73:1f:21:be:34:3f:0a:9c:
         f1:2a:5c:98:d4:28:b8:9c:78:44:e8:ea:70:f3:11:6b:26:c3:
         d6:29:b3:25:a0:81:ea:a2:55:31:f2:63:c8:60:6d:68:e3:ab:
         24:c9:46:33:92:8f:f2:a7:72:43:c6:aa:bd:8d:e9:6f:64:64:
         9e:fe:30:48:3f:06:2e:58:7c:b5:ef:b1:4d:c3:84:cc:02:a5:
         58:c3:3f:d8:ed:98:c7:54:b9:5e:50:44:5e:be:99:c2:e4:03:
         81:4b:1f:47:9a:b0:4d:74:7b:10:29:2f:84:fd:d1:70:88:2e:
         ea:f3:42:b7:06:94:4a:06:f6:92:10:4c:ce:de:65:89:2d:0a:
         f1:0f:79:90:02:a4:b9:6d:b8:39:db:de:6e:34:61:4f:21:36:
         a0:b5:73:2b:2b:c6:7e:2f:f2:e5:1e:51:9f:85:c8:17:9c:1a:
         b6:59:b0:41:a7:06:c8:5b:f4:88:92:c9:34:71:9d:73:f0:2e:
         31:ae:ed:ab:35:0e:b4:8a:9a:72:7c:6f:7a:3e:5d:66:49:26:
         26:99:e1:69
```

# 3 使用OpenSSL生成SAN证书

参考链接：https://geekflare.com/san-ssl-certificate/

SAN证书的好处：减少SSL成本与维护工作，多个网站使用一个证书

SAN代表**Subject Alternative Names**，可以使用一个证书配置多个CN(Common Name)

你也可能考虑使用[wildcard SSL](https://namecheap.pxf.io/c/245992/386170/5618?u=https%3A%2F%2Fwww.namecheap.com%2Fsecurity%2Fssl-certificates%2Fcomodo%2Fessentialssl-wildcard.aspx)通配符证书，但还是有些不同的，SAN证书中你可以配置多个**完整的CN**。例如：

- geekflare.com
- gf.dev
- siterelic.com
- chandan.io

SAN证书的CSR创建与传统的OpenSSL命令有些不同。下图是skype.com证书的真实示例

<img src="https://geekflare.com/wp-content/uploads/2015/09/skype-san.png" style="zoom:150%;" />

## 3.1 生成具有SAN的CSR过程

- 登录到有OpenSSL命令的服务器

- 创建一个目录

- 创建一个配置文件san.conf，内容如下

  ```
  [ req ]
  default_bits       = 2048
  distinguished_name = req_distinguished_name
  req_extensions     = req_ext
  [ req_distinguished_name ]
  countryName                 = Country Name (2 letter code)
  stateOrProvinceName         = State or Province Name (full name)
  localityName               = Locality Name (eg, city)
  organizationName           = Organization Name (eg, company)
  commonName                 = Common Name (e.g. server FQDN or YOUR name)
  [ req_ext ]
  subjectAltName = @alt_names
  [alt_names]
  DNS.1   = bestflare.com
  DNS.2   = usefulread.com
  DNS.3   = chandank.com
  ```

  `alt_names`部分是需要你添加额外DNS的地方

- 保存文件并执行如下命令，就会生成CSR和key文件

  ```bash
  openssl req -out sslcert.csr -newkey rsa:2048 -nodes -keyout private.key -config san.cnf
  ```

  上面的命令会生辰sslcert.csr与private.key文件，你需要将sslcert.csr发给证书签发机构。

  ## 3.2 如何验证

  使用如下命令验证

  ```bash
  [root@Chandan test]# openssl req -noout -text -in sslcert.csr | grep DNS
                 DNS:bestflare.com, DNS:usefulread.com, DNS:chandank.com
  [root@Chandan test]#
  ```

  

# Reference

- https://www.openssl.org/docs/man1.0.2/man5/x509v3_config.html
- https://gist.github.com/tuxnotes/037ca62bacbc9cd08f32fbeae629cfa3
- https://cloud.tencent.com/developer/ask/94482
- https://www.golinuxcloud.com/openssl-subject-alternative-name/