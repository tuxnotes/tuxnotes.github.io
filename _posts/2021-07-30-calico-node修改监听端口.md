---
layout: post
title: Kubernetes集群中calico-node Pod修改监听端口
date: 2021-07-30
author: tux
tags: calico
---

# 1 背景

默认情况下，calico-node监听端口为179，因为容器安全的问题，修改修改为高位的端口，如12020.假定Kubernetes集群已经正确部署了calico，以daemon-set方式运行的calico-node监听端口为179。

可以在宿主机上通过calicoctl二进制文件和yaml配置文件就绪修改。

# 2 修改方法

```bash
[root@vm-paasyy24-057 calico-typha-v3.18.1]# pwd
/apps/cluster_modules_setup/calico-typha-v3.18.1
[root@vm-paasyy24-057 calico-typha-v3.18.1]# ls
calico-bgpport.yaml  calicoctl  calico-typha-v3.18.1-prod.yaml
```
确保calicoctl具有执行权限

找打calico-node Pod对应的容器，可通过`docker ps`找到。
找到此容器在宿主机上的PID，命令如下：
```bash
[root@vm-paasyy24-057 sysadm]# docker inspect -f {{.State.Pid}} 36f201
24381
```
其中36f201是容器的ID。
进入容器的namespace
```bash
[root@vm-paasyy24-057 calico-typha-v3.18.1]# nsenter -t 24381 -n
```
更改配置：
yaml配置文件的名称为`calico-bgpport.yaml`，内容如下：
```yaml
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  listenPort: 12020
```
更改配置
```bash
[root@vm-paasyy24-057 calico-typha-v3.18.1]# ./calicoctl apply -f calico-bgpport.yaml 
Successfully applied 1 'BGPConfiguration' resource(s)
```
检查
```bash
[root@vm-paasyy24-057 calico-typha-v3.18.1]# ss -tln|grep 12020
LISTEN     0      8            *:12020                    *:*                  
[root@vm-paasyy24-057 calico-typha-v3.18.1]# exit
logout
[root@vm-paasyy24-057 calico-typha-v3.18.1]#
```
发现当前主机的监听端口已经改变。
需要注意的是，虽然calico-node是以daemon-set的方式运行的，在每个主机上都有运行，但此配置的更改只需要在一个容器内执行，验证如下：
```bash
[root@vm-paasyy24-057 calico-typha-v3.18.1]# ansible -i /home/sysadm/files/kubernetes_cluster_setup/hosts node -m shell -a "ss -tln|grep :12020"
10.248.24.61 | CHANGED | rc=0 >>
LISTEN     0      8            *:12020                    *:*                  
10.248.24.57 | CHANGED | rc=0 >>
LISTEN     0      8            *:12020                    *:*                  
10.248.24.59 | CHANGED | rc=0 >>
LISTEN     0      8            *:12020                    *:*                  
10.248.24.58 | CHANGED | rc=0 >>
LISTEN     0      8            *:12020                    *:*                  
10.248.24.60 | CHANGED | rc=0 >>
LISTEN     0      8            *:12020                    *:*                  
[root@vm-paasyy24-057 calico-typha-v3.18.1]# ansible -i /home/sysadm/files/kubernetes_cluster_setup/hosts node -m shell -a "ss -tln|grep :179"
10.248.24.61 | FAILED | rc=1 >>
non-zero return code
10.248.24.57 | FAILED | rc=1 >>
non-zero return code
10.248.24.60 | FAILED | rc=1 >>
non-zero return code
10.248.24.59 | FAILED | rc=1 >>
non-zero return code
10.248.24.58 | FAILED | rc=1 >>
non-zero return code
```
可以看到都监听在了12020端口，179端口已经不在监听。
