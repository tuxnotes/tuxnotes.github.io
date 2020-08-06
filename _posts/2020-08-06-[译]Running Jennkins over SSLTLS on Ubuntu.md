---
layout: post
date: 2020-08-06
title: 基于SSL/TLS运行Jenkins
author: tux
tags: jenkins tls
---

原文链接：https://medium.com/@ikonnea/running-jenkins-over-ssl-tls-on-ubuntu-e70de750d92，by Azunna Ikonne

默认情况下，jenkins运行基于http协议，这意味着用户名和密码均以明文方式传输。为了开启https协议，需要一个证书，这个证书是需要导入到
trusted keystore中的。Jenkins使用java开发，默认的java的keystore文件位于$JAVA_HOME目录。

### 开始之前的工作

#### 1 环境变量

为了使接下来的步骤更加容易，需要检查$JENKINS_HOME和$JAVA_HOME环境变量已正确设置并生效。

```bash
$ echo $JENKINS_HOME
/var/lib/jenkins
$ echo $JAVA_HOME
/usr/lib/jvm/java-8-openjdk-amd64/
```
如果有问题可使用下面的命令来生成：

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
export JENKINS_HOME=/var/lib/jenkins/
```
使环境变量永久生效的办法是将命令加入`~/.profile`文件中，然后执行：

```bash
$ source ~/.profile
```

#### 2 创建目录

在$JENKINS_HOME目录下创建`.keystore`目录，并对jenkins用户授权

```bash
$ mkdir $JENKINS_HOME/.keystore
$ chown -R jenkins: $JENKINS_HOME/.keystore
```
### 创建自签名ssl证书

这个步骤中将会用到OpenSSL和keytool工具。openssl工具默认会安装好，keytool工具由java提供。

#### 1 编辑openssl.cnf配置文件

在配置文件末尾添加下面内容：

```
[ subject_alt_name ] subjectAltName = DNS:yourdomain.com, DNS:example.yourdomain.com, DNS: localhost
```

如果你想在内网远程访问Jenkins，添加localhost是非常重要的

#### 2 创建公司钥对并添加CN name和organisation details

在创建证书前，进入$JENKINS_HOME/.keystore目录

```bash
$ sudo openssl req -x509 -nodes -newkey rsa:2048 -config /etc/ssl/openssl.cnf -extensions subject_alt_name \
-keyout private.key -out self_signed.pem \
-subj '/C=NG/ST=Lagos/L=Victoria_Island/O=Your_Organization/OU=Your_department /CN=www.yourdomain.com/emailAddress=youremail@yourdomain.com' -days 365
```

#### 3 检查`self_signed.pem`文件是否成功生成

```bash
$ sudo openssl x509 -in self_signed.pem -text -noout
```

输出大致如下：

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            5d:48:9c:47:ca:68:d9:90:22:8a:ca:df:5c:a6:91:51:7a:fc:d5:48
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = NG, ST = lagos, L = lagos, O = mygroup, OU = servicing, CN = www.zunnet.com, emailAddress = postmaster@example.com
        Validity
            Not Before: Jan 3 10:23:26 2020 GMT
            Not After : Jan 2 10:23:26 2021 GMT
        Subject: C = NG, ST = lagos, L = lagos, O = mygroup, OU = servicing, CN = www.zunnet.com, emailAddress = postmaster@example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:c8:75:12:40:17:0f:bd:cd:4f:14:bc:4c:53:b4:
```

#### 4 导出公钥(self_signed.pem)文件为PKCS12格式

这里会提示输入密码，这个密码会在接下来的步骤中用到

```bash
$ sudo openssl pkcs12 -export -keypbe PBE-SHA1-3DES -certpbe PBE-SHA1-3DES \
-export -in self_signed.pem -inkey private.key -name myalias -out jkeystore.p12
```

#### 5 将.p12文件转换为.jks格式

```bash
$ sudo keytool -importkeystore -destkeystore jkeystore.jks -deststoretype PKCS12 -srcstoretype PKCS12 -srckeystore jkeystore.p12
```

