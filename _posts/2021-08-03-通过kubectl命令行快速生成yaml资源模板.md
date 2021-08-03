---
layout: post
title: 通过kubectl命令行快速生成yaml资源模板
date: 2021-08-03
author: tux
tags: kubernetes
---

# 1 背景

有的时候需要在Kubernetes集群环境中进行调试，而手动从头编写想要的使用的yaml文件可能比较耗时，因此使用命令行先生成yaml的大致框架，然后在编辑就比较方便了。

# 2 示例

## 2.1 生成deployment

deployment的生成可以采用`kubectl create`命令，比如：
```bash
# kubectl create deploy test --image=busybox:latest --dry-run=client -o yaml > test.yaml
```
这样在test.yaml文件，得到自己想要的内容。

## 2.2 通过命令行启动pod

可以使用`kuebctl run`命令，如：
```bash
kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
```
使用`--overrides`选项可以进行默认配置的修改，比如设置`nodeSelector`将pod调度到特定节点：
```bash
kubectl run -ti --rm test --image=ubuntu:18.04 --overrides='{"spec": { "nodeSelector": {"nodename": "eks-prod-4"}}}'
```
如果想保留yaml文件，同样可以使用`--dry-run=client -o yaml`的方式重定向到一个文件。
