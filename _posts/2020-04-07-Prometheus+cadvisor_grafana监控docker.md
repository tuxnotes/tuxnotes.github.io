---
layout: post
title: Prometheus+cadvisor+grafana监控docker
date: 2020-04-07
author: tux
tags: prometheus, docker
---

### 组件介绍

- prometheus：启发于google的borgmon监控系统，是开源的metrics监控告警系统。详情参考官网。
- cadvisor: CAdvisor是谷歌开发的用于分析运行中容器的资源占用和性能指标的开源工具。CAdvisor是一个运行时的守护进程，负责收集、聚合、处理和输出运行中容器的信息。
- grafana: 漂亮和优秀的数据展示组件。详情参考官网。

### 组件部署规划

所有需要被监控的运行docker的主机都要安装cadvirso。prometheus和grafana根据生产的实际需要进行部署和调整。

### 组价安装与部署

#### cadvisor
在docker宿主机上启动cadvisor容器即可：
```bash
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

#### prometheus

安装方法略过，可参考官网。这里说一下配置。可以采用两种组织配置文件的方式的一种：1.采用类似于nginx的include方式。当job较多的时候可采用这种方式，配置文件管理方便。2.将配置都写在同一个配置文件，即`prometheus.yml`。示例如下：
方法一：
```
vim /usr/local/prometheus/prometheus.yml 
  - job_name: cadvisor
    file_sd_configs:
    - files: ['/usr/local/prometheus/sd_config/docker-node.yml']
      refresh_interval: 3s

vim /usr/local/prometheus/sd_config/docker-node.yml
- targets:
  - 192.168.1.155:8080
  - 192.168.1.156:8080
  labels:
    type: docker
```
检查配置文件：
```bash
/usr/local/prometheus/promtool check config /usr/local/prometheus/prometheus.yml
```

方法二：由于篇幅关系，这里只列出部分配置。一下是prometheus.yml的部分内容：
```
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
 
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
 
    static_configs:
    - targets: ['192.168.247.211:9090']
  - job_name: 'docker'
    static_configs:
    - targets:
      - "192.168.247.211:8090"
      - "192.168.247.212:8090"
```
加载配置文件
```bash
kill -HUP `ps -ef |grep prometheus|grep -v grep|awk '{print $2}'`
```
表达式相关
```bash
容器CPU使用率:
sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) without (cpu)

查询容器内存使用量（单位：字节）:
container_memory_usage_bytes{image!=""}

查询容器网络接收量速率（单位：字节/秒）：
sum(rate(container_network_receive_bytes_total{image!=""}[1m])) without (interface)

查询容器网络传输量速率（单位：字节/秒）：
sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)

查询容器文件系统读取速率（单位：字节/秒）：
sum(rate(container_fs_reads_bytes_total{image!=""}[1m])) without (device)

查询容器文件系统写入速率（单位：字节/秒）：
sum(rate(container_fs_writes_bytes_total{image!=""}[1m])) without (device)
```
#### grafana

grafana的安装略过，可参考官网。这里说一下其配置。
可采用如下几个模板：
```
https://grafana.com/grafana/dashboards/893
https://grafana.com/grafana/dashboards/179
https://grafana.com/grafana/dashboards/193
```
其中893下载量较多，推荐使用。
导入模板：
- 在web UI界面点击左侧的加号，然后按提示要求导入模板并配置。
