---
layout: post
title: Kubernetes基础知识
date: 2020-10-09
author: tux
tags: kubernetes
---

>NOTE:本文参考[华为云文档](https://support.huaweicloud.com/basics-cce/kubernetes_0002.html)

# 1 容器与Kubernetes

## 1.1 容器

### 1 容器优点

相比于虚拟机，容器具有如下优点：

- 更高效的资源利用
- 更快速的启动时间
- 一致的运行环境
- 更轻松的迁移、维护和扩展

### 2 Docker容器典型的使用流程

Docker容器的主要概念：

- 镜像：Docker镜像包含了已打包的应用程序及其依赖的环境。它包含应用程序可用的文件系统和其他元数据，如镜像运行时的可执行文件路径
- 镜像仓库：Docker镜像仓库用于存放Docker镜像。
- 容器：一个运行中的容器就是一个运行在Docker主机上进程。

Docker容器典型使用流程如下图所示：

![使用流程](/assets/img/zh-cn-image-0258868444.png)

## 1.2 Kubernetes

### 1.2.1 是什么

kubernetes是一个部署和管理容器化应用，并对容器进行调度和编排的软件系统。

kubernetes提供了服务发现、伸缩、负载均衡、自愈设置选举等功能，让开发者从基础设施相关配置等解脱出来。

### 1.2.2 集群架构

Kubernetes集群包括master节点和node节点。应用部署在node节点上，且可以通过配置选择应用部署在某些特定节点上。kubernetes集群架构如下图所示：

![集群架构](/assets/img/cluster-arch.png)

#### master节点

master节点是集群的控制节点，主要由一下4个组件构成：

- API server：集群的入口，接受外部请求，并将信息写入etcd。是个组件相关通信的中转站。
- scheduler：调度应用，根据各种条件(如可用资源、节点亲和性等)将容器调度到node上。
- controller manager：执行集群级功能，如复制组件，跟踪node节点，处理节点故障等
- etcd：分布式数据存储，负责存储集群的配置信息

生产环境中，为了保证集群的高可用，通常会部署多个Master。

#### node节点

node节点是集群的计算节点，即运行容器化应用的节点。主要包含以下组件：

- kubelet: kubelet主要负责同container runtime打交道，并与API server交互，管理节点上的容器。
- kube-proxy:应用组件键的访问代理，解决节点上应用的访问问题。
- container runtime:容器运行时，最主要的功能是下载镜像和运行容器。

### 1.2.3 kubernetes的扩展性

kubernetes开放了容器运行时结构CRI，容器网络接口CNI和容器存储接口CSI，这些接口让kubernetes的扩展性最大化，而kubernetes本身则专注于容器调度。

### 1.2.4 kubernetes中的基本对象

kubernetes基本对象及其之间的关系如下图所示：

![对象及其关系](/assets/img/objects.png)

- pod:kubernetes创建或部署的最小单位。一个Pod分装一个或多个container，存储资源volume、一个独立的网络IP以及管理控制容器运行方式的策略选项。
- Deployment：对Pod的服务化封装，其可以包含一个或多个Pod，每个Pod的角色相同，所以系统会自动为Deployment的多个Pod分发请求
- StatefulSet：管理有状态应用。与Deployment相同的是，StatefulSet管理了基于相同容器定义的一组Pod；但与Deployment不同的是，StatefulSet为每个Pod维护了一个固定的ID。这些Pod基于相同的声明来创建的，但不能相互替换，无论如何调度，每个Pod都有一个永久不变的ID
- Job：控制批处理型任务对象。批处理业务与长期伺服业务(Deployment)的主要区别是批处理任务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置把任务成功完成就自动退出(Pod自动删除)
- CronJob:基于时间控制的job，类似于Linux系统的crontab，在指定的时间周期运行指定的任务。
- Daemonset:守护进程，在集群的每个节点都运行一个Pod，且保证只有一个Pod。这非常适合系统层面的应用，如日志收集，资源监控，这类应用需要每个节点都运行，且不需要太多实例，一个比较好的例子就是kubernetes的kube-proxy
- service:由于Pod的IP不是固定不变的，所以采用service来解决Pod访问的问题。service有一个固定IP，service将流量转发给Pod，且为这些Pod做负载均衡
- ingress:service是基于四层tcp和udp协议转发，Ingress可以基于七层的HTTP和https协议转发，可通过域名和路径做更细粒度的划分
- ConfigMap：用于存储应用所需的配置信息的资源类型，用于保存配置数据的键值对。通过ConfigMap可以方便的做到配置解耦，使得不同环境有不同配置
- Secret：一种加密存储的资源对象，可以将认证信息，证书，私钥等保存在Secret中，而不需要将这些敏感数据暴露到镜像或Pod定义中，从而更加安全和灵活。
- PersistentVolume(PV):PV持久化数据存储卷，主要定义的是一个持久化存储在宿主机上的目录，如一个NFS挂在目录。
- PersistentVolumeClaim(PVC):kubernetes提供PVC专门用于持久化存储的申请，PVC可以让开发者无需关心底层存储资源如何创建、释放等动作，而只需要申明需要如何类型的存储资源、多大的存储空间。

### 1.2.5 kubernetes对象的描述

kubernetes中资源可使用yaml描述，也可以使用json。其内容有一下四个部分组成：

- typeMeta:对象类型的元信息，声明对象使用哪个API版本，哪个类型的对象
- objectMeta:对象的元信息，包括对象名称、使用的标签等
- spec：对象期望的状态，如使用什么镜像，有多少副本等
- status：对象的实际状态，只能在对象创建后看到，创建对象时无需指定

YAML描述文件如下图所示：

![YAML描述](/assets/img/object-yaml.png)

### 1.2.6 在kubernetes上运行应用

将1.2.5节中图片的status部分去掉，保存为一个nginx-deployment.yaml的文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  lables:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
```
使用kubectl工具进行部署：
```bash
# kubectl create -f nginx-deployment.yaml
deployment.apps/nginx created
```
使用如下命令查询Deployment和Pod
```bash
# kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           9s

# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-685898579b-qrt4d   1/1     Running   0          15s
nginx-685898579b-t9zd2   1/1     Running   0          15s
nginx-685898579b-w59jn   1/1     Running   0          15s
```
# 2 Pod,Label和Namespace

## 2.1 Pod:kubernetes中的对象调度对象

### 2.1.1 什么是Pod

Pod是kubernetes创建或部署的最小单位，封装一个或多个容器，容器间共享存储资源volume和网络空间以及控制容器运行方式的策略。

Pod使用的两种方式：

- Pod中运行一个容器。这是kubernetes最常见的用法，但kubernetes是直接管理Pod而不是容器
- Pod中运行多个需要耦合在一起工作、需要共享资源的容器。此场景下应用包含一个主容器和几个辅助容器(sidecar container).如主container是一个web服务，从一个笃定目录下对外提供文件服务，而辅助容器周期性的从外部下载文件存到这个固定目录。如下图所示：


![pod](/assets/img/pod.png)

实际中很少直接创建Pod，而是使用kubernetes中称为controller的抽象层来管理Pod实例，如Deployment和Job。controller可以创建和管理多个Pod，提供副本管理、滚动设计和自愈能力。通常controller会使用Pod template来创建相应的Pod。

### 2.1.2 创建Pod

下面的示例描述了一个名为nginx的Pod。其包含一个名为container-0的容器，使用镜像nginx:alpine,使用资源为0.1核CPU、200MB内存：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: container-0
    image: nginx:alpine
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
   imagePullSecrets:       # 拉取镜像使用的证书，在CCE上必须为default-secret
   - name: default-secret
```
### 2.1.3 使用环境变量

环境变量是容器运行环境中设定的一个变量。环境变量为应用程序提供极大的灵活性，可以在应用程序中使用环境变量，在创建容器时为环境变量赋值，容器运行时读取环境变量的值，从而做到灵活的配置，而不是每次都重新编写应用程序制作镜像。**环境变量通过配置spec.containers.env字段来使用**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: container-0
    image: nginx:alpine
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    env:
    - name: env_key
      value: env_value
  imagePullSecrets:
  - name: default-secret
```
查看环境变量：
```bash
# kubectl exec -it nginx -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
TERM=xterm
env_key=env_value
```
环境变量还可以引用ConfigMap和secret.

### 2.1.4 容器启动命令

启动容器就是启动主进程，但有时启动主进程前需要一些准备工作。如MySQL类的数据库可能需要一些数据库配置，初始化的工作，这些工作要在最终的MySQL服务器运行前做完。这些操作可以在制作镜像时通过在dockerfile文件中设置ENTRYPOINT或CMD来完成。如下Dockerfile中设置了ENTRYPOINT["top","-b"]命令，其将会在容器启动时执行。

```
FROM ubuntu
ENTRYPOINT ["top","-b"]
```
实际使用时，值需要配置Pod的containers.command字段，该参数是list类型，第一个参数为执行的命令，后面均为命令的参数。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: container-0
    image: nginx:alpine
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    command:
    - top
    - b
```
### 2.1.5 容器的生命周期

Kubernetes提供了容器生命周期回调，在容器生命周期的特定阶段执行调用，比如容器在停止前希望执行某项操作，就可以注册响应的回调函数。目前提供的声明周期回调函数如下：

- PostStart: 此回调在创建容器之后立即执行。但不能保证回调会在容器ENTRYPOINT之前执行。没有参数传递给处理程序。
- PreStop: 在容器因为API请求或管理事件(诸如存活探针失败、资源抢占、资源竞争等)而被终止前，此回调会被调用。如果容器已经处于终止或完成状态，则PreStop回调的调用将失败。此调用是阻塞的，也是同步调用，因此必须在删除容器的调用之前完成。

实际使用时，只需要配置Pod的lifecycle.postStart或lifecycle.preStop参数，如下所示：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: container-0
    image: nginx:alpine
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    lifecycle:
      postStart: # 启动后处理
        exec:
          command:
          - "/postStart.sh"
      preStop: # 停止前处理
        exec:
          command:
          - "/preStop.sh"
   imagePullSecrets:
   - name: default-secret
  ```
## 2.2 存活探针(Liveness Probe)
### 2.2.1 存活探针
kubernetes提供了自愈能力，具体就是能感知到容器崩溃，然后能重启这个容器。但有的应用，如Java程序内存泄漏了，程序无法正常工作，但jvm进程却是一直运行的，对于这种应用本省业务出了问题的情况，kubernetes提供了liveness Probe机制，通过检测容器影响是否正常来决定是否重启，这是一种健康检查机制。

毫无疑问，每个Pod最好都定义Liveness Probe，否则kubernetes无法感知Pod是否正常运行。

kubernetes支持如下三种探测机制：
- HTTP GET: 向容器发送HTTP GET请求，如果Probe收到2xx或3xx，说明容器是健康的
- TCP Socket: 尝试与容器指定端口建立TCP连接，如果连接成功建立，说明容器是健康的
- Exec: Probe执行容器中的命令并检查命令退出的状态码，如果状态码为0则说明容器是健康的

与存活探针对应的还有一个就绪探针Readiness Probe,在kubernetes网络-->就绪探针部分介绍。

### 2.2.2 HTTP GET

定义方法如下：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: nginx:alpine
    livenessProbe:
      httpGet:
        path: /
        port: 80
```

### 2.2.3 TCP Socket

定义方法如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp
  labels:
    test: liveness
spec:
  containers:
  - name: liveness
    image: nginx:alpine
    livenessProbe:
      tcpSocket:
        port: 80
```
### 2.2.4 Exec
定义方法如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: livess
    image: nginx:alpine
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
```
上面的定义在容器中执行cat /tmp/healthy命令，如果成功执行并返回0，则说明容器是健康的。上面定义中，30秒后命令会删除/tmp/healthy，这导致liveness Probe判定Pod处于不健康状态，然后会重启容器。

### 2.2.5 Liveness Probe高级配置

上面liveness-http的describe命令有输出如下内容：
```
Liveness: http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
```
这一行表示Liveness Probe的具体参数配置，含义如下：
- delay:延迟，delay=0表示容器启动后立即开始探测，没有延迟时间
- timeout：超时，timeout=1s表示容器必须在1s内进行响应，否则这次探测记作失败
- period:周期，period=10s,表示每10s探测一次容器
- success:成功，#success=1,表示连续1次成功后记作成功
- failure：失败，#failure=3，表示连续3此失败后会重启容器

以上存活探针表示：容器启动后立即进行探测，如果1s内容器没有给出回应则记作探测失败。每次间隔10s进行一次探测，在探测连续失败3次后重启容器。

这些是创建是默认配置的，也可以手动配置，如：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
  - image: k8s.gcr.io/liveness
    livenessProbe:
      httpGet:
        path: /
        port: 8080
    initialDelaySeconds: 10 # 容器启动多久开始探测
    timeoutSeconds: 2   # 表示容器必须在2s内做出相应反馈给probe，否则视为探测失败
    periodSeconds: 30   # 探测周期，每30s探测一次
    successThreshold: 1 # 连续探测1次成功，则判定为成功
    failureThreshold: 3 # 连续探测3次失败，则判断为失败
    ```
initialDelaySeconds一般要设置为大于0，因为很多情况下容器已启动成功，但应用就绪也需要一定时间，需要等待就绪时间之后才能返回成功，否则就会导致probe经常失败。

另外failureThreshold可以设置多次循环探测，这样在实际应用中健康检查的程序就不需要多次循环，这一点在开发应用是需要注意。

### 2.2.6 配置有效的Liveness Probe

- Liveness Probe应该检查什么

好的Liveness Probe应该检查应用内部所有的关键部分是否健康，并使用一个专有的URL访问，如/health,当访问/health时，执行这个功能，然后返回对应结果。这里要注意不能做鉴权，否则Probe可能会一直失败导致陷入重启的死循环

另外检查只能限制在应用内部，不能检查依赖外部的部分。如当前web server不能连接数据库时，这个就不能看成web server不健康了。

- Liveness Probe必须轻量

Liveness Probe不能占用过多资源，且不能占用过长时间，否则所有资源都在做健康检查就没意义了。如Java应用，最好用HTTP GET方式，如果用Exec方式，jvm启动就占用了非常多的资源

## 2.3 Label:组织Pod的利器
### 2.3.1 为什么需要Label
当资源变得非常多的时候，如何分类管理就非常重要了。kubernetes提供了一种机制来为资源分类，那就是Label(标签)。Label非常简单，但却很请打，kubernetes中几乎所有的资源都可以用Label来组织。
Label的具体形式是key-value的标记对，可以在创建资源的时候设置，也可以在后期添加和修改。
以Pod为例，当Pod变得多起来后，就显得杂乱且难以管理，没有分类组织的Pod如下图所示：

![](/assets/img/pod-without-label.png)

如果为Pod打上不同的标签，那情况就完全不同了，使用Label组织的Pod如下图所示：

![](/assets/img/pod-with-label.png)

### 2.3.2 添加Label
Label的形式为key-value形式，使用非常简单，如下，为Pod设置了app=nginx和env=prod两个Label

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:  # 为Pod设置两个Label
    app: nginx
    env: prod
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
```

Pod有了Label后，在查询Pod的时候带上--show-labels就可以看到Pod的Label
```bash
$ kubectl get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          50s   app=nginx,env=prod
```
还可以使用-L只查询某些特定的Label
```bash
$ kubectl get pod -L app,env
NAME              READY   STATUS    RESTARTS   AGE   APP     ENV
nginx             1/1     Running   0          1m    nginx   prod
```
对已存在的Pod，可以直接使用kubectl label命令直接添加Label
```bash
$ kubectl label po nginx creation_method=manual
pod/nginx labeled

$ kubectl get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          50s   app=nginx, creation_method=manual,env=prod
```
### 2.3.3 修改Label
对于已存在的Label，如果需要修改的话，需要在命令中带上--overwrite选项：
```bash
$ kubectl label po nginx env=debug --overwrite
pod/nginx labeled

$ kubectl get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          50s   app=nginx,creation_method=manual,env=debug
```

## 2.4 Namespace:资源分组
### 2.4.1 为什么需要namespace
Label虽好，但只用Label的话，则Label会非常多，有时候会重叠。且每次查询之类的动作都会带一堆的Label，非常不方便。Kubernetes提供了namespace来做资源组织和划分，使用namespace可以将包含很多组件的系统分成不同的组。namespace也可以用来做多租户划分，这样多个团队就可以共用一个集群，使用的资源使用namespace划分开。
不同的namespace下可以有相同的名字，kubernetes中大部分资源可以用namespace划分，不过有些资源不行，它们属于全局资源，不属于namespace。
使用下面的命令查询当前集群拥有的namespace：
```bash
# kubectl get ns
```
到目前位置，都是在名为default的namespace下操作，当使用kubectl get而不指定namespace时，默认为default namespace,下面的命令查询kube-system namespace下的Pod：
```bash
kubectl get po -n kube-system
```
可以看到kube-system有很多Pod，其中coredns用于服务发现，everest-csi用于华为对接华为云存储服务，icagent用于对接华为云监控系统。
这些通用的、必须的应用放到kube-system这个namespace中，以便与其他Pod之间隔离。其他namespace不会看到kube-system这个namespace中的东西，不会造成影响。

### 2.4.2 创建namespace
使用如下yaml定义namespace:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
  ```
使用命令行方式创建namespace的方式如下：
```bash
kubectl create namespace custom-namespace
```
### 2.4.3 namespaced的隔离说明
namespace只能做到组织上划分，对运行的对象来说，不能做到真正的隔离。举例来说，如果两个namespace下的Pod知道对方的IP，而kubernetes依赖的底层网络没有提供namespace之间的网络隔离的话，那这两个Pod就可以相互访问。

# 3 Pod的编排与调度

## 3.1 Deployment
### 3.1.1 什么是Deployment
Pod是kubernetes创建和部署的最小单位，但是Pod是被设计为相对短暂的一次性实体，Pod可以被驱逐(当节点资源不足时)、随着集群的节点崩溃而消失。kubernetes提供了Controller来管理Pod，Controller可以创建和管理多个Pod，提供副本管理、滚动升级和自愈能力，其中最为常用的就是Deployment。
**一个Deployment可以包含一个或多个Pod副本，每个Pod副本的角色相同**,所以系统会自动为Deployment的多个Pod副本分发请求。
Deployment集成了上线部署、滚动升级、创建副本、恢复上线的功能，在某种程度上，Deployment实现无人值守的上线，大大降低了上线过程的复杂性和操作风险。

### 3.1.2 创建Deployment
下面的实例创建一个名为nginx的Deployment，使用nginx:lasest镜像创建两个Pod，每个Pod占用100m core CPU、200M内存。

```yaml
apiVersion: apps/v1 # 注意这里与Pod的区别，是apps/v1
kind: Deployment    # 资源类型
metadata:
  name: nginx       # Deployment的名称
spec:
  relicas: 2        # Pod的数量，Deployment会确保一直有2个Pod运行
  selector:         # Label Selector
    matchLabels:
      app: nginx
  template:         # Pod的定义，用于创建Pod，也称为Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:latest
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
```
从定义中可以看到Deployment的名称为nginx，spec.replicas定义了Pod的数量，即这个Deployment控制2个Pod；spec.selector是Label Selector(标签选择器)，表示这个Deployment会选择Label为app=nginx的Pod；spec.template是Pod的定义，内容与Pod中的定义完全一致。将上面的定义保存到deployment.yaml文件中，使用kubectl创建这个Deployment。

```bash
kubectl craete -f deployment.yaml
```
查看Deployment的Pod
```bash
$ kubectl get deploy
NAME           READY     UP-TO-DATE   AVAILABLE   AGE
nginx          2/2       2            2           4m5s
```
可以看到**READY**值为2/2，前一个2表示当前有2个Pod运行，后一个2表示期望有2个Pod；**AVAILABLE**为2表示有2个Pod是可用的。
### 3.1.3 Deployment如何控制Pod
继续查询Pod
```bash
$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7f98958cdf-tdmqk   1/1       Running   0          13s
nginx-7f98958cdf-txckx   1/1       Running   0          13s
```
如果删掉一个Pod，立马会有一个新的Pod被创建出来，如下所示，这就是前面所说的Deployment会确保有2个Pod在运行。如果删掉一个，Deployment会重新创建一个，如果某个Pod故障或其他有问题，Deployment会自动拉起这个Pod：
```bash
$ kubectl delete pod nginx-7f98958cdf-txckx

$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7f98958cdf-tdmqk   1/1       Running   0          21s
nginx-7f98958cdf-tesqr   1/1       Running   0          21s
```
如上所示，有两个名为nginx-7f98958cdf-tdmqk和nginx-7f98958cdf-tesqr的Pod，Pod的名称中用短线分割，第一部分为nginx是Deployment的名称，-7f98958cdf-tdmqk和-7f98958cdf-tesqr是kubernetes随机生成的后缀。
你也许已经发现这两个后缀中的前面一部分是相同的，都是7f98958cdf，这是因为Deployment不是直接控制Pod的，Deployment通过一种名为ReplicaSet的控制器控制Pod，使用如下命令查询ReplicaSet，简写为rs。
```bash
$ kubectl get rs
NAME               DESIRED   CURRENT   READY     AGE
nginx-7f98958cdf   2         2         2         1m
```
ReplicaSet的名称为nginx-7f98958cdf,后-7f98958cdf也是随机生成的。
Deployment控制Pod的方式如下图所示，Deployment控制ReplicaSet，ReplicaSet控制Pod。

![](/assets/img/rs.png)

如果使用kubectl describe命令查看Deployment的详情，可以看到有一行NewReplicaSet: nginx-7f98958cdf (2/2 replicas created)，而且Events里面事件确是把ReplicaSet的实例扩容到2个。在实际使用中您也许不会直接操作ReplicaSet，但了解Deployment通过控制ReplicaSet来控制Pod会有助于定位问题。

```bash
$ kubectl describe deploy nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Sun, 16 Dec 2018 19:21:58 +0800
Labels:                 app=nginx

...

NewReplicaSet:   nginx-7f98958cdf (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  5m    deployment-controller  Scaled up replica set nginx-7f98958cdf to 2
```

### 3.1.4 升级
实际应用中，升级是一个常见场景，Deployment能方便的支持应用升级。Deployment可以设置不同的升级策略，有如下两种：

- RollingUpdate:滚动升级，即逐步创建新Pod在删除旧Pod，为默认策略
- Recreate: 替换升级，即先把当前Pod删掉再重新创建Pod

Deployment的升级可以是声明式的，即只需要修改Deployment的YAML定义即可。
Deployment通过maxSurge和maxUnavailable两个参数控制升级过程中同时重新创建Pod的比例，很多时候这非常有用：

```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
```
- maxSurge: 与Deployment中spec.replicas定义数量相比，可以有多个个Pod存在，默认是25%，比如spec.replicas为4，那升级过程中就不能超过5个Pod存在，即按1个的步伐升级，实际升级过程中会换算成数字，且换算会向上取整。这个值也是可以直接设置成数字。
- maxUnavailabel: 与Deployment中spec.replicas相比，可以有多少个Pod失效，也就是删除的比例。默认是25%，比如spec.replicas为4，那升级过程中就至少有3个Pod存在，即删除Pod的步伐是1。同样这个值也可以设置成数字。

在前面的例子中，由于spec.replicas是2，如果maxSurge和maxUnavailable都为默认值25%，那实际升级过程中，maxSurge允许最多3个Pod存在（向上取整，2 X 1.25=2.5，取整为3），而maxUnavailable设置为0，则不允许有Pod Unavailable，也就是说在升级过程中，一直会有2个Pod处于运行状态，每次新建一个Pod，等这个Pod创建成功后再删掉一个旧Pod，直至Pod全部为新Pod。

### 3.1.5 回滚

回滚也称为回退，即当发现升级出现问题时，让应用回到老的版本。Deployment可以非常方便的回滚到老版本。可以执行kubectl rollout undo命令进行回滚。

```bash
$ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
```
Deployment之所以能如此容易的做到回滚，是因为Deployment是通过ReplicaSet控制Pod的，升级后之前ReplicaSet都一直存在，Deployment回滚做的就是使用之前的ReplicaSet再次把Pod创建出来。Deployment中保存ReplicaSet的数量可以使用revisionHistoryLimit参数限制，默认值为10。

## 3.2 StatefulSet
### 3.2.1 为什么需要StatefulSet
前面提到Deployment控制器下的Pod都有个共同点，就是每个Pod除了名称和IP不同，其余完全相同。需要的时候通过Pod模板创建Pod，不需要的时候Deployment可以删除任意Pod。
但某些场景下，这并不满足需求，比如有些分布式的场景，要求每个Pod都有自己单独的状态时，比如分布式数据库，每个Pod要求有独立的存储，这时Deployment就不能满足需求了。
详细分析一下有状态应用的需求，分布式有状态的特点主要是应用中每个部分的角色不同(即分工不同),比如数据库有准备，Pod之前有依赖，对应到kubernetes中就是对Pod有如下要求：

- Pod能够被别的Pod找到，这就要求**Pod有固定的标识**
- **每个Pod有单独存储**，Pod被删除恢复后，读取的数据必须还是以前那份，否则状态就会不一致。

kubernetes提供了StatefulSet来解决这个问题，具体如下：
1. StatefulSet**给每个Pod提供固定名称**，Pod名称增加从0-N的固定后缀，Pod重新调度后Pod的名称和hostname不变
2. StatefulSet通过headless service给每个Pod提供固定的访问域名
3. StatefulSet通过创建固定标识的PVC保证Pod重新调度后还是能访问到相同的持久化数据,如下图所示：

![](/assets/img/stateful-pvc.png)

下面通过创建StatefulSet来体验这些特性

### 3.2.2 创建Headless Service
创建StatefulSet需要一个headless service用于访问Pod，service的概念会在后面介绍，这里先介绍headless service的创建方法。使用如下文件描述headless service：

```yaml
apiVersion: v1
kind: service # 对象类型为Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - name: nginx  # Pod间通信的端口名称
      port: 80     # Pod间通信的端口号
  selector:
    app: nginx     # 选择标签为app: nginx的Pod
  clusterIP: None  # 必须设置为None，表示headless service
```
执行如下命令进行创建headless service：
```bash
kubectl create -f headless.yaml
```

### 3.2.3 创建StatefulSet
StatefulSet的YAML定义与其他对象基本相同，主要有两个差异点;
- serviceName指定了StatefulSet使用哪个headless service，需要填写headless service的名称
- volumeClaimTemplate用来申请持久化声明PVC，这里定义了一个名为data的模板，它为每个Pod创建一个PVC，storageClassName指定了持久化存储的类型；volumeMounts是为Pod挂载存储。当然如果不需要存储的话可以删除volumeClaimTemplate和volumeMounts字段
```yaml
apiVersion: v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: nginx    # headless service的名称
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: container-0
        image: nginx-alpine
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:                        # Pod挂载的存储
        - name: data
          mountPath: /usr/share/nginx/html   # 存储挂载到/usr/share/nginx/html
      imagePullSecrets:
      - name: default-secret
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests；
          storage: 1Gi
      storageClassName: csi-nas      # 持久化存储的类型
```
使用`kubectl create -f statefulset.yaml`创建之后，查询StatefulSet和Pod，可以看到Pod的名称后缀从0开始到2，逐个递增。
```bash
# kubectl get statefulset
NAME    READY   AGE
nginx   3/3     107s

# kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
nginx-0   1/1     Running   0          112s
nginx-1   1/1     Running   0          69s
nginx-2   1/1     Running   0          39s
```
此时如果手动删除nginx-1这个Pod。再次查询Pod，可以看到StatefulSet重新创建了一个名称相同的Pod，通过创建时间5s可以看出nginx-1是刚刚创建的。
```bash
# kubectl delete pod nginx-1
pod "nginx-1" deleted

# kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
nginx-0   1/1     Running   0          3m4s
nginx-1   1/1     Running   0          5s
nginx-2   1/1     Running   0          1m10s
```
进入容器查看容器的hostname，可以看到同样是nginx-0,nginx-1和nginx-2
```bash
# kubectl exec nginx-0 -- sh -c 'hostname'
nginx-0
# kubectl exec nginx-1 -- sh -c 'hostname'
nginx-1
# kubectl exec nginx-2 -- sh -c 'hostname'
nginx-2
```
同时可以看一下StatefulSet创建的PVC，可以看到这些PVC，都以"PVC名称-StatefulSet名称-编号"的方式命名，且处于Bound状态。
```bash
# kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-nginx-0   Bound    pvc-f58bc1a9-6a52-4664-a587-a9a1c904ba29   1Gi        RWX            csi-nas        2m24s
data-nginx-1   Bound    pvc-066e3a3a-fd65-4e65-87cd-6c3fd0ae6485   1Gi        RWX            csi-nas        101s
data-nginx-2   Bound    pvc-a18cf1ce-708b-4e94-af83-766007250b0c   1Gi        RWX            csi-nas        71s
```

### 3.2.4 StatefulSet的网络标识
StatefulSet创建后，可以看到Pod是有固定名称的，那headless service是如何起作用的呢，那就是使用DNS，为Pod提供固定的域名，这样Pod间就可以通过域名访问，即便Pod被重新创建而导致的Pod的IP地址变化，这个域名也不发生变化。
headless service创建后，每个Pod的IP都会有下面格式的域名。
**<pod-name>.<svc-name>.<namespace>.svc.cluster.local**
例如上面三个Pod的域名如下：
- nginx-0.nginx.default.svc.cluster.local
- nginx-1.nginx.default.svc.cluster.local
- nginx-2.nginx.default.svc.cluster.local

由于在同一个namespace下，实际访问时可以省略后面的.<namespace>.svc.cluster.local
下面命令会使用tutum/dnsutils镜像创建一个Pod，进入这个Pod的容器，使用nslookup查看Pod对应的域名，可以发现能解析出Pod的IP地址。这里可以看到DNS服务器的地址是10.247.3.10，这是创建CCE集群时迷人安装的CoreDNS插件，用于提供DNS服务。
```bash
$ kubectl run -i --tty --image tutum/dnsutils dnsutils --restart=Never --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx-0.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-0.nginx.default.svc.cluster.local
Address: 172.16.0.31

/ # nslookup nginx-1.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-1.nginx.default.svc.cluster.local
Address: 172.16.0.18

/ # nslookup nginx-2.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-2.nginx.default.svc.cluster.local
Address: 172.16.0.19
```
此时如果手动删除这两个Pod，查询被StatefulSet重新创建的PodIP，然后使用nslookup命令解析Pod的域名，可以发现nginx-0.nginx和nginx-1.nginx仍然能解析到对应的Pod，这就保证了StatefulSet网络标识不变。

### 3.2.5 StatefulSet存储状态
前面提到StatefulSet可以通过PVC做持久化存储，保证Pod重新调度后还是能访问到相同的持久化数据，在删除Pod时，PVC不会被删除。StatefulSet的Pod重建过程如下图所示：

![](/assets/img/pod-recreate.png)

下面通过实际操作验证这一点是如何做到的，执行下面的命令，在nginx-1的目录/usr/share/nginx/html中写入一些内容，例如将index.html的内容修改为"hello world"
```bash
# kubectl exec nginx-1 -- sh -c 'echo hello world > /usr/share/nginx/html/index.html'
```
修改完后，如果在Pod中访问"http://localhost",就会返回"hello world"
```bash
# kubectl exec -it nginx-1 -- curl localhost
hello world
```
此时如果手动删除nginx-1这个Pod，然后再次查询Pod，可以看到StatefulSet重新创建了一个名称相同的Pod，通过创建时间4s可以看出nginx-1是刚刚创建的。
```bash
# kubectl delete pod nginx-1
pod "nginx-1" deleted

# kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
nginx-0    1/1     Running   0          14m
nginx-1    1/1     Running   0          4s
nginx-2    1/1     Running   0          13m
```
再次访问该Pod的index.html页面，会发现仍然返回"hello world",这说明这个Pod仍然是访问相同的存储。
```bash
# kubectl exec -it nginx-1 -- curl localhost
hello world
```

## 3.3 Job和CronJob
Job和CronJob是负责批量处理短暂的一次性任务(short lived one-off tasks),即执行一次的任务，它保证批处理任务的一个或多个Pod成功结束。
- Job：是kubernetes用来控制批处理型任务的资源对象。批处理业务与长期伺服业务(Deployment、StatefulSet)的主要区别是批处理任务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置包任务成功完成就自动退出(Pod自动删除)
- CronJob:是基于时间的Job，就类似于Linux系统的crontab文件中的一行，在指定的时间周期运行指定的Job。

任务负载的这种用完即停止的特性特别适合一次性任务，比如持续集成。

### 3.3.1 创建Job
以下是一个Job配置，其计算π到2000位并打印输出。Job结束需要运行50个Pod，这个示例中就是打印π50次，并行运行5个Pod，Pod如果失败，则最多重试5次

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  completions: 50           # 运行的次数，即job结束需要成功运行的Pod的个数
  parallelism: 5            # 并行运行Pod的数量，默认为1
  backoffLimit: 5           # 表示失败Pod的重试最大次数，超过这个次数不会继续重试
  activeDeadlineSeconds: 10 # 表示Pod超时时间，一旦达到这个时间，job及其所有的Pod都会停止
  template:                 # Pod定义
    spec:
      containers:
      - name: pi
        image: perl
        command:
        - perl
        - "-Mbignum=bpi"
        - "-wle"
        - print bpi(2000)
      restartPolicy: Never
```
根据completions和parallelism的设置，可以将job划分为以下几种类型：

| Job类型 | 说明 | 使用示例 |
|:--- |:--- |:---|
|一次性Job|创建一个Pod直到其成功结束|数据库迁移|
|固定结束次数的Job|依次创建一个Pod运行直到completions个成功结束|处理工作队列的Pod|
|固定结束次数的并行Job|依次创建多个Pod运行直到completions个成功结束|多个Pod同时处理工作队列|
|并行Job|创建一个或多个Pod直到有一个成功结束 | 多个Pod同时处理工作队列|

### 3.3.2 创建CronJob
相比Job，CronJob就是加了定时的Job，CronJob执行时是在指定的时间创建出Job，然后由Job创建出Pod。

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-example
spec:
  schedule: "0,15,30,45 * * * *"     # 定时相关配置
  jobTemplate:                       # Job的定义
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: main
            image: pi
```
CronJob的格式从前到后是：
- Minute
- Hour
- Day
- Month
- week

如 "0,15,30,45 * * * * " ，前面逗号隔开的是分钟，后面第一个* 表示每小时，第二个 * 表示每个月的哪天，第三个表示每月，第四个表示每周的哪天。

*如果你想要每个月的第一天里面每半个小时执行一次，那就可以设置为" 0,30 * 1 * * " 如果你想每个星期天的3am执行一次任务，那就可以设置为 "0 3 * * 0"。

## 3.4 DaemonSet
DaemonSet是这样一种对象(守护进程)，它在集群的每个节点上运行一个Pod，且保证只有一个Pod，这非常适合一些**系统层面的应用**,例如日志收集、资源监控等，这类应用需要每个节点都运行，且不需要太多实例，一个典型的例子就是kubernetes的kube-proxy.
DaemonSet跟节点相关，如果节点异常，也不会在其他节点重新创建。
DaemonSet的yaml定义如下所示：
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  labels:
    app: nginx-daemonset
spec:
  selector:
    matchLabels:
      app: nginx-daemonset
  template:
    metadata:
      labels:
        app: nginx-daemonset
    spec:
      nodeSelector: # 节点选择，当节点拥有daemon=need时候才在节点上创建Pod
        daemon: need
      containers:
      - name: nginx-daemonset
        image: nginx:alpine
        resources:
          limits:
            cpu: 250m
            memory: 512Mi
          requests:
            cpu: 250m
            memory: 512Mi
```
这里可以看出没有Deployment或StatefulSet中的replicas参数，因为是每个节点固定一个。
Pod模板中有个nodeSelector，指定了只在有“daemon=need”的节点上才创建Pod，如下图所示，DaemonSet只在指定标签的节点上创建Pod。如果需要在每一个节点上创建Pod可以删除该标签。DaemonSet在指定标签的节点上创建Pod，如下图所示：

![](/assets/img/ds.png)

如果修改掉192.168.0.94节点的标签，可以发现DaemonSet会删除这个节点上的Pod。

```bash
$ kubectl label node 192.168.0.94 daemon=no --overwrite
node/192.168.0.94 labeled

$ kubectl get ds
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx-daemonset   1         1         1       1            1           daemon=need     4m5s

$ kubectl get pod -owide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE
nginx-daemonset-g9b7j   1/1     Running   0          2m23s   172.16.3.0   192.168.0.212
```

## 3.5 亲和与反亲和调度
在DaemonSet中提到使用nodeSelector选择Pod要部署的节点，其实kubernetes还支持更精细、更灵活的调度机制，那就是亲和和反亲和调度。
**kubernetes支持节点和Pod两个层级的亲和和反亲和**。通过配置亲和与反亲和规则，可以允许你指定硬性限制或偏好，例如将前台Pod和后台Pod部署在一起、某类应用部署到某些特定的节点、不同应用部署到不同的节点等等。
### 3.5.1 Node Affinity(节点亲和)
**亲和的基础也是标签**,先看一下CCE集群中节点上有什么标签：
```bash
$ kubectl describe node 192.168.0.212
Name:               192.168.0.212
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/is-baremetal=false
                    failure-domain.beta.kubernetes.io/region=cn-east-3
                    failure-domain.beta.kubernetes.io/zone=cn-east-3a
                    kubernetes.io/arch=amd64
                    kubernetes.io/availablezone=cn-east-3a
                    kubernetes.io/eniquota=12
                    kubernetes.io/hostname=192.168.0.212
                    kubernetes.io/os=linux
                    node.kubernetes.io/subnetid=fd43acad-33e7-48b2-a85a-24833f362e0e
                    os.architecture=amd64
                    os.name=EulerOS_2.0_SP5
                    os.version=3.10.0-862.14.1.5.h328.eulerosv2r7.x86_64
```
这些标签都是在创建节点的时候CCE自动加上的，下面介绍一个在调度中会用到比较多的标签。
- failure-domain.beta.kubernetes.io/region:表示节点所在的区域，如果上面这个节点标签值为cn-east-3，表示节点在上海一区域
- failure-domain.beta.kubernetes.io/zone:表示节点所在的可用区(availability zone)
- kubernetes.io/hostname: 节点的hostname
此外还可以添加自定义标签，通常情况下，对于一个大型kubernetes集群，肯定会根据业务需要定义很多标签。
在DaemonSet中介绍了nodeSelector，通过nodeSelector可以让Pod只部署在具有特定标签的节点上。如下所示，Pod智慧部署在拥有gpu=true的这个标签的节点上：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector: # 节点选择，当节点拥有gpu-true时才在节点上创建Pod
    gpu: true
```
通过节点亲和性规则配置，也可以做到同样的事情，如下所示：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu
  labels:
    app: gpu
spec:
  selector:
    matchLabels:
      app: gpu
  replicas: 3
  template:
    metadata:
      labels:
        app: gpu
    sepc:
      containers:
      - image: nginx:alpine
        name: gpu
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoreDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
```
这种方式看起来复杂很多，但这种方式可以得到更强的表达能力，后面会进一步介绍。
这里affinity表示亲和，nodeAffinity表示节点亲和,requiredDuringSchedulingIgnoredDuringExection非常长，不过可以将这个分作两端来看：
- 前半段requiredDuringScheduling表示下面定义的规则必须强制满足(require)
- 后半段IgnoredDuringExecution表示不会影响已经在节点上运行的Pod，目前kubernetes提供的规则都是以IgnoredDuringExection结尾的，因为当前的节点亲缘性规则只会影响正在被调度的Pod，最终，kubernetes也会支持RequiredDuringExecution，即去除节点上的某个标签，哪些需要节点包含该标签的Pod将会被剔除。

另外早作福operator的值为In，表示标签值需要在values的列表中，其他的operator取值如下：
- NotIn:标签的值不在某个列表中
- Exists:某个标签存在
- DoesNotExist:某个标签不存在
- Gt:标签的值大于某个值(字符串比较)
- Lt:标签的值小于某个值(字符串比较)
需要说明的是并没有nodeAntiAffinity(节点反亲和),因为NotIn和DoesNotExist可以提供相同的功能。
下面来验证这段规则是否生效，首先给192.168.0.212这个节点打上gpu=true的标签：
```bash
$ kubectl label node 192.168.0.212 gpu=true
node/192.168.0.212 labeled

$ kubectl get node -L gpu
NAME            STATUS   ROLES    AGE   VERSION                            GPU
192.168.0.212   Ready    <none>   13m   v1.15.6-r1-20.3.0.2.B001-15.30.2   true
192.168.0.94    Ready    <none>   13m   v1.15.6-r1-20.3.0.2.B001-15.30.2
192.168.0.97    Ready    <none>   13m   v1.15.6-r1-20.3.0.2.B001-15.30.2
```
创建这个Deployment，可以发现所有的Pod都部署在了192.168.0.212这个节点上了
```bash
$ kubectl create -f affinity.yaml
deployment.apps/gpu created

$ kubectl get pod -owide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE
gpu-6df65c44cf-42xw4     1/1     Running   0          15s   172.16.0.37   192.168.0.212
gpu-6df65c44cf-jzjvs     1/1     Running   0          15s   172.16.0.36   192.168.0.212
gpu-6df65c44cf-zv5cl     1/1     Running   0          15s   172.16.0.38   192.168.0.212
```

### 3.5.2 节点优先选择规则
上面提到的requiredDuringSchedulingIgnoredDuringExecution是一种**强制**选择的规则，节点亲和还有一种优先选择规则，即preferredDuringSchedulingIgoredDuringExecution，表示会根据规则优先选择哪些节点。
为了演示这个效果，先为上面的集群添加一个节点，且这个节点个另外三个节点不在同一个可用区，创建完之后查询节点的可用区标签，如下所示，新添加的节点在cn-east-3c这个可用区。

```bash
$ kubectl get node -L failure-domain.beta.kubernetes.io/zone,gpu
NAME            STATUS   ROLES    AGE     VERSION                            ZONE         GPU
192.168.0.100   Ready    <none>   7h23m   v1.15.6-r1-20.3.0.2.B001-15.30.2   cn-east-3c   
192.168.0.212   Ready    <none>   8h      v1.15.6-r1-20.3.0.2.B001-15.30.2   cn-east-3a   true
192.168.0.94    Ready    <none>   8h      v1.15.6-r1-20.3.0.2.B001-15.30.2   cn-east-3a   
192.168.0.97    Ready    <none>   8h      v1.15.6-r1-20.3.0.2.B001-15.30.2   cn-east-3a```
下面定义个Deployment,要求Pod优先部署在可用区cn-east-3a的节点上，可以像下面这样定义，使用preferredDuringSchedulingIgnoredDuringExection规则，给cn-east-3a设置权重(weight)为80，而gpu=true权重为20，这样pod就优先部署在on-east-3a的节点上。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu
  labels:
    app: gpu
spec:
  selector:
    matchLabels:
      app: gpu
  replicas: 10
  template:
    metadata:
      labels:
        app: gpu
    spec:
      containers:
      - image: nginx:alpine
        name: gpu
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            preference:
              matchExpressions:
              - key: failure-domain.beta.kubernetes.io/zone
                opetator: In
                values:
                - cn-east-3a
          - weight: 20 
            preference:
              matchExpressions:
              - key: gpu
                opetator: In
                values:
                - "true"
```
来看一下实际部署后的情况，可以看到部署到192.168.0.212这个节点上的pod有5个，而192.168.0.100上只有2个。
```bash
$ kubectl create -f affinity2.yaml 
deployment.apps/gpu created

$ kubectl get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP            NODE         
gpu-585455d466-5bmcz   1/1     Running   0          2m29s   172.16.0.44   192.168.0.212
gpu-585455d466-cg2l6   1/1     Running   0          2m29s   172.16.0.63   192.168.0.97 
gpu-585455d466-f2bt2   1/1     Running   0          2m29s   172.16.0.79   192.168.0.100
gpu-585455d466-hdb5n   1/1     Running   0          2m29s   172.16.0.42   192.168.0.212
gpu-585455d466-hkgvz   1/1     Running   0          2m29s   172.16.0.43   192.168.0.212
gpu-585455d466-mngvn   1/1     Running   0          2m29s   172.16.0.48   192.168.0.97 
gpu-585455d466-s26qs   1/1     Running   0          2m29s   172.16.0.62   192.168.0.97 
gpu-585455d466-sxtzm   1/1     Running   0          2m29s   172.16.0.45   192.168.0.212
gpu-585455d466-t56cm   1/1     Running   0          2m29s   172.16.0.64   192.168.0.100
gpu-585455d466-t5w5x   1/1     Running   0          2m29s   172.16.0.41   192.168.0.212
```
上面这个例子中，对节点排序优先级如下所示，有个两个标签的节点排序最高，只有cn-east-3a标签的节点排序第二(权重为80)，只有gpu=true的节点排序第三，没有标签的节点排序最低。优先级排序顺序如下图所示：

![](/assets/img/schedule-order.png)

这里也可以看到pod并没有调度到192.168.0.94这个节点上，这是因为这个节点上部署了很多其他pod，资源使用较多，所以并没有往这个节点上调度，这也侧面说明preferredDuringSchedulingIgnoredDuringExecution是优先规则，而不是强制规则。

### 3.5.1 Pod Affinity(Pod亲和)
节点亲和规则只能影响pod和节点之间的亲和，kubernetes还支持pod和pod直接的亲和，例如将应用的前端和后端部署在一起，从而减少访问延迟。pod亲和同样有requiredDuringSchedulingIgnoredDuringExecution和preferredDuringSchedulingIgnoredDuringExecution两种规则。来看下面的例子，假设有个应用的后端已经创建，且带有app=backend的标签。

```bash
$ kubectl get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE         
backend-658f6cb858-dlrz8   1/1     Running   0          2m36s   172.16.0.67   192.168.0.100
```
将前端frontend的pod部署在backend一起时，可做如下pod亲和规则配置：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: nginx:alpine
        name: frontend
        resources:
          limits:
            cpu: 100m
            momery: 200Mi
          requests:
            cpu: 100m
            momery: 200Mi
      imagePullSecrets:
      - name: default-secret
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - topologyKey: kubernetes.io/hostname
            labelSelector:
              matchLabels:
                app: backend
```
创建frontend后查看，可以看到frontend都创建到跟backend一样的节点上了。
```bash
$ kubectl create -f affinity3.yaml 
deployment.apps/frontend created

$ kubectl get po -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE         
backend-658f6cb858-dlrz8    1/1     Running   0          5m38s   172.16.0.67   192.168.0.100
frontend-67ff9b7b97-dsqzn   1/1     Running   0          6s      172.16.0.70   192.168.0.100
frontend-67ff9b7b97-hxm5t   1/1     Running   0          6s      172.16.0.71   192.168.0.100
frontend-67ff9b7b97-z8pdb   1/1     Running   0          6s      172.16.0.72   192.168.0.100
```
这里有个topologyKey字段，意思是先圈定topologyKey指定的范围，然后再选择下面规则定义的内容。这里每个节点上都有kuberntes.io/hostname，所以看不出topologyKey起到的作用。
如果backend有两个pod分别在不同的节点上。
```bash
$ kubectl get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE         
backend-658f6cb858-5bpd6   1/1     Running   0          23m     172.16.0.40   192.168.0.97
backend-658f6cb858-dlrz8   1/1     Running   0          2m36s   172.16.0.67   192.168.0.100
```
给192.168.0.97和192.168.0.94打上一个perfer=true的标签。
```bash
$ kubectl label node 192.168.0.97 perfer=true
node/192.168.0.97 labeled
$ kubectl label node 192.168.0.94 perfer=true
node/192.168.0.94 labeled

$ kubectl get node -L perfer
NAME            STATUS   ROLES    AGE   VERSION                            PERFER
192.168.0.100   Ready    <none>   44m   v1.15.6-r1-20.3.0.2.B001-15.30.2   
192.168.0.212   Ready    <none>   91m   v1.15.6-r1-20.3.0.2.B001-15.30.2   
192.168.0.94    Ready    <none>   91m   v1.15.6-r1-20.3.0.2.B001-15.30.2   true
192.168.0.97    Ready    <none>   91m   v1.15.6-r1-20.3.0.2.B001-15.30.2   true
```
将podAffinity的toppologyKey定义为prefer.
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoreDuringExection:
    - topologyKey: perfer
      labelSelector:
        matchLabels:
          app: backend
```
调度时先圈定用于perfer标签的节点，这里也就是192.168.0.97和192.168.0.94，然后在匹配app=backend标签的pod，从而frontend就会全部部署在192.168.0.97上。
```yaml
$ kubectl create -f affinity3.yaml 
deployment.apps/frontend created

$ kubectl get po -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE         
backend-658f6cb858-5bpd6    1/1     Running   0          26m     172.16.0.40   192.168.0.97
backend-658f6cb858-dlrz8    1/1     Running   0          5m38s   172.16.0.67   192.168.0.100
frontend-67ff9b7b97-dsqzn   1/1     Running   0          6s      172.16.0.70   192.168.0.97
frontend-67ff9b7b97-hxm5t   1/1     Running   0          6s      172.16.0.71   192.168.0.97
frontend-67ff9b7b97-z8pdb   1/1     Running   0          6s      172.16.0.72   192.168.0.97
```
### 3.5.1 Pod AntiAffinity(Pod反亲和)
通过pod亲和将pod部署在一起，有时候需求却恰恰相反，需要将pod分开部署，例如pod之前部署在一起回影响性能的情况。
下面的例子定义了反亲和规则，这个规则表示pod不能调度到拥有app=frontend标签pod的节点上，也就是下面将frontend分别调度到不同的节点上(每个节点只有一个pod)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  relicas: 5
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: nginx:alpine
        name: frontend
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
      affinity:
        podAffinity:
          requiredDuringSchedulingIngnoredDuringExecution:
          - topologyKey: kubernetes.io/hostname
            labelSelector:
              matchLabels:
                app: frontend
```
创建并查看，可以看到每个节点上只有一个frontend的pod，还有一个在pending，因为在部署第5个时4个节点上都有了app=frontend的pod，所以第5个一直是pending。
```bash
$ kubectl create -f affinity4.yaml 
deployment.apps/frontend created

$ kubectl get po -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE         
frontend-6f686d8d87-8dlsc   1/1     Running   0          18s   172.16.0.76   192.168.0.100
frontend-6f686d8d87-d6l8p   0/1     Pending   0          18s   <none>        <none>       
frontend-6f686d8d87-hgcq2   1/1     Running   0          18s   172.16.0.54   192.168.0.97 
frontend-6f686d8d87-q7cfq   1/1     Running   0          18s   172.16.0.47   192.168.0.212
frontend-6f686d8d87-xl8hx   1/1     Running   0          18s   172.16.0.23   192.168.0.94 
```
# 4 配置管理
## 4.1 ConfigMap
ConfigMap是一种存储应用所需配置信息的资源类型，用于保存配置数据的键值对，可以用来保存单个属性，**也可以用来保存配置文件**。通过ConfigMap可以方便的做到配置解耦，使得不同环境具有不同的配置。相比环境变量。Pod中引用的ConfigMap可以做到实时更新，更新ConfigMap的数据后，Pod中引用的ConfigMap会同步刷新。
### 4.1.1 创建ConfigMap
下面的示例创建了一个名为configmap-test的ConfigMap，ConfigMap的配置数据在data字段定义：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-test
data:
  property_1: Hello
  property_2: World
```
### 4.1.2 在环境变量中引用ConfigMap
ConfigMap最为常见的使用方式就是在环境变量和volume中引用。如下示例中，引用了configmap-test的`property_1`,将其作为环境变量`EXAMPLE_PROPERTY_1`的值，这样容器启动后里面的`EXAMPLE_PROPERTY_1`的值就是property_1的值，即"Hello".
```yaml
apiVersion: v1
kind: Pod
metadata；
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    env:
    - name: EXAMPLE_PROPERTY_1
      valueFrom:
        configMapKeyRef:  # 引用configMap
          name: configmap-test  # configmap的名称
          key: property_1
  imagePullSecrets:
  - name: default-secret
```
### 4.1.3 在Volume中引用ConfigMap
在Volume引用ConfigMap，就是通过文件的方式直接将ConfigMap的每条数据填入Volume，每条数据是一个文件，键就是文件名，键值就是文件内容。
如下示例中，创建一个名为vol-configmap的Volume，这个Volume引用名为"configmap-test"的ConfigMap，再将Volume挂载到容器的"/tmp"路径下。Pod创建成功后，在容器的"/tmp"路径下，就有两个文件`property_1`和`property_2`,它们的值分别为"Hello"和"World".
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    volumeMounts:
    - name: vol-configmap   # 挂载名为vol-configmap的Volume
      mountPath: "/tmp"
  imagePullSecrets:
  volumes:
  - name: vol-configmap
    configMap:             # 引用ConfigMap
      name: configmap-test
```
## 4.2 Secret
Secret是一种加密存储的资源对象，您可以将认证信息，证书，私钥等保存在Secret中，而不需要把这些敏感数据暴露到镜像或Pod定义中，从而更加安全和灵活。
Secret与ConfigMap非常像，都是key-value键值对形式，使用方式也相同，不同的是Secret会加密存储，所以适用于存储敏感信息。
### 4.2.1 Base64编码
Secret与ConfigMap相同，是以键值对形式保存数据，所不同的是在创建是，Secret的Value必须使用Base64编码。
对字符创进行Base64编码，可直接使用"echo -n 要编码的内容|base64"即可，示例如下：
```bash
root@ubuntu:~# echo -n "3306" | base64
MzMwNg==
```
### 4.2.2 创建Secret
如下示例中定义的Secret中包含两天Key-Value.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  key1: aGVsbG8gd29ybGQ=   # "hello world" Base64编码后的值
  key2: MzMwNg==           # "3306" Base64编码后的值
```
### 4.2.3 在环境变量中引用Secret
Secret最常见的用法是作为环境变量注入到容器中，示例如下：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    env:
  imagePullSecrets:
  - name: default-secret
```
### 4.2.4 在Volume中引用Secret
在Volume中引用Secret，就是通过文件的方式，直接将Secret的每条数据填入Volume，每条数据是一个文件，键就是文件名称，键值就是文件内容。
如下示例中，创建一个名为vol-secret的Volume，这个Volume引用名为"mysecret"的Secret，再将Volume挂载到容器的"/tmp"路径下。Pod创建成功后，在容器的"/tmp"路径下，就有两个key1和key2
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec；
  containers:
  - image: nginx:alpine
    name: container-0
    resources:
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
    volumeMounts:
    - name: vol-secret   # 挂载名为vol-secret的Volume
      mountPath: "/tmp"
  imagePullSecrets:
  volumes:
  - name: vol-secret
    secret:              # 引用secret
      secretName: mysecret
```
进入Pod容器中，可以在/tmp目录下发现key1和key2两个文件，并看到文件中的值是base64解码后的值，分贝为"Hello world"和"3306".

# 5 kubernetes网络
## 5.1 容器网络
kubernetes本身并不负责网络通信，kubernetes提供了容器网络接口CNI(container network interface)，具体的网络通信交给CNI插件来负责，开源的CNI插件非常多，像Flannel,Calico等，华为云CCE也专门为kubernetes定制了CNI插件，使得kubernetes可以使用华为云VPC网络。
kubernetes虽然不负责网络，但要求集群中的Pod能相互通信，且Pod必须通过非NAT网络连接，即收到数据包的源IP就是发送数据包Pod的IP。同时Pod与节点之前的通信也是通过非NAT网络。但是Pod访问集群外部时源IP会被修改成节点的IP。
Pod内部是通过虚拟Ethernet接口对(Veth pair)与Pod外部连接，Veth pair就像一个网线，一端留在Pod内部，一端在Pod之外。而同一个节点上的Pod通过网桥(Linux Bridge)通信，如下图所示：

![](/assets/img/pod-on-one-node.png)

不同节点间的网桥连接有很多种方式，这跟具体实现相关。但集群要求Pod的地址唯一，所以跨几点的网桥通常使用不同的地址段，以防止Pod的IP地址重复。不同节点上的Pod通信如下图所示：

![](/assets/img/pod-on-diff-node.png)

以上就是容器网络底层视图，后面将进一步介绍kubernetes是如何在此基础上向用户提供访问方案的。具体参见service和Ingress。
## 5.2 Service
### 5.2.1 直接访问Pod的问题
Pod创建完成后，如何访问Pod呢？直接访问Pod会有如下几个问题：
- Pod随时会被Deployment这样的控制器删除重建，那访问Pod的结果就会变得不可预知
- Pod的IP地址是Pod启动后才被分配，在启动前并不直到Pod的IP地址
- 应用往往都是由多个运行相同镜像的一组Pod组成，逐个访问Pod也变得不现实
举个例子，假设有这样一个应用程序，使用Deployment创建了前台和后台，前台会调用后台做一些计算处理，如下图所示。后台运行了3个Pod。这些Pod是相互独立且可被替换的，当Pod出现状况被重建时，新建的Pod的IP地址是新IP，前台的Pod无法直接感知。

![](/assets/img/service-pod.png)

### 5.2.2 使用service解决Pod访问的问题
kubernetes中的service对象就死用来解决上述Pod访问问题的。service有一个固定IP地址(在创建CCE集群是有一个服务网段的设置，这个网段专门用于给Service分配IP地址)，Service将访问它的流量转发给Pod，具体转发给哪些Pod通过Label来选择，而且Service可以给这些Pod做负载均衡。
那么对于上面的例子，为后台添加一个Service，通过Service来访问Pod，这样前台Pod就无需感知后台Pod的变化，如下图所示：

![](/assets/img/svc-backend.png)

### 5.2.3 创建后台Pod
首先创建一个3副本的Deployment，即3个Pod，且Pod上带有标签"app:nginx",具体如下：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:latest
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
```
### 5.2.4 创建service
下面示例创建一个名为"nginx"的Service，通过selector选择到标签"app:nginx"的Pod，目标Pod的端口为80，Service对外暴露的端口为8080.
访问服务只需要通过"**服务名称:对外暴露的端口**"接口，对应本例即"nginx:8080"。这样，在其他Pod中，只需要通过"nginx:8080"就可以访问到"nginx"关联的Pod。
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:    # Label Selector,选择包含app=nginx标签的Pod
    app: nginx
  ports:
    - name: service0
      targetPort: 80 # Pod的端口
      port: 8080     # Service对外暴露的端口
      protocol: TCP  # 转发协议类型，支持TCP和UDP
  type: ClusterIP    # Service的类型
```
将上面service的定义保存到nginx-svc.yaml文件中，使用kubectl创建这个Service。
```bash
$ kubectl create -f nginx-svc.yaml
service/nginx created

$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.247.0.1       <none>        443/TCP    7h19m
nginx        ClusterIP   10.247.124.252   <none>        8080/TCP   5h48m
```
可以看到Service有个ClusterIP，这个IP是固定不变的，除非将service删除，所以你可以使用ClusterIP在集群内部访问Service。下面创建一个Pod并进入容器，使用ClusterIP访问Pod，可以看到能直接返回内容。
```bash
$ kubectl run -i --tty --image nginx:alpine test --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # curl 10.247.124.252:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
### 5.2.5 使用ServiceName访问service
通过DNS进行域名解析后，可以使用"**ServiceName:Port**"访问Service，这也是kubernetes最常见的一种使用方式，在创建CCE集群的时候，会默认要求安装CoreDNS插件，在kube-system命名空间下可以查看到CoreDNS的Pod。
```bash
$ kubectl get po --namespace=kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
coredns-7689f8bdf-295rk                   1/1     Running   0          9m11s
coredns-7689f8bdf-h7n68                   1/1     Running   0          11m
```
CoreDNS安装成功后，会称为DNS服务器，当创建Service后，CoreDNS会将Service的名称与IP记录起来，这样Pod就可以通过想CoreDNS查询Service的名称获得Service的IP地址。
访问是通过nginx.<namespace>.svc.cluster.local访问，其中nginx为Service的名称，<namespace>为命名空间名称，svc.cluster.local为域名后缀，在实际使用中，在同一个命名空间下可以省略<namespace>.svc.cluster.local,直接使用ServiceName即可。
例如上面创建的名为nginx的service，可以直接通过"**nginx:8080**"就可以访问到Service，进而访问后台Pod。
使用ServiceName的方式有个主要的优点就是可以在开发应用程序时可以将ServiceName写在程序中，这样无需感知具体Service的IP地址。
下面创建一个Pod并进入容器，查询nginx域名的地址，可以发现是解析出nginx这个Service的IP地址10.247.124.252；同时访问Pod的域名，可以看到能直接返回内容。
```bash
$ kubectl run -i --tty --image tutum/dnsutils dnsutils --restart=Never --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx
Server:		10.247.3.10
Address:	10.247.3.10#53

Name:	nginx.default.svc.cluster.local
Address: 10.247.124.252

/ # curl nginx:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
### 5.2.6 Service是如何做到服务发现的
前面提到，有了Service后，无论Pod如何变化，Service都能发现到Pod。如果是用kubectl describe 命令查看Service的信息，将看到如下信息：
```bash
$ kubectl describe svc nginx
Name:              nginx
......
Endpoints:         172.16.2.132:80,172.16.3.6:80,172.16.3.7:80
......
```
可以看到一个Endpoints,Endpoints同样也是kubernetes的一种资源对象，可以查询得到。kubernetes正是通过Endpoints监控到Pod的IP，从而让Service能发现Pod。
```bash
$ kubectl get endpoints
NAME         ENDPOINTS                                     AGE
kubernetes   192.168.0.127:5444                            7h19m
nginx        172.16.2.132:80,172.16.3.6:80,172.16.3.7:80   5h48m
```
这里的172.16.2.132:80是Pod的IP:Port，通过如下命令可以查看到Pod的IP，与上面的IP一致。
```bash
$ kubectl get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE
nginx-869759589d-dnknn   1/1     Running   0          5h40m   172.16.3.7     192.168.0.212
nginx-869759589d-fcxhh   1/1     Running   0          5h40m   172.16.3.6     192.168.0.212
nginx-869759589d-r69kh   1/1     Running   0          5h40m   172.16.2.132   192.168.0.94
```

![](/assets/img/svc-endpoints.png)

如果删除一个Pod，Deployment会将Pod重建，新的Pod IP会发生变化。
```bash
$ kubectl delete po nginx-869759589d-dnknn
pod "nginx-869759589d-dnknn" deleted

$ kubectl get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE
nginx-869759589d-fcxhh   1/1     Running   0          5h41m   172.16.3.6     192.168.0.212
nginx-869759589d-r69kh   1/1     Running   0          5h41m   172.16.2.132   192.168.0.94
nginx-869759589d-w98wg   1/1     Running   0          7s      172.16.3.10    192.168.0.212
```
再次查看Endpoints，会发现Endpoints的内容随着Pod发生了变化。
```bash
$ kubectl get endpoints
NAME         ENDPOINTS                                      AGE
kubernetes   192.168.0.127:5444                             7h20m
nginx        172.16.2.132:80,172.16.3.10:80,172.16.3.6:80   5h49m
```
下面进一步了解这又是如何实现的。
在kubernetes集群架构中提到过Node节点上的kube-proxy，实际上Service相关的事情都有节点上的kube-proxy处理。在Service创建是kubernetes会分配IP给Service，同时通过API server通知所有的kube-proxy有新Service创建了，kube-proxy收到通知后通过iptables记录Service和IP/端口对的关系，从而让Service在节点上可以被查询到。
下面是一个实际访问Service的图示，Pod X访问Service(10.247.124.252:8080)，在往外发数据包时，再节点上根据iptables规则，目的IP:Port被随机替换为Pod1的IP:Port，从而通过Service访问到实际的Pod。
除了记录Service和IP/渡口对的关系，kube-proxy还会监控Service和Endpoints的变化，从而保证Pod重建后仍然能通过Service访问到Pod。Pod X访问Service的过程如下图所示：

![](/assets/img/pod-svc-pod.png)

### 5.2.7 Service的类型与使用场景
Service的类型除了ClusterIP还有NodePort、LoadBalancer和None，这几种类型的Service有着不同的用途。
- ClusterIP:用于在集群内部相互访问的场景，通过ClusterIP访问Service
- NodePort:用于从集群外部访问的场景，通过节点上的端口访问Service
- LoadBalancer:用于从集群外部访问的场景，其实是NodePort的扩展，通过一个特定的LoadBalancer访问Service，这个LoadBalancer将请求转发到节点的NodePort,而外部只需要访问LoadBalancer
- None:用于Pod键的相互发现，这种类型的Service又叫Headless Service

### 5.2.8 NodePort类型的Service
NodePort类型的Service可以让kubernetes集群的每个Node节点上保留一个相同的端口，外部访问连接首先访问Node节点的IP:Port，然后将这些连接转发给服务对应的Pod，如下图所示：

![](/assets/img/np-svc.png)

下面是一个创建NodePort类型的Service。创建完成后，可以通过Node节点的IP:Port访问到后台的Pod。
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: nodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30120
  selector:
    app: nginx
```
创建并查看，可以看到PORT这一列为8080:30120/TCP，说明Service的8080端口是映射到节点的30120端口。
```bash
$ kubectl create -f nodeport.yaml
service/nodeport-service created

$ kubectl get svc -owide
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes         ClusterIP   10.247.0.1       <none>        443/TCP          107m   <none>
nginx              ClusterIP   10.247.124.252   <none>        8080/TCP         16m    app=nginx
nodeport-service   NodePort    10.247.210.174   <none>        8080:30120/TCP   17s    app=nginx
```
此时，通过节点IP:端口访问Service可以访问到Pod，如下所示。
```bash
$ kubectl run -i --tty --image nginx:alpine test --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # curl 192.168.0.212:30120
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
......
```
### 5.2.9 LoadBalancer类型的Service
LoadBalancer类型的Service其实是NodePort类型Service的扩展，通过一个特定的LoadBalancer访问Service，这个LoadBalancer将请求转发到节点的NodePort。
LoadBalancer本身不是kubernetes的组件，通常是有具体厂商(云服务提供商)提供，不同厂商的kubernetes集群与LoadBalancer的对接实现各不相同，例如华为云CCE对接了ELB。这就导致了创建LoadBalancer类型的Service有不同的实现。示意图如下所示：

![](/assets/img/lb-svc.png)

下面是一个创建LoadBalancer类型的Service。创建完成后，可以通过**ELB的IP:Port**访问到后台Pod。

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubernetes.io/elb.id: 3c7caa5a-a641-4bff-801a-feace27424b6
  name: nginx
  labels:
    app: nginx
spec:
  loadBalancerIP: 10.78.42.242  # ELB示例的IP地址
  ports:
  - port: 80
    name: service0
    protocol: TCP
    targetPort: 80
    nodePort: 30120
  type: LoadBalancer
  selector:
    app: nginx
```
上面metadata.annotations里的参数配置是CCE的LoadBalancer类型Service需要配置的参数，表示这个Service绑定哪个ELB实例。CCE还支持创建LoadBalancer类型Service时新建ELB实例
### 5.2.10 Headless Service
前面将的Service解决了Pod的内外部访问问题，但还有下面这些问题没解决。
- 同时访问所有Pod
- 一个Service内部Pod相互访问

Headless Service正是解决这个问题的，headless Service不会创建ClusterIP，并且查询会返回所有Pod的DNS记录，这样就可以查询到所有Pod的IP地址。StatefulSet正是使用headless service解决Pod间相互访问的问题。
```yaml
apiVersion: v1
kind: Service  # 对象类型为Service
metadata:
  name: nginx-headless
  labels:
    app: nginx
spec:
  selector:
    app: nginx    # 选择app:nginx的Pod
  ports:
  - name: nginx   # Pod间通信的端口名称
    port: 80      # Pod间通信的端口号
  ClusterIP: None  # 必须设置为None，表示headless service
```
创建headless service后，可以查询service
```bash
# kubectl create -f headless.yaml
service/nginx-headless created
# kubectl get svc
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx-headless   ClusterIP   None         <none>        80/TCP    5s
```
创建一个Pod来查询DNS，可以看到能返回所有Pod的记录，这就解决了访问所有Pod的问题了。
```bash
$ kubectl run -i --tty --image tutum/dnsutils dnsutils --restart=Never --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx-0.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-0.nginx.default.svc.cluster.local
Address: 172.16.0.31

/ # nslookup nginx-1.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-1.nginx.default.svc.cluster.local
Address: 172.16.0.18

/ # nslookup nginx-2.nginx
Server:         10.247.3.10
Address:        10.247.3.10#53
Name:   nginx-2.nginx.default.svc.cluster.local
Address: 172.16.0.19
```
## 5.3 Ingress
### 5.3.1 为什么需要Ingress
Service是基于四层TCP和UDP协议转发的，而Ingress可以基于七层的HTTP和HTTPS协议转发，可以通过域名和路径做到更细粒度的划分，如下图所示：

![](/assets/img/ing.png)

### 5.3.2 Ingress工作机制
要想使用Ingress功能，必须在kubernetes集群上安装Ingress controller。Ingress controller有多种实现，最常见的就是kubernetes官方维护的NGINX Ingress Controller；不同厂商通常有自己的实现，如华为云CCE使用华为云弹性负载均衡ELB实现Ingress的七层负载均衡。
外部请求首先到达Ingress Controller，Ingress Controller根据Ingress的路由规则，查找到对应的Service，进而通过Endpoint查询到Pod的IP地址，然后将请求转发给Pod。Ingress工作机制如下图所示：

![](/assets/img/ing-to-pod.png)

### 5.3.3 创建Ingress
下面的示例中，使用http协议，关联后端Service为"nginx:8080"，使用ELB作为Ingress controller(metadata annotations字段指定使用那个ELB实例)，当访问http://192.168.10.155:8080/test时，流量转发nginx:8080对应的Service，从而将流量转发到对应Pod。
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: cce
    kubernetes.io/elb.port: '8080'
    kubernetes.io/elb.ip: 192.168.10.155
    kubernetes.io/elb.id: aa7cf5ec-7218-4c43-98d4-c36c0744667a
spec:
  rules:
  - host: ''
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 8080
        path: "/test"
        property:
          ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH
```
**Ingress中还可以设置外部域名**,这样就可以通过域名来访问到ELB，进而访问到后端服务。
>说明：域名访问依赖于域名解析，需要将域名解析指向ELB实例的IP地址

```bash
spec:
  rules:
  - host: www.example.com    # 域名
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
```
### 5.3.4 路由到多个服务
ngress可以同时路由到多个服务，配置如下所示：
- 当访问http://foo.bar.com/foo时，访问的是s1:80后端
- 当访问http://foo.bar.com/bar时，访问的是s2:80后端
```yaml
spec:
  rules:
  - host: foo.bar.com  # host地址
    http:
      paths:
      - path: "/foo"
        backend:
          serviceName: s1
          servicePort: 80
      - path: "/bar"
        backend:
          serviceName: s2
          servicePort: 80
```
## 5.4 就绪探针(Readliness Probe)
一个新Pod创建后，service就能立即选择到它，并会把请求转发给Pod，那问题来了，通常一个Pod启动是需要时间的，如果Pod还没准备好(可能需要时间来加载配置或数据，或可能需要执行一个预热程序之类)，这是把请求转给Pod的话，Pod也无法处理，造成请求失败。
kubernetes解决这个问题的方法就是给Pod加一个业务就绪探针Readiness Probe，当检查到Pod就绪后才允许service将请求转发给Pod。
Readiness Probe同样四周期性的检测Pod，然后根据响应来判断Pod是否就绪，与存活探针相同，就绪探针也支持如下三种类型：
- Exec：Probe执行容器中的命令并检查命令退出的状态码，如果状态码为0则说明已经就绪
- HTTP GET：往容器的IP:Port发送HTTP GET请求，如果Probe收到2xx或3xx，说明已经就绪
- TCP Socket：尝试与容器建立TCP连接，如果能建立连接说明已经就绪
### 5.4.1 Readiness Probe的工作原理
通过Endpoints就可以实现Readiness Probe的效果，当Pod还未就绪时，将Pod的IP:Port从Endpoints中删除，Pod就绪后再加入到Endpoints中，Readiness Probe的实现原理如下图所示：

![](/assets/img/readiness.png)
### 5.4.2 Exec
Exec方式与HTTP GET方式一致，如下所示，这个探针执行ls /ready命令，如果这个文件存在，则返回0，说明Pod就绪了，否则返回其他状态码。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        readinessProbe:      # Readiness Probe
          exec:              # 定义 ls /ready 命令
            command:
            - ls
            - /ready
      imagePullSecrets:
      - name: default-secret
```
将上面Deployment的定义保存到deploy-read.yaml文件中，删除之前创建的Deployment，用deploy-read.yaml创建这个Deployment。
```bash
# kubectl delete deploy nginx
deployment.apps "nginx" deleted

# kubectl create -f deploy-read.yaml
deployment.apps/nginx created
```
这里由于nginx镜像不包含/ready这个文件，所以在创建完成后容器不在Ready状态，如下所示，注意READY这一列的值为0/1，表示容器没有Ready。
```bash
# kubectl get po
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7955fd7786-686hp   0/1       Running   0          7s
nginx-7955fd7786-9tgwq   0/1       Running   0          7s
nginx-7955fd7786-bqsbj   0/1       Running   0          7s
```
创建Service。
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: service0
    targetPort: 80
    port: 8080
    protocol: TCP
  type: ClusterIP
```
查看Service，发现Endpoints一行的值为空，表示没有Endpoints。
```bash
$ kubectl describe svc nginx
Name:              nginx
......
Endpoints:
......
```
如果此时给容器中创建一个/ready的文件，让Readiness Probe成功，则容器会处于Ready状态。再查看Pod和Endpoints，发现创建了/ready文件的容器已经Ready，Endpoints也已经添加。
```bash
# kubectl exec nginx-7955fd7786-686hp -- touch /ready

# kubectl get po -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP
nginx-7955fd7786-686hp   1/1       Running   0          10m       192.168.93.169
nginx-7955fd7786-9tgwq   0/1       Running   0          10m       192.168.166.130
nginx-7955fd7786-bqsbj   0/1       Running   0          10m       192.168.252.160

# kubectl get endpoints
NAME       ENDPOINTS           AGE
nginx      192.168.93.169:80   14d
```
### 5.4.3 HTTP GET
### 5.4.4 TCP Socket
### 5.4.5 Readiness Probe高级配置
与Liveness Probe相同，Readiness Probe也有同样的高级配置选项，上面nginx Pod的describe命令回显有中有如下行。
```bash
Readiness: exec [ls /var/ready] delay=0s timeout=1s period=10s #success=1 #failure=3
```
这一行表示Readiness Probe的具体参数配置，其含义如下：

- delay=0s 表示容器启动后立即开始探测，没有延迟时间
- timeout=1s 表示容器必须在1s内做出相应反馈给probe，否则视为探测失败
- period=10s 表示每10s探测一次
- #success=1 表示探测连续1次成功表示成功
- #failure=3 表示探测连续3次失败后会重启容器

这些是创建时默认设置的，您也可以手动配置，如下所示。
```yaml
        readinessProbe:      # Readiness Probe
          exec:              # 定义 ls /readiness/ready 命令
            command:
            - ls
            - /readiness/ready
          initialDelaySeconds: 10    # 容器启动后多久开始探测
          timeoutSeconds: 2          # 表示容器必须在2s内做出相应反馈给probe，否则视为探测失败
          periodSeconds: 30          # 探测周期，每30s探测一次
          successThreshold: 1        # 连续探测1次成功表示成功
          failureThreshold: 3        # 连续探测3次失败表示失败
```
## 5.5 NetworkPolicy
NetworkPolicy是kubernetes设计用来限制Pod访问的对象，通过设置NetworkPolicy策略，可以允许Pod被哪些地址访问(入规则)、或Pod访问哪些地址(出规则)。这相当于从应用的层面构建了一道防火墙，进一步保证了网络安全。
NetworkPolicy支持的能力取决于集群的网络插件的能力，如CCE的集群只支持设置Pod的入规则。
默认情况下，如果命名空间中不存在任何策略，则所有进出该命名空间中的Pod流量都被允许。NetworkPolicy规则可以选择如下3种：
- namespaceSelector:根据命名空间的标签选择，具有该标签的命名空间都可以访问
- podSelector:根据Pod的标签选择，具有该标签的Pod都可以访问
- ipBlock:根据网络选择，网段内的IP地址都可以访问(CCE当前不支持此种方式)

### 5.5.1 使用podSelector设置访问范围
使用podSelector设置访问范围的yaml定义如下：
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:  # 表示入规则
  - from:
    - podSelector:  # 只允许具有role=frontend标签的Pod访问
        matchLabels:
          role: fontend
    ports:   # 只能使用TCP协议访问6379端口
    - protocol: TCP
      port: 6379
```
示意图如下所示：

![](/assets/img/nwpl-pod.png)

### 5.5.2 使用namespaceSelector
使用namespaceSelector设置访问范围的YAML定义如下：
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:   # 表示入规则
  - from:
    - namespaceSelector: # 只允许具有project=myproject标签的命名空间中的Pod访问
        matchLabels:
          project: myproject
    ports:  # 只能使用TCP协议访问6379端口
    - protocol: TCP
      port: 6379
```
示意图如下所示：
![](/assets/img/nwpl-ns.png)

# 6 持久化存储
## 6.1 Volume
容器中的文件在磁盘上是临时存放的，当容器重建时，容器中的文件将会丢失，另外当在Pod中同时运行多个容器时，常常需要在这些容器之间共享文件，这也是容器不好解决的问题。kubernetes抽象出了Volume来解决这个两个问题，也就是存储卷，**kubernetes的Volume是Pod的一部分**,Volume不是单独的对象，不能独立创建，只能在Pod中定义。
Pod中的所有容器都可以访问Volume，但必须要挂载，且可以挂载到容器中任何目录。
实际中使用容器存储如下图所示，将容器的内容挂载到Volume中，通过Volume，两个容器键实现了存储共享。
![](/assets/img/volume.png)
Volume的生命周期与挂载它的Pod相同，但是Volume里面的文件可能在Volume消失后仍然存在，这取决于Volume的类型。
### 6.1.1 Volume的类型
kubernetes的Volume有非常多的类型。但实际中使用最多的类型如下：
- emptyDir:一种简单的空目录，主要用于临时存储
- hostPath:将主机某个目录挂载到容器中
- ConfigMap/Secret:特殊类型，将kubernetes特定的对象类型挂载到Pod
- persistentVolumeClaim:kubernetes的持久化存储类型
### 6.1.2 EmptyDir
emptyDir是最简单的一种Volume类型，根据名字就能看出，这个Volume挂载后就是一个空目录，应用程序可以在里面读写文件，emptyDir Volume的生命周期与Pod相同，Pod删除后Volume的数据也同时删掉。
emptyDir的一些用途如下：
- 缓存空间，例如基于磁盘的归并排序
- 为耗时较长的计算任务提供检查点，以便任务能从崩溃前的状态恢复执行
emptyDir配置示例如下：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```
emptyDir实际是将Volume的内容写在Pod所在节点的磁盘上。另外emptyDir也可以设置存储介质为内存，如下所示，medium设置为Memory:
```yaml
volumes:
- name: html
  emptyDir:
    medium: Memory
```
### 6.1.3 HostPath
hostPath是一种持久化存储，emptyDir里的内容会随着Pod的删除而消失，但hostPath不会，如果对应的Pod删除，hostPath Volume里的内容依然存在与节点的目录中，如果后续重新创建Pod并调度到同一个节点，挂载后依然可以读取到之前Pod写的内容。
hostPath存储的内容与节点相关，所以它不适合想数据库这类的应用，想象如果数据库的Pod被调度到了别的节点，那读取的内容就完全不一样了。
永远记住不要使用hostPath存储跨Pod的数据，一定要把hostPath的使用范围限制在读取节点文件上，这是因为Pod被重建后不确定调度那个节点上，写文件可能会导致前后不一致。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-hostpath
spec:
  containers:
    - image: nginx:alpine
      name: hostpath-container
      volumeMounts:
        - name: test-volume
          mountPath: /test-pd
  volumes:
    - name: test-volume
      hostPath:
        path: /data
```
## 6.2 PV PVC和StorageClass
前面提到hostPath是一种持久化存储，但hostPath的内容是存储在节点上，导致只适合读取。
### 6.2.2 PV
如果要求Pod重新调度后仍然能使用之前读写过的数据，就只能使用网络存储了，网络存储种类非常多且有不同的使用方法，通过一个云服务提供商至少有块存储、文件存储、对象存储这三种。kubernetes解决这个问题的方式是抽象了PV(PersistentVolume)和PVC(PersistentVolumeClaim)来解耦这个问题，从而让使用者不用关心具体的基础设施，当需要存储资源的时候，，只要想CPU和内存一样，声明要多少即可。
- PV:PV描述的是持久化存储卷，主要定义的是一个持久化存储在宿主机上的目录，比如一个NFS的挂载目录。
- PVC:PVC描述的是Pod所希望使用的持久化存储的属性，比如，Volume存储的大小，可读写权限等等。
kubernetes管理员设置好网络存储的类型，提供对应的PV描述符配置到kubernetes，使用者需要存储的时候只需要创建PVC，然后在Pod中使用Volume关联PVC，即可让Pod使用到存储资源，它们之间的关系如下图所示：
![](/assets/img/pv-pvc.png)
### 6.2.1 CSI
Kubernetes提供了CSI接口（Container Storage Interface，容器存储接口），基于CSI这套接口，可以开发定制出CSI插件，从而支持特定的存储，达到解耦的目的。例如在Namespace：资源分组中看到的kube-system命名空间下everest-csi-controller和everest-csi-driver就是华为云CCE开发存储控制器和驱动。有了这些驱动就可以使用华为云上的EVS、SFS、OBS存储。
```bash
$ kubectl get po --namespace=kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
everest-csi-controller-6d796fb9c5-v22df   2/2     Running   0          9m11s
everest-csi-driver-snzrr                  1/1     Running   0          12m
everest-csi-driver-ttj28                  1/1     Running   0          12m
everest-csi-driver-wtrk6                  1/1     Running   0          12m
```
### 6.2.2 PV
PV是如何描述持久化存储的呢？假设在华为云SFS中创建了一个文件存储，这个文件存储ID为68e4a4fd-d759-444b-8265-20dc66c8c502，挂载地址为sfs-nas01.cn-north-4b.myhuaweicloud.com:/share-96314776。如果想在CCE中使用这个文件存储，则需要先创建一个PV来描述这个存储，如下所示：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  accessModes:
    - ReadWriteMany   # 读写模式
    capacity:
      storage: 10Gi   # 定义PV的大小
    csi:
      driver: nas.csi.everest.io         # 声明使用的驱动                   
      fsType: nfs                        # 存储类型
      volumeAttributes:
        everest.io/share-export-location: sfs-nas01.cn-north-4b.myhuaweicloud.com:/share-96314776   # 挂载地址
      volumeHandle: 68e4a4fd-d759-444b-8265-20dc66c8c502                                            # 存储ID
```
这里csi下面的内容就是华为云CCE中特定的字段，在其他地方无法使用。下面创建这个PV并查看。
```bash
$ kubectl create -f pv.yaml
persistentvolume/pv-example created

$ kubectl get pv
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM    STORAGECLASS   REASON   AGE
pv-example           10Gi       RWX            Retain           Available                                    4s
```
RECLAIM POLICY是指PV的回收策略，Retain表示PVC被释放后PV继续保留。STATUS值为Available，表示PV处于可用的状态。
### 6.2.3 PVC
PVC可以绑定一个PV，示例如下：
```yaml
apiVersion: v1
kind: PersistentVoulmeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Gi   # 声明存储的大小
  volumeName: pv-example # PV的名称
```
创建PVC并查看，可以看到状态是Bound,VOLUME是pv-example,表示PVC已经绑定了PV
```bash
$ kubectl create -f pvc.yaml
persistentvolumeclaim/pvc-example created

$ kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-example   Bound    pv-example   10Gi       RWX                           9s
```
再次查看PV，可以看到状态变成了Bound,CLAIM是default/pvc-example,表示这个PV绑定了default namespace下的pvc-example这个PVC。
```bash
$ kubectl get pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pv-example    10Gi       RWX            Retain           Bound    default/pvc-example                            50s
```
示意图如下所示：
![](/assets/img/pvc-bound-pv.png)
### 6.2.4 StorageClass
上面提到的PV和PVC方法虽然能实现屏蔽底层存储，但是PV创建比较复杂(可以看到PV中csi字段配置很麻烦),通常是由集群管理员管理，这非常不方便。
kubernetes解决这个问题的办法是提供动态配置PV的方法，可以自动创建PV。管理员可以部署PV配置器(provisioner),然后定义对应的StorageClass，这样开发者在创建PVC的时候就可以选择需要创建存储的类型，PVC会把StorageClass传递给PV provisioner,由provisioner自动创建PV。如CCE就提供csi-disk,csi-nas,csi-obs等StorageClass，在声明PVC加上StorageClassName，就可以自动创建PV，并自动创建底层的存储资源。
>说明：下面是以CCE 1.15及以上版本集群使用方法举例

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata；
  name: pvc-sfs-auto-example
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: csi-nas  # StorageClass
```
创建PVC并查看PVC和PV
```bash
$ kubectl create -f pvc2.yaml
persistentvolumeclaim/pvc-sfs-auto-example created

$ kubectl get pvc
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-sfs-auto-example   Bound    pvc-1f1c1812-f85f-41a6-a3b4-785d21063ff3   10Gi       RWX            csi-nas        29s

$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
pvc-1f1c1812-f85f-41a6-a3b4-785d21063ff3   10Gi       RWO            Delete           Bound    default/pvc-sfs-auto-example  csi-nas                 20s
```
可以看到**使用StorageClass后,不仅创建了PVC，而且创建PV，并且将二者绑定了**
定义了StorageClass后，就可以减少创建并维护PV的工作，PV变成了自动创建，作为使用者，只需要在声明PVC时指定StorageClassName即可，这就大大减少了工作量。
**再次说明，StorageClassName的类型在不同厂商的产品上各不相同，这里只是使用华为云文件存储作为示例**.

### 6.2.5 在Pod中使用PVC
有了PVC后，在Pod中使用持久化存储就非常方便了，在Pod Template中的Volume直接关联PVC的名称，然后挂载到容器中即可，如下所示。甚至在StatefulSet中可以直接声明PVC。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:alpine
        name: container-0
        volumeMounts:
        - mountPath: /tmp
          name: pvc-sfs-example
      restartPolicy: Always
      volumes:
      - name: pvc-sfs-example
        persistentVolumeClaim:
          claimName: pvc-example
```

# 7 认证与授权
## 7.1 ServiceAccount
### 7.1.1 认证与ServiceAccount
### 7.1.2 创建ServiceAccount
### 7.1.3 在Pod中使用ServiceAccount
## 7.2 RBAC
### 7.2.1 RBAC资源
### 7.2.2 创建Role
### 7.2.3 创建RoleBinding
### 7.2.4 ClusterRole和ClusterRoleBinding
# 8 弹性伸缩





