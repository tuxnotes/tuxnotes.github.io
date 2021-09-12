---
layout: post
title: Debian中确定命令或文件属于哪个包
author: tux
date: 2021-09-12
tags: Debian
---

有时需要确定一个命令或文件属于哪个包，或者哪个包安装时产生的。对于Redhat发行版，使用rpm或yum provide来确定，对于Debian发行版，可以采用如下工具：

```bash
$ dpkg -S quux
$ dlocate quux
$ apt-file search quux
```
其中quux是需要查询的文件

Reference

- https://wiki.debian.org/WhereIsIt