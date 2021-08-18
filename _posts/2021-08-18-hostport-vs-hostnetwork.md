---
layout: post
title: hostport vs hostnetwork
author: tux
date: 2021-08-18
tags: kubernetes
---

# 1 概念

在kubernetes生态中存在hostport,hostnetwork,nodeport,cluster ip等几个概念。其使用场景是不同的。

# 2 解释

nodeport,hostport,hostnetwork都是在kubernetes节点上暴露端口。nodeport是推荐的方式，其由kubernetes的service管理。其官方定义如下：
Kubernetes master will allocate a port from a flag-configured range (default: 30000–32767), and each Node will proxy that port。
意思就是kubernetes的master节点根据启动参数的配置(默认：30000-32767)，分配一个端口，每个节点都可以代理这个端口。

hostPort会强制在kubernetes节点上保留端口，具体取决于集群中CNI的实现。
尽量不要使用hostPort，除非你真的需要，比如节点上的守护进程。它会在宿主机上暴露指定的端口，当pod使用hostPort时，其能调度到的节点数量会因为端口冲突受限。因为每个<hostIP,hostPort,protocol>的组合必须唯一，如果没有明确的指定hostIP,与protocol，则kubernetes默认hostIP为0.0.0.0，默认协议为tcp。

而hostnetwork则让pod具备了访问节点网络namespace的能力，这与hostPort类似，但不取决于CNI。
hostnetwork控制pod可以使用宿主机的network namespace，这就使得pod可以访问loopback device, service监听在localhost，并且pod也可以监视同节点上其他pod的网络活动。

以下是stack overflow上的回答：

HostPort (nodes running a pod): Similiar to docker, this will open a port on the node on which the pod is running (this allows you to open port 80 on the host). This is pretty easy to setup an run, however:见[kubernetes.io](https://kubernetes.io/docs/concepts/configuration/overview/)

NodePort (On every node): Is restricted to ports between port 30,000 to ~33,000. This usually only makes sense in combination with an external loadbalancer (in case you want to publish a web-application on port 80)

Cluster IP (Internal only): As the description says, this will open a port only available for internal applications running in the same cluster. A service using this option is accessbile via the internal cluster-ip.

**hostport**

- When a pod is using a hostPort, a connection to the node’s port is forwarded directly to the pod running on that node
- pods using a hostPort, the node’s port is only bound on nodes that run such pods
- The hostPort feature is primarily used for exposing system services, which are deployed to every node using DaemonSets

**nodeport**

- With a NodePort service, a connection to the node’s port is forwarded to a randomly selected pod (possibly on another node)
- NodePort services bind the port on all nodes, even on those that don’t run such a pod

**cluster ip**

- Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.

## host ports and hostnetwork: the NATty gritty

kubernetes中的pod具有自己独立的ip地址和网络namespace，因此同一个主机的两个pod可以绑定相同的端口而不发生冲突。同样运行与宿主机的network namespace中的程序也绑定到与pod相同的端口也是可以的。

但有时我们需要将pod的端口绑定但主机的ip上，如果我们要这么做，那么我们要确保在pod调度后不会发生端口冲突。使用hostPort或者hostnetwork通常被认为是反模式，只有在特定的场景下才会用到。

还有nodeport service，但这种并不是直接暴露pod的端口。

### hostnetwork

hostnetwork允许pod允许在宿主机的network namespace中。通过pod的spec中的字段`hostNetwork: true`设置的。采用hostnetwork的pod在启动时与普通pod启动时的区别在于下面的两件事没有发生：

1. 为pod创建独立的network namespace
2. 允许CNI插件

hostnetwork模式运行pod访问宿主机的所有网卡，所以这种模式比较危险，pod可能在主机的network namespace干坏事。
hostnetwork的另一个负面作用是kubernetes的网络策略不能应用到启用了hostnetwork模式的pod。

hostnetwork的一个好处是不需要等到一些pod功能正常，pod才有连通性。只要主机的联通性没问题就可以。如果你有一些pod需要在pod的网络正常工作前就要启动，这hostnetwork就是一种必须的选择。

hostport是一种在宿主机上绑定端口的轻量级方式，允许在调度期间进行强制冲突检查。通过CNI插件的portmap实现，通过容器配置的ports部分的字段配置。

当配置了hostport的pod启动时，portmap插件在nat表中的prerouting和output链中添加如下iptables规则：

1. 目标为hostPort的任何流量都重定向到pod的ip和端口。例如如果hostPort是8080，pod的端口是8081，pod的ip为172.16.15.15，则到达宿主机8080端口的流量都会重定向到172.16.15.15:8081
2. a rule that looks for hairpin traffic (eg pod -> hostport -> back to pod) that marks traffic for MASQUERADE. without this rule the source address of the traffic going back to the pod be the pod's own IP.
3. a rule that looks for local traffic (eg localhost -> local address) that marks traffic for masquerading. without this rule the source address of the traffic going to the pod would be 127.0.0.1 which would route to the pod's own net namespace instead of the host's

host ports are a better option than hostNetwork if you can use them, but with one caveat that I think is worth mentioning. because the NAT is implemented in iptables, you won't see a socket listening on the host port being used which may be unexpected behavior if you don't understand what's happening. ie nc localhost <myport> will work fine, but if you look at ss -l nothing will show up for <myport> and you have to go to iptables to see what's actually "listening" on the host.

# 3 Reference

- https://stackoverflow.com/questions/50709001/difference-between-nodeport-hostport-and-cluster-ip
- https://medium.com/@maniankara/kubernetes-tcp-load-balancer-service-on-premise-non-cloud-f85c9fd8f43c
- https://kubernetes.io/docs/concepts/configuration/overview/
- https://lambda.mu/hostports_and_hostnetwork/