#### 6 验证.jks文件

会提示输入密码

```bash
$ sudo keytool -list -v -keystore jkeystore.jks
Enter keystore password:
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

Alias name: myalias
Creation date: Jan 3, 2020
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: EMAILADDRESS=postmaster@example.com, CN=www.zunnet.com, OU=servicing, O=mygroup, L=lagos, ST=lagos, C=NG
Issuer: EMAILADDRESS=postmaster@example.com, CN=www.zunnet.com, OU=servicing, O=mygroup, L=lagos, ST=lagos, C=NG
Serial number: 5d489c47ca68d990228acadf5ca691517afcd548
Valid from: Fri Jan 03 10:23:26 UTC 2020 until: Sat Jan 02 10:23:26 UTC 2021
Certificate fingerprints:
          MD5: 1B:DC:0A:8B:A7:9C:A5:AB:89:0D:97:67:FD:94:F7:7F
          SHA1: 42:79:99:36:63:2F:14:4F:EA:29:E2:7D:87:25:39:E0:74:D4:DE:A3
          SHA256: EC:E7:7E:9B:53:2A:85:3C:FC:C1:71:AC:10:AD:E0:9A:17:FD:3C:36:5F:45:90:16:E7:A2:32:C8:98:DF:7E:D5
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions:

#1: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: example.mydomain1.com
  DNSName: zunnet.com
  DNSName: jenkins.zunnet.com
  DNSName: localhost
```

从输出结果可以看到subject alternative names已经导入到证书。

#### 7 从.jks文件生成证书

```bash
$ sudo keytool -export -keystore jkeystore.jks -alias myalias -file self-signed.crt
```

自签名证书会导入到默认的java trusted keystore(cacerts)，位于$JAVA_HOME/jre/lib/security目录。

#### 8 将自签名证书导入java cacerts trusted keystore

```bash
$ sudo keytool -importcert -file self-signed.crt -alias myalias -keystore \
$JAVA_HOME/jre/lib/security/cacerts
```
现在证书导入完成，接下来可以停止Jenkins服务

```bash
$ sudo service jenkins stop
```
### 测试配置

在编辑/etc/default/jenkins文件前，应该使用下面的命令测试配置

```bash
$ sudo java -jar /usr/share/jenkins/jenkins.war \
--httpsPort=8443 \
--httpPort=-1 \
--httpsKeyStore=jkeystore.jks \
--httpsKeyStorePassword=yourkeystorepass
```

**Note: This command will reset your database if you had setup Jenkins prior to this configuration. Skip this step if you don’t want to lose your jobs and configurations.**

### 编辑Jenkins配置文件

编辑jenkins配置文件，传入如下参数：

```bash
$ sudo nano /etc/default/jenkins
```
添加下面的文本。(cacerts keystore的默认密码是changeit)
```
# arguments to pass to java
JAVA_ARGS="Djavax.net.ssl.trustStore=$JAVA_HOME/jre/lib/security/cacerts -Djavax.net.ssl.trustStorePassword=changeit"
```

接着增加heap allocation size.在JAVA_ARGS="-Djava.awt.headless=true"中增加参数：

```
JAVA_ARGS="-Xmx2048m -Djava.awt.headless=true"
```
通过将`HTTP_PORT`从8080改为-1来关闭http协议连接。

接着按如下修改`JANKINS_ARGS`参数：

```
JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --httpsPort=8443 --httpsKeyStore=$JENKINS_HOME/.keystore/jkeystore.jks --httpsKeyStorePassword=yourkeystorepass"
```
保存配置并退出，启动jenkins
```bash
$ sudo service jenkins start
```
使用浏览器通过8443端口连接jenkins服务器：https://yourserverip:8443/
你会收到一个自签名证书的警告，但可以忽略错误并继续。

当然这只是Jenkins基于ssl/tls的一种方式，也可以将jenkins放到开启了ssl/tls的apache或nginx服务之后。

