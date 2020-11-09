---
layout: post
title: Kubernetes from scratch:certificates[译]
date: 2020-11-08
author: tuxnotes
tags: kubernetes, certificate
---

原文链接：https://medium.com/@oleg.pershin/kubernetes-from-scratch-certificates-53a1a16b5f03

理想情况，下kubernetes的所有组件之间的相互交互都要通过TLS。并且没有证书就没有K8S认证和RBAC。这其中有大量的客户端，服务端，parent(CA)和child证书需要提前创建好，在部署kubernetes集群之前。首先需要明确的是我们需要什么样的证书，以及如何生成这些证书。

### Authentication认证

首先看一下在kubernetes集群中基于证书的认证是如何工作的，如下图所示：

![](/assets/img/k8s-auth.jpeg)

上图展示了apiserver为什么使用一个特殊的client certificate对kubectl进行认证；kubectl为什么信任apiserver的certificate。因为apiserver和kubectl都信任这个相同的intermediate kubernetes CA,此CA用来签署client和server的证书。

#### Prerequisites

1. openssl
2. docker and docker-compose
3. kubectl

假定我们部署kubernetes的公司已经有了root CA证书，但是对我们不公开。但公司生成并签署了一个intermediate证书，专门用于kubernetes集群部署。这一步是可选的，没有公司的root CA我们也可以完成部署，但我们的就是我们的kubernetes的CA证书不是签过名的。

出于演示的目的，我们可以自己生成一个假的公司root CA证书，命令如下：

```bash
mkdir certs
openssl req \
 -nodes \
 -subj "/C=US/ST=None/L=None/O=None/CN=example.com" \
 -new -x509 \
 -days 9999 \
 -keyout certs/company-root-ca.key \
 -out certs/company-root-ca.crt
```

接下来生成中间证书kubernetes-ca。首先我们需要一个私钥，命令如下：

```bash
openssl genrsa -out certs/kubernetes-ca.key 4096
```

现在我们需要创建一个证书签名请求Certificate Signing Request(CSR)，并将其传给root CA持有者(公司)来对CSR进行签名。公司的管理员将会使用如下的命令进行操作来创建证书签名请求：

```bash
openssl req \
  -new \
  -nodes \
  -sha256 \
  -subj "/C=US/ST=None/L=None/O=None/CN=kubernetes-ca" \
  -key certs/kubernetes-ca.key \
  -out certs/kubernetes-ca.csr
```

>注:证书签名请求本质上是带有公私信息和组织信息的综合体，公钥是由私钥根据算法生成的，但不能由公钥反推出私钥。

公司收到你的CSR后就可以对其签名了，命令如下：

```bash
openssl x509 \
  -req \
  -days 9999 \
  -sha256 \
  -CA certs/company-root-ca.crt \
  -CAkey certs/company-root-ca.key \
  -set_serial 01 \
  -extensions req_ext \
  -in certs/kubernetes-ca.csr \
  -out certs/kubernetes-ca.crt
```

作为CSR的产出无，公司会返回给你证书，即`kubernetes-ca.crt`文件。

**如果公私不能对你的中间证书进行签名，那就只创建不签名**,命令如下：

```bash
openssl req \
 -nodes \
 -subj "/C=US/ST=None/L=None/O=None/CN=example.com" \
 -new -x509 \
 -days 9999 \
 -keyout certs/kubernetes-ca.key \
 -out certs/kubernetes-ca.crt
```

由此可见，如果有上级证书可用于签名的话，就可使用CSR的方式生成证书。如果没有上级证书可用，则可以由自己生成的私钥直接生成没有上级证书签名的证书。

到目前你拥有了一个intermedia kubernetes CA:由私钥kubernetes-ca.key和证书kubernetes-ca.crt构成。你可以将kuberntes-ca.crt分享给任何你想分享的人，但kubernetes-ca.key必须保密。

到此我们已经完成了可以签名kube-apiserver的服务端证书的准备工作，接下来首先需要生成apiserver的私钥，命令如下：

```bash
openssl genrsa -out certs/kube-apiserver.key 4096
```
因为我们已经有了中间证书kubernetes-ca.crt用于签名，所以接下来创建apiserver的签名请求CSR:

```bash
openssl req \
  -new \
  -nodes \
  -sha256 \
  -subj "/C=US/ST=None/L=None/O=None/CN=kube-apiserver" \
  -key certs/kube-apiserver.key -out certs/kube-apiserver.csr
```
对签名请求进行签名并生成证书：

```bash
openssl x509 \
  -req \
  -days 9999 \
  -extfile <\
(printf "subjectAltName=\
IP:127.0.0.1,\
DNS:localhost,DNS:kube-local") \
  -sha256 \
  -CA certs/kubernetes-ca.crt \
  -CAkey certs/kubernetes-ca.key \
  -set_serial 01  \
  -in certs/kube-apiserver.csr \
  -out certs/kube-apiserver.crt
```
现在我们需要创建用于kubernetes的administrator的client证书，首先生成私钥，命令如下：

```bash
openssl genrsa -out certs/admin.key 4096
```

创建签名请求：

```bash
openssl req \
  -new \
  -nodes \
  -sha256 \
  -subj "/C=US/ST=None/L=None/O=system:masters/CN=kubernetes-admin"\
  -key certs/admin.key \
  -out certs/admin.csr
```
签名并生成admin.crt证书：

```bash
openssl x509 \
  -req \
  -days 9999 \
  -sha256 \
  -CA certs/kubernetes-ca.crt \
  -CAkey certs/kubernetes-ca.key \
  -set_serial 01 \
  -extensions req_ext \
  -in certs/admin.csr \
  -out certs/admin.crt
```
到目前为止，我们已经有了intermediate kubernetes CA,apiserver的服务端证书和管理员admin的client证书。让我们运行一个实际的kube-apiserver服务来验证一下证书。首先我们需要运行etcd(无证书，到此还没有创建etcd相关证书)

```bash
docker run \
  -it \
  --rm \
  --name kube-etcd \
  -p 6443:6443 \
  --hostname=kube-local \
  gcr.io/google-containers/etcd:3.4.3 etcd \
  --listen-client-urls=http://0.0.0.0:2379 \
  --advertise-client-urls=http://kube-local:2379
```

以Docker容器的方式运行kube-apiserver:

```bash
docker run \
  -it \
  --rm \
  --name kube-apiserver \
  --network=container:kube-etcd  \
  --name kube-local \
  -v $(pwd)/certs:/certs \
  gcr.io/google-containers/kube-apiserver:v1.16.4 kube-apiserver \
  --anonymous-auth=false --bind-address=0.0.0.0 \
  --etcd-servers=http://kube-local:2379 \
  --tls-cert-file=/certs/kube-apiserver.crt \
  --tls-private-key-file=/certs/kube-apiserver.key \
  --client-ca-file=/certs/kubernetes-ca.crt \
  --authorization-mode=RBAC
```
接下来我们就可以是用kubectl尝试连接apiserver，但是在连接之前我们需要创建`kubeconfig`文件：

```bash
mkdir conf
export KUBECONFIG=conf/admin-local.conf
kubectl config set-cluster default-cluster \
  --server=https://localhost:6443 \
  --certificate-authority certs/kubernetes-ca.crt \
  --embed-certs
kubectl config set-credentials default-admin \
  --client-key certs/admin.key \
  --client-certificate certs/admin.crt \
  --embed-certs
kubectl config set-context default-system \
  --cluster default-cluster \
  --user default-admin
kubectl config use-context default-system
```
关于创建kubeconfig文件的更多信息，请参考连接:https://kubernetes.io/docs/setup/best-practices/certificates/#configure-certificates-for-user-accounts

接下来让我们检验一下：

```bash
kubectl get ns
NAME              STATUS   AGE
default           Active   13m
kube-node-lease   Active   13m
kube-public       Active   13m
kube-system       Active   13m
```
### 授权Authorization(RBAC)

你可能注意到了，上面kube-apiserver启动的时候指定了`--authorization-mode=RBAC`参数。这意味着每个已经通过了证书签名验证的客户端接下来需要进行授权。但是kubernetes如何理解具有admin.crt证书的client就是真正的administrator呢？

kubernetes会从证书的Subject字段获取此信息：

```bash
openssl x509 -in certs/admin.crt -text -noout | grep Subject
        Subject: ... O = system:masters, CN = kubernetes-admin
```
kubenetes通过从Subject字段读取"O"和"CN"的配置来区分用户组和用户。前面给admin用户创建签名请求的时候在subject字段指定了`O = system:masters,CN = kubernetes-admin`."O = system:masters"表示组名为"system:masters";而"CN = kubernetes-admin"表示用户名为"kubernetes-admin".

让我们看一下用户组"system:masters"赋予了哪些角色和用户：

```bash
kubectl get clusterrolebinding | grep admin
cluster-admin
kubectl get clusterrolebinding cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-01-19T19:17:23Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "93"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin
  uid: baab8407-59ce-486f-b98f-9f37a981276f
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

subject字段告诉我们"system:masters"组(O = system:masters)关联到了cluster-admin角色。

kubernetes的其他组件也采用类似的工作机制。

### kube-scheduler authorization kube-scheduler授权

```bash
kubectl get clusterrolebinding system:kube-scheduler -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-01-19T19:17:23Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-scheduler
  resourceVersion: "100"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/system%3Akube-scheduler
  uid: ff173e14-63f2-4dee-a13a-569d4493cbaf
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-scheduler
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-scheduler
```
kube-scheduler的client证书应该有subject字段为`CN=system:kube-scheduler`

### kube-controller-manager authorization kube-controller-manager授权

```bash
kubectl get clusterrolebinding system:kube-controller-manager -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-01-19T19:17:23Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-controller-manager
  resourceVersion: "98"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/system%3Akube-controller-manager
  uid: 89734662-8034-4d78-b213-987b7207d3c2
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-controller-manager
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: system:kube-controller-manager
```

kube-controller-manager 客户端证书的subject字段应该有`CN=system:kube-controller-manager`

### Full list of certificates全部证书列表

集群的其他组件还需要证书，下面连接是一个生成全部证书的脚本：

https://github.com/spender0/kubernetes-sandbox/blob/master/generate-certs.sh

下面的连接是一个docker-compose,在里面你可以找到不同的kubernetes集群中证书是如何安装的：

https://github.com/spender0/kubernetes-sandbox/blob/master/docker-compose.yml

更多关于证书要求的内容参考：https://kubernetes.io/docs/setup/best-practices/certificates/
更多关于手动运行kubernetes组件的内容参考：https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/



