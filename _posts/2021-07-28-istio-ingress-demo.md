---
layout: post
title: istio之Ingress访问入口Demo
date: 2021-07-28
author: tux
tags: istio
---

# 1 前置条件

- Kubernetes环境，比如minikube，在线交互式环境以及其他多节点的Kubernetes环境
- istio安装完成

# 2 步骤

本例需要使用istio安装包中的sample目录中的yaml

## 2.1 创建网关

```bash
root@dev01:/home/tux/istio-1.10.2# kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml 
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
root@dev01:/home/tux/istio-1.10.2# kubectl get gw
NAME               AGE
bookinfo-gateway   7s
root@dev01:/home/tux/istio-1.10.2# kubectl get vs
NAME       GATEWAYS               HOSTS   AGE
bookinfo   ["bookinfo-gateway"]   ["*"]   29s
```
## 2.2 创建bookinfo相关的资源对象
```bash
root@dev01:/home/tux/istio-1.10.2# kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
root@dev01:/home/tux/istio-1.10.2# kubectl get -f samples/bookinfo/platform/kube/bookinfo.yaml
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/details   ClusterIP   10.110.173.94   <none>        9080/TCP   169m

NAME                              SECRETS   AGE
serviceaccount/bookinfo-details   1         169m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/details-v1   1/1     1            1           169m

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/ratings   ClusterIP   10.96.140.236   <none>        9080/TCP   169m

NAME                              SECRETS   AGE
serviceaccount/bookinfo-ratings   1         169m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ratings-v1   1/1     1            1           169m

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/reviews   ClusterIP   10.110.144.94   <none>        9080/TCP   169m

NAME                              SECRETS   AGE
serviceaccount/bookinfo-reviews   1         169m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/reviews-v1   1/1     1            1           169m
deployment.apps/reviews-v2   1/1     1            1           169m
deployment.apps/reviews-v3   1/1     1            1           169m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/productpage   ClusterIP   10.103.149.58   <none>        9080/TCP   169m

NAME                                  SECRETS   AGE
serviceaccount/bookinfo-productpage   1         169m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/productpage-v1   1/1     1            1           169m
```
bookinfo.yaml的内容如下
```bash
root@dev01:/home/tux/istio-1.10.2# cat samples/bookinfo/platform/kube/bookinfo.yaml
# Copyright Istio Authors
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

##################################################################################################
# This file defines the services, service accounts, and deployments for the Bookinfo sample.
#
# To apply all 4 Bookinfo services, their corresponding service accounts, and deployments:
#
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
#
# Alternatively, you can deploy any resource separately:
#
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l service=reviews # reviews Service
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l account=reviews # reviews ServiceAccount
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l app=reviews,version=v3 # reviews-v3 Deployment
##################################################################################################

##################################################################################################
# Details service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: details
  labels:
    app: details
    service: details
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: details
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-details
  labels:
    account: details
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
      - name: details
        image: docker.io/istio/examples-bookinfo-details-v1:1.16.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        securityContext:
          runAsUser: 1000
---
##################################################################################################
# Ratings service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: ratings
  labels:
    app: ratings
    service: ratings
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: ratings
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-ratings
  labels:
    account: ratings
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-v1
  labels:
    app: ratings
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ratings
      version: v1
  template:
    metadata:
      labels:
        app: ratings
        version: v1
    spec:
      serviceAccountName: bookinfo-ratings
      containers:
      - name: ratings
        image: docker.io/istio/examples-bookinfo-ratings-v1:1.16.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        securityContext:
          runAsUser: 1000
---
##################################################################################################
# Reviews service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-reviews
  labels:
    account: reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v1:1.16.2
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v2
  labels:
    app: reviews
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v2
  template:
    metadata:
      labels:
        app: reviews
        version: v2
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.16.2
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.16.2
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
##################################################################################################
# Productpage services
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
  labels:
    account: productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.16.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        securityContext:
          runAsUser: 1000
      volumes:
      - name: tmp
        emptyDir: {}
---
```
## 2.3 获取访问入口

首先执行如下命令
```bash
root@dev01:/home/tux/istio-1.10.2# kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
istio-egressgateway    ClusterIP      10.99.238.252    <none>        80/TCP,443/TCP                                                               13d
istio-ingressgateway   LoadBalancer   10.96.84.206     <pending>     15021:31449/TCP,80:30902/TCP,443:31697/TCP,31400:30445/TCP,15443:30169/TCP   13d
istiod                 ClusterIP      10.108.197.215   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        13d
```
因为Kubernetes环境是通过minikube搭建的，istio-ingressgateway服务的EXTERNAL-IP为pending，所以只能通过NodePort的方式访问，从上面的命令输出可以看出，80端口映射为主机的30902端口。
检查主机上是否已经监听
```bash
root@dev01:/home/tux/istio-1.10.2# ss -tln|grep 30902
LISTEN   0         128                  0.0.0.0:30902            0.0.0.0:*
```
所以在浏览器中输入localhost:30902/productpage，页面显示如下图所示：

![](/assets/img/productpage.png)

