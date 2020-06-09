---
layout: post
title: Ansible之Ad-hoc模式的坑
date: 2020-05-02
author: tux
tags: ansible
---

### 背景

有的时候只是简单的排查一下，就不需要写playbook了。在命令行执行一条ansible命令是非常方便的。比如检查磁盘根分区使用百分比的情况。如果大于80%就输出实际使用百分比(当然还有很多其他可能更简单的方式，这里只是其中一种)，不使用ansible命令，单独执行时命令如下：
```bash
df -h | grep root | awk '{if (substr($5,1,length($5-1)) + 0 > 80) print $5}'
```
如果将上述命令放到ansible的Ad-hoc模式，使用方式如下：
```bash
ansible mesos-slave -m shell -a "df -h | grep root | awk '{if (substr($5,1,length($5-1)) + 0 > 80) print $5}'"
```
则会出现如下报错：
```bash
awk: cmd. line:1: {if (substr(,1,length($5-1)) + 0 > 80) print }
awk: cmd. line:1:             ^ systex errornon-zero return code
```
解决办法也很简单，就是使用反斜杠对报错位置的`$5`进行转义，写法如下：
```bash
ansible mesos-slave -m shell -a "df -h | grep root | awk '{if (substr(\$5,1,length($5-1)) + 0 > 80) print $5}'"
```
一定要注意，在ansible的ad-hoc模式下使用特殊字符时，要注意转义。