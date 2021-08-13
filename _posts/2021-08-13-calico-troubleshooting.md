---
layout: post
title: calico troubleshooting
author: tux
date: 2021-08-13
tags: calico
---

# Troubleshooting and diagnostics

## Logs and diagnostics日志与诊断
使用`calicoctl`命令行工具进行诊断信息的收集,需要超级管理员权限
```bash
$ sudo calicoctl node diags
```
观察日志使用下面的命令
```bash
# kubectl logs -n kube-system <pod_name>
```
如果要观察calico组件的debug日志，需要通过环境变量设置`LogSeverityScreen`

## Containers do not have network connectivity容器网络不能连通
### Check for mismatched node names检查匹配错误的节点名称
如果你发现workload网络没有连通，检查以下主机名称是否正确的配置了。[node resource](https://docs.projectcalico.org/reference/resources/node)的名称必须匹配那台主机上[workload endpoint resources](https://docs.projectcalico.org/reference/resources/workloadendpoint)中的node name。如果名称不匹配，则在那个节点上的所有workload都不能接收到网络数据。

为了检查这项，查询b其中之一的broken workload endpoint并检查其node name：
```bash
calicoctl get workloadendpoints -n <namespace>
```
接着查看是否存在一个node resources
```bash
calicoctl get nodes
```
为了更正这个，你必须进行下面的步骤(下面是使用Kubernetes的示例)
1. 组织新的workload调度到这个有问题的节点
```bash
kubectl cordon mynode.internal.projectcalico.org
```
2. Drain all workloads from the node
```bash
kubectl drain mynode.internal.projectcalico.org --ignore-daemonsets
```
3. On the bad node, set the hostname to the desired value
```bash
sudo hostnamectl set-hostname <desired-hostname>
```
4. Delete the bad node configuration from calico
```bash
calicoctl delete node <name-of-bad-node>
```
5. restart calico/node on the bad node to pick up the change
```bash
kubectl delete pod -n kube-system <name-of-calico-pod>
```
6. reenable scheduling of workloads on the node
```bash
kubectl uncordon mynode.internal.projectcalico.org
```
为了防止这类问题的发生，建议在安装calico的时候，总是将`/var/lib/calico`目录挂载到容器的`calico/node`中。这允许所有的组件探测和使用相同的node name。更多信息参考[node name determination](https://docs.projectcalico.org/reference/node/configuration#node-name-determination)

### Check BGP peer status检查BGP同伴状态
如果同一个节点上的不同容器可以通信，容器与公网也可以通信，但是不同主机上的容器不能通信，那很可能是你的BGP配置有问题。
在每个主机上执行`calicoctl node status`，输出应该包含下面的内容
```bash
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+----------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+--------------+-------------------+-------+----------+-------------+
| 172.17.8.102 | node-to-node mesh | up    | 23:30:04 | Established |
+--------------+-------------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```
如果没有出现上面的内容，请检查下面两项

- 确保主机之间IP可以通信
- 确保你的网络运行在TCP 179端口请求BGP数据

### Configure NetworkManager配置NetworkManager

在使用calico网络之前需要配置NetworkManager
NetworkManager在默认的网络namespace中控制这网卡的路由表，而网卡是calico的veth pairs绑定的地方用于连接到容器。这会干预calico的agent路由的正确性。
在文件`/etc/NetworkManager/conf.d/calico.conf`添加如下内容，来防止NetworkManager的干预
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico
```
## Errors when running sudo calicoctl运行sudo calicoctl报错
如果你使用sudo运行如calicoctl node run等命令时，请记住你的环境变量不会传给sudo的环境，所以使用sudo时必须使用-E参数来抱哈环境变量
```bash
sudo -E calicoctl node run
```
或者给sudo命令设置环境变量
```bash
sudo ETCD_ENDPOINTS=http://172.25.0.1:2379 calicoctl node run
```
还要注意连接信息可以以配置文件的形式指定，而不是环境变量。

## Error: calico/node is not ready: BIRD is not ready: BGP not established with 10.0.0.1
大多数情况下，Kubernetes中的"unready"状态意味着集群中的某个peer不可达。检查两个peer之间BGP的连通性。
如果inactive node resources配置给了node-to-node mesh也会发生这种错误。参考[decommission the stale nodes](https://docs.projectcalico.org/maintenance/decommissioning-a-node)来解决这个问题。

当BGP要连接到的non-mesh peer宕机时也会发生这种错误。如果你的BGP拓扑中经常发生这种情况，你可以关闭BIRD的readiness检查。详情参考[node readiness](https://docs.projectcalico.org/reference/node/configuration#node-readiness)

## Linux conntrack table is out of space conntrack表没有空间了

当Linux系统中的conntrack表没有空间的时候，会导致严重的iptables性能问题。下面几种情况会发生这种问题：
- 当在一个给定的主机上运行大量的workload的时候
- 当workload创建很多TCP连接的时候
- workload创建了很多bidirectional UDP streams的时候

为了避免这种问题，使用下面的命令增减conntrack表的大小：
```bash
sysctl -w net.netfilter.nf_conntrack_max=1000000
echo "net.netfilter.nf_conntrack_max=1000000" >> /etc/sysctl.conf
```

