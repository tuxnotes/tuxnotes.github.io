---
layout: post
title: CentOS系统root逻辑卷扩容
date: 2020-04-16
author: tux
tags: lvm
---

### 扩容方法与流程

1. 对添加的硬盘进行分区
2. 创建pv
3. 扩展vg
4. 扩展lv
5. 调整大小

### 具体步骤

磁盘分区
```bash
# parted -s /dev/sdc mktable gpt mkpart primary xfs 0% 100% toggle 1 lvm
# partprobe /dev/sdc
```
`-s`选项，意思是非交互式，不提示。
关于分区设置为lvm的方法，还有如下方式：
```bash
# parted -s /dev/sdc mktable gpt mkpart primary xfs 0% 100% set 1 lvm on
```
创建pv
```bash
# pvcreate /dev/sdc1
```
扩展vg
```bash
# vgextend rhel /dev/sdc1
```
扩展lv
```bash
# lvextend -l +100%FREE /dev/rhel/root
```
上面的命令是添加lv所在的vg剩余所有空间，也可采用如下命令：
```bash
# lvextend -l 100%VG /dev/rhel /dev/rhel/root
```
注： VG：/dev/rhel，LV：/dev/rhel/root
调整大小
```bash
# xfs_growfs /dev/rhel/root
```
如果是ext文件系统，则使用如下命令：
```bash
# resize2fs /dev/rhel/root
```