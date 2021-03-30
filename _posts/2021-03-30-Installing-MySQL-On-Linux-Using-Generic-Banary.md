---
layout: post
title: 使用通用二进制包在Linux上安装MySQL
data: 2021-03-30
author: tux
tags: mysql
---

# 使用通用二进制包在Linux上安装MySQL

## 1 安装

首先需要下载通用二进制tar包，然后下载依赖。依赖主要是libaio。包下载这里略过，Linux系统以Debian10为例：
```bash
$ apt search libaio
$ sudo apt install libaio1
$ sudo apt install libncurses5
```
当前系统安装的是libncurses6，而执行下面的命令报错：
```bash
$ /usr/local/mysql/bin/mysql -u root -p
/usr/local/mysql/bin/mysql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory
```
所以安装libncurses5解决。
## 2 创建mysql组和用户
mysqld默认的安装目录是/usr/local,为例后面减少参数的指定，这里将tar解压到/usr/local目录下
```bash
$ groupadd mysql
$ useradd -r -g mysql -s /bin/false mysql
```
## 3 数据目录初始化
```bash
$ cd /usr/local
$ sudo tar zxvf mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz
$ sudo ln -s /usr/local/mysql-5.7.32-linux-glibc2.12-x86_64 /usr/local/mysql
$ cd mysql
$ mkdir mysql-files
$ chown mysql:mysql mysql-files
$ chmod 750 mysql-files
$ bin/mysqld --initialize --user=mysql
2021-03-30T05:22:33.263441Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2021-03-30T05:22:33.498816Z 0 [Warning] InnoDB: New log files created, LSN=45790
2021-03-30T05:22:33.556445Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2021-03-30T05:22:33.625165Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: ee7b16a8-9117-11eb-9b2e-000c29afd275.
2021-03-30T05:22:33.626283Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2021-03-30T05:22:34.475675Z 0 [Warning] CA certificate ca.pem is self signed.
2021-03-30T05:22:34.878181Z 1 [Note] A temporary password is generated for root@localhost: 2x?gitEoAfts
$ bin/mysql_ssl_rsa_setup
$ bin/mysqld_safe --user=mysql &
# Next command is optional

```
初始化时如果mysqld不能定位目录位置，可采用如下方式：
```bash
bin/mysqld --initialize --user=mysql
  --basedir=/opt/mysql/mysql
  --datadir=/opt/mysql/mysql/data
```
# 2 安装后启动脚本优化（可选）

```bash
$ cp support-files/mysql.server /etc/init.d/mysql.server
$ chmod +x /etc/init.d/mysql
$ sudo systemctl daemon-reload
$ sudo /etc/init.d/mysql start
$ export PATH=$PATH:/usr/local/mysql/bin
```
# 3 修改初始化过程中生成的随机密码

```bash
$ mysql -u root -p
Enter password: (enter the random root password here)
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
```
如果初始化的时候使用的是mysqld --initialize-insecure，则此时root账户存在，连接服务器的时候不使用密码，通过下面方面设置密码：
```bash
shell> mysql -u root --skip-password
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
```
使用mysqladmin关闭服务：
```bash
shell> mysqladmin -u root -p shutdown
Enter password: (enter root password here)
```

