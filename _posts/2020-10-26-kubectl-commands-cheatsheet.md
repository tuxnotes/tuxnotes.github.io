---
layout: post
title: kubectl commands cheatsheet
date: 2020-10-26
author: tux
tags: kubernetes
---

转自https://medium.com/faun/kubectl-commands-cheatsheet-43ce8f13adfb Kenichi Shibata

### impersonate a user and group

```bash
kubectl get pods --as-group="somecompany:somecompany-teamname" --as="test"
```

### Explain a resource
```bash
kubectl explain hpa
kubectl explain svc
```

### Get nodes region and zone
```bash
kubectl get nodes --label-columns failure-domain.beta.kubernetes.io/region,failure-domain.beta.kubernetes.io/zone
```

### Get Arch, OS, Instance type and node type if kops
```bash
kubectl get nodes -o wide -L beta.kubernetes.io/arch -L beta.kubernetes.io/os -L beta.kubernetes.io/instance-type -L  kops.k8s.io/instancegroup
kubectl get nodes -L beta.kubernetes.io/arch -L beta.kubernetes.io/os -L beta.kubernetes.io/instance-type -L  kops.k8s.io/instancegroup
```
### Get node version and name only
```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,VER:.status.nodeInfo.kubeletVersion
```
### Get scheduleable nodes
```bash
kubectl get nodes --output 'jsonpath={range $.items[*]}{.metadata.name} {.spec.taints[*].effect}{"\n"}{end}' | awk '!/NoSchedule/{print $1}'
```
### Get all deployments nameonly
```bash
kubectl get deployment -o=jsonpath={.items[*].metadata.name}
```
### Get one deployment only (first one)
```bash
kubectl get deployment -o=jsonpath={.items[0].metadata.name}
```
### Get all pods statuses only
```bash
kubectl get pods -o=jsonpath=‘{.items[*].status.phase}’ --all-namespaces
```
### Get pods qos
```bash
kubectl get pods --all-namespaces -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,QOS-CLASS:.status.qosClass
```
### Get images running
```bash
kubectl get pod -o=jsonpath='{.spec.containers[*].image}' --all-namespaces
kubectl get pod -o=custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[*].image --all-namespaces
```
### Where is my pod running
```bash
kubectl get pods -n sock-shop -l name=carts -o wide
```
### Check node/pod usage memory and cpu
```bash
kubectl top nodes
kubectl top pods
```
### Check health of etcd
```bash
kubectl get --raw=/healthz/etcd
```
### Check status of node autoscaler
```bash
kubectl describe configmap cluster-autoscaler-status -n kube-system
```
### Get where pods are running from nodenames
```bash
kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces
kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name --all-namespaces
```
sorting pods by nodenames:
```bash
kubectl get pods -o wide --sort-by="{.spec.nodeName}"
```
getting pods on nodes using label filter:
```bash
for n in $(kubectl get nodes -l your_label_key=your_label_value --no-headers | cut -d " " -f1); do 
        kubectl get pods --all-namespaces  --no-headers --field-selector spec.nodeName=${n} 
  done
 ```
 sorting pods by number of restarts:
 ```bash
 kubectl get pods --sort-by="{.status.containerStatuses[:1].restartCount}"
 ```
 filtering by nodeName using — template flag:
 ```bash
 $ kubectl get nodes
  NAME                         STATUS                     AGE
  ip-254-0-90-30.ec2.internal   Ready                      2d
  ip-254-0-90-35.ec2.internal   Ready                      2d
  ip-254-0-90-50.ec2.internal   Ready,SchedulingDisabled   2d
  ip-254-0-91-60.ec2.internal   Ready                      2d
  ip-254-0-91-65.ec2.internal   Ready                      2d
  $ kubectl get pods --template '{{range .items}}{{if eq .spec.nodeName "ip-254-0-90-30.ec2.internal"}}{{.metadata.name}}{{"\n"}}{{end}}}{{end}}'
  filebeat-000
  app-0000
  node-exporter-0000
  prometheus-000
```
### Check pods which are not Runnning
```bash
kubectl get pods --field-selector=status.phase!=Running --all-namespaces
```
### Sort Nodes by Role, Age and kubelet version
```bash
kubectl get nodes --sort-by={.metadata.labels."kubernetes\.io\/role"}
kubectl get node --sort-by={.status.nodeInfo.kubeletVersion}
watch kubectl get node --sort-by={.status.nodeInfo.kubeletVersion}
watch "kubectl get nodes --sort-by={.metadata.labels.\"kubernetes\.io\/role\"}"
kubectl get nodes --sort-by=".status.conditions[?(@.reason == 'KubeletReady' )].lastTransitionTime"
```
### Query apiservers
```bash
kubectl get --raw=/apis
kubectl get --raw=/logs/kube-apiserver.log
```
### Setup a deployment with limits and requests

- `kubectl run ken-test --image=kenichishibata/docker-curl -i --tty --limits='cpu=50m,memory=128Mi' --requests='cpu=50m,memory=128Mi'`
- `kubectl delete deployment ken-test`

### Get events for an individual resource
```bash
kubectl get event --field-selector=involvedObject.name=foo -w
```
### Get apiresources
- Check for an api resources available, this should show your crd api endpoints as well
- `kubectl api-resources`
- `kubectl api-versions`
- Check apiservices added (registered)
- kubectl get apiservices.apiregistration.k8s.io
- kubectl get apiservices.apiregistration.k8s.io v1beta1.metrics.k8s.io -o yaml
- Check hpa (maybe because you have custom metrics enabled in prometheus)?
- kubectl get hpa
- `kubectl get hpa --all-namespaces` kubectl get --raw /apis/metrics.k8s.io

### Kube Diff

`kubectl alpha diff -h`
