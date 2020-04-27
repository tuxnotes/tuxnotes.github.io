---
layout: post
date: 2020-04-27
title: docker构建debug
author: tux
tags: docker
---

### 问题描述
同一个dockerfile在相同的docker版本(包括server版本)进行`docker build`时，一个主机报错，而另一个主机正常构建。dockerfile内容如下：
```
FROM 10.x.x.x:5000/test
RUN rm -rf /opt/weblogic/user_projects/domains/domain1/test
```
其中报错的内容如下：
```
rm: cannot remove /opt/weblogic/user_projects/domains/domain1/test/WEB-INF: Directory not empty
```

### 问题定位
开始没有找到突破口，于是先执行了`docker rmi 10.x.x.x:5000/test`.结果报错：
```
Error response from daemon: conflict: unable to remove repository reference "10.x.x.x:5000/test"(must force) - container 69a4c44875d2 is using its referenced image aa53a6afeaed
```

根据报错内容目录不为空，先排查一下是否有容器在使用基础镜像，所以使用`docker ps -a`查看一下：
结果发现了状态为exited(1)的容器,容器ID是`69a4c44875d2`，使用的镜像是`aa53a6afeaed`.而镜像`aa53a6afeaed`正是`10.x.x.x:5000/test`的ID。

由于在另一台主机上构建成功，所以使用`docker info`查看一下docker相关的信息，发现了与成功构建的主机的不同，这里只列出重点信息：
```
Server Version: 1.13.1
Storage Driver: overlay2
 Backing filesystem: xfs
 Supports d_type: true(正常构建的) 有问题的主机此处为false
Kernel Version： 4.4.178-1.el7.elrepo.x86_64(正常构建) 有问题主机的内核版本为4.8.6-1.el7.elrepo.x86_64
```
从这里可以看出主要是xfs文件系统的`d_type`不支持导致的问题。
