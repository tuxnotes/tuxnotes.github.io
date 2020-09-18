---
layout: post
title: prometheus笔记
date: 2020-09-17
author: tux
tags: prometheus
---

# 1 组件

根据prometheus的架构图，主要组件如下：

- prometheus server:

 - retrieval: 抓取数据
 - TSDB：存储时序数据
 - http server

- push gateway: support for short-lived jobs
- alermanager:处理告警
- export
- web UI

# 2 prometheus配置文件

prometheus.yml配置文件主要包含以下三个部分：

- global: prometheus server的全局配置
- rule_files: 指定prometheus需要加载的rule文件的位置
- scrape_configs:配置prometheus需要监控那些资源

prometheus期望targe暴露的metric的URL上下文为/metrics

# 3 altermanager

prometheus的告警分成了两部分：
- 告警规则由prometheus server发送告警给altermanager
- altermanager管理告警，可选择silencing,inhibition,aggregation等处理方式，并通过其他方式发送告警

## 2.1 altermanager

altermanager处理由客户端程序如prometheus server发送过的告警信息。altermanager会对告警
信息进行去重，分组，或路由的管理。还可以对告警进行silencing和inhibition处理

### 2.1.1 Grouping 

将相似的告警信息进行分组到不通类别。这在大量系统同时故障并产生大量告警信息的场景是非常
有用的。

例如：当由于网络故障，集群几十台甚至上百台的机器发生故障，大量服务不能访问到数据库时。
但prometheus设置的是每个服务实例在不能访问数据库时都会发送一个告警。这样会导致大量的
告警信息发送到altermanager.

此时作为告警处理管理员希望看到的是这些告警都分到一类中，而不想看到每个服务实例的告警。
此时就可以配置altermanager使用集群或告警名称，发送一条简化版的告警通知。




