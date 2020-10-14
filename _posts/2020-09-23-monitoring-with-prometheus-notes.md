---
layout: post
title: monitoring with pormetheus
date: 2020-09-23
author: tux
tags: prometheus
---
# 1 监控简介

## 1.1 什么是监控

从技术角度看，监控是衡量和管理技术系统的工具和流程。监控将系统和应用程序生产的
指标转换为对应的业务价值。将指标转化为衡量用户体验的依据，该依据为业务提供反馈，以确保为客户提供了所需的产品。同时还提供了对技术的反馈，指出哪些组件不起作用或
导致服务质量下降。

实际上监控系统有如下两个“客户”:

- 技术
- 业务

### 1.1.1 技术作为客户

监控系统的第一个客户是技术，是运维工程师、devops或是SRE需要面对的。通过监控系统
了解技术环境，帮助检测、诊断和解决技术环境中的故障和问题。

Google SRE手册中有一个很棒的图标，显示了监控是构建和管理应用程序层次结构的基础。

### 1.1.2 业务作为客户

监控系统是为了支持业务，并确保业务持续开展。监控可以提供报告，是企业能进行良好的
产品和技术投资。还有助于企业衡量技术带来的价值。

## 1.2 监控基础知识

监控是管理机车设施和业务的核心工具。监控也是必须的，应该和应用程序一起构建和部署。

- 事后监控：监控与安全性一样，也是应用程序的核心功能。务必把应用程序的每个组件的监控指标考虑进来，千万不能等到项目结束或部署之前再做这件事。
- 机械式监控：常见的例子是只监控每台主机上的基础资源，二不监控主机上应用程序是否正常运行的关键服务。
- 不够准确的监控：如通过检查HTTP 200状态码来确定程序是否正常运行，这种方式只会告诉你应用程序正在响应请求，但不会反映出是否返回来正确但数据。因此要监控食物但内容和速率，二不是监控它运行但web服务器但运行时间
- 静态监控：使用静态阈值。为了更好地监控，需要查看数据窗口，而不是静态但时间点，需要使用只能但技术来分析指标和阈值
- 不频繁但监控：设置监控周期是一项挑战，但需要牢记但是存储足够多但历史数据可以有效地识别性能但问题和趋势。
- 缺少自动化或自服务：监控系统但实施和部署应尽可能但自动化

**良好的监控系统应能提供以下内容**：

- 全局视角，从最高曾（业务）依次展开。
- 协助故障诊断
- 作为基础设施、应用程序开发和业务人员的信息源
- 内置于应用程序设计、开发和部署的生命周期中
- 尽可能自动化， 并提供自服务

## 1.3 监控机制

### 1.3.1 探针和内省

监控应用程序主要有两种方法：探针(probing)和内省(introspection)。

**探针**:在应用程序外部，查询应用程序的外部特征：监听端口是否有响应并返回正确的数据或状态码。探针监控的一个例子是执行ICMP检查并确认可以收到响应。Nagios就是一个主要基于探针监控的监控系统。

**内省监控**：查看应用程序内部的内容。应用程序经过检测，并返回其状态、内部组件，或事物和四件性能的量度。这些数据可准确显示应用程序的运行方式，而不仅是其可用性或表面行为。内省监控可直接将事件、日志和指标发送到监控工具，也可将信息发送给状态或健康检查接口，然后由监控工具收集。

### 1.3.2 拉取和推送

pull和push两种方法各有利弊。Prometheus主要是基于拉取第监控系统，但也支持接受推送到网关到事件。

### 1.3.3 监控数据类型

- 指标：指标存储为时间序列数据，用于记录应用程序度量的状态.Prometheus主要关注收集时间序列数据。
- 日志：日志是应用程序发出的事件(通常是文本)。日志有助于故障诊断和调查。

## 1.4 指标

指标似乎始终是任何监控体系结构中最直接的部分。然后我们有时并没有投入足够的时间去理解指标，为什么收集，以及对指标做了什么。
许多监控框架的重点是故障检测，即检测是否发生了特定的系统事件或处于什么状态(这是Nagios的风格)。当收到特定系统事件的通知时，
才会查看收集到的指标，以找出发生的确切情况及原因。这个思路，指标被是为故障检测的副产品或者补充。

Prometheus改变了"指标作为补充"的观念，指标变成了监控工作流程中最重要的部分。Prometheus颠覆了以故障检测为中心的模型，指标
用来反映环境的状态、可用性以及性能。

### 1.4.1 什么是指标

指标是软件或硬件组件属性的量度。为了是指标有价值，我们会跟踪其状态，通常记录一段时间内的数据点。这些数据点称为观察点(observation)。观察点通常包括值、时间戳，有时也涵盖描述观察点的一系列属性(如源或标签)。观察的集合称为时间序列。

时间序列数据的典型示例是网站的访问或点击。定期收集有关网站点击量的观察点，记录点击次数和查看次数。还可能收集如访问来源、被访问
到的服务器或其他各种信息等的属性。

通常以固定的时间间隔收集数据，该时间间隔称为颗粒度（granularity）或分辨率(resolution),取值可以从1秒到5分钟，甚至更长。正确的
选择指标的可路堵至关重要，颗粒度太粗，则容易错过细节。颗粒度太细，则需要存储和分析大量数据。

### 1.4.2 指标类型

#### 测量型

测量型(gauge),这种类型是上下增减的数字，本质上是特定度量的快照。常见的监控指标如CPU、内存和磁盘使用率等都属于这个类型。对于
业务指标来说，指标可能是网站上的客户数量。

#### 计数型

计数型(counter)，这种类型岁时间增加而不会减少的数字。永远不会减少，但有时可以将其重置为零并再次开始递增。应用成俗和基础设施
的计数型示例包括系统正常的运行时间、设备收发包的字节数或登录次数。业务方面的示例可能是一个月内的销售数量或应用程序收到的订单
数量。

计数型指标的一个优势在于可以让你计算变化率。通过变化率可以理解许多有用的信息。如登录次数指标，通过计算变化率来查看每秒的登录
次数，有助于确定网站这段时间的受欢迎程度。

#### 直方图

直方图(histogram)是对观察点进行采样的指标类型，可以展现数据集的频率分布。将数据分组在一起并以这样的方式显示，这个被称为
"分箱"(binning)的过程可以直观地查看数值的相对大小。统计每个观察点并将其放入不通的桶中，这样可以产生多个指标：每个桶一个，
加上所有值的总和以及计数。

直方图可以很好地展现时间序列数据，尤其适用于数据的可视化(如应用程序延迟等)。

关于直方图的更多信息请参考：https://prometheus.io/docs/practices/histograms/

还有另一种指标类型，称为“摘要型”（summary），类似于直方图，但它还会计算百分位数。

### 1.4.3 指标摘要

通常单个指标价值很小，往往需要联合并可视化多个指标。这其中需要应用一些数学变换。如将统计还书应用于指标或指标组，常见函数如下：

- 计数：计算特定时间间隔内的观察点数。
- 求和：将特定时间间隔内所有观察点的值累计相加。
- 求平均值：提供特定时间间隔内所有值的平均值。
- 中间数：数值的几何中点，正好50%的数值位于它的前面，而另外50%位于它后面。
- 百分位数：度量占总数特定百分比的观察点的值。
- 标准差：显示指标分布中与平均值的标准差，这可以测量出数据集的差异和程度。标准差为0表示数据都等于平局值，较高的标准差意味着数据分布的范围很广。
- 变化率：显示时间序列中数据之间的变化程度。

### 1.4.4 指标聚合

除指标摘要，你可能希望能看到来自多个源的指标的聚合试图。如所有应用服务器的磁盘空间使用情况。指标聚合最典型的样式就是在一张图上
显示多个指标，这有助于你识别环境的发展趋势。如负载均衡器中的间歇性故障可能导致多个服务器的web流量下降，这通常比通过查看每个单独的指标更容易发现。

#### 平均值

监控领域，高峰或低谷可能被平均值掩盖。这些隐藏的异常值可能意味着，当我们认为大多数用户都享受这高质量的服务体验时，实际上很可能
并非如此。如果我们仅依靠平均值，则可能会认为应用程序的性能比真实情况要好得多。

#### 中间数

中间数处在所有数值的正中心：正好50%的数值位于它的前面，另外50%位于它后面。如果有奇数项个值，则处于中间位置的值即为中间数。
对于数据集(12、22、15、3、7、94、39)来说，中间数为3.如果从数据集中删除39，则数据集有偶数个值，其中间数是中间两个值的平均
值，即中间数为9.

识别性能问题的另一种常用技术是根据平均值来计算指标的标准差。

#### 标准差

标准差衡量的是数据集的变化或分布。标准差为0表示大部分数据接近平均值，标准差越大意味着数据越分散。标准差由正或负加上sigma
符号表示，例如，1 sigma表示与平均值有一个标准差。

许多监控方法都会利用经验法则，当出现超过平均值两个标准差的事务或事件时出发警报，以捕获性能的异常值。如果数据不是正态分布的，
那么最终的并标准差可能会误导你。

#### 百分位数

百分位数度量的是占总数特定百分比的观察点的值。本质上讲，它会展示数据集的分布。对于指标而言，百分位数很有意义，因为它们可以
清洗地展现数值的分布。如，一个事务的99百分位数为10毫秒。这很容易理解：99%的事务在10毫秒或更短事件内完成，1%的事务处理时间
超过10毫秒。

**百分位数是识别异常值的理想选择**。如果响应时间小于10毫秒表示网站上的一个良好体验，那么99%的用户都是这样的——但其中1%但
用户没有。一旦意识到这一点，你就可以专注于解决造成那1%但性能问题。

然而，百分位数并不是完美但。建议绘制集中指标组合，以获得更清晰但数据图。例如，在测量延迟时，最好可以展示以下几项内容：

- 50百分位数(或中间数)
- 99百分位数
- 最大值

## 1.5 监控方法论

这里主要介绍两种监控方法：

- Brendan Gregg但USE(Utilization,Saturation,Error)方法，侧重主机级别监控
- Google但四个黄金指标，专注于应用程序级别监控。

### 1.5.1 USE方法

USE是使用率、饱和度和错误但缩写。由Netflix的内核和性能工程师Brendan Gregg开发。USE方法建议创建服务器分析清单，以便快速识别问题。

USE方法可以概括为：针对每个资源，检查使用率、饱和度和错误。该方法对于监控那些受高使用率或饱和度的性能问题影响的资源来说是最有
效的。让我们快速查看每个术语的定义以帮助理解。

- 资源：系统的一个组件。在Gregg对模型的定义中，它是一个传统意义上的物理服务器组件，如CPU、磁盘等，但许多人也将软件资源包含在定义中。
- 使用率：资源忙于工作的平均时间。它通常用随时间变化的百分比表示。
- 饱和度：资源排队工作的指标，无法再处理额外的工作。通常用队列长度表示
- 错误：资源错误事件的计数

可以查看Brendan Gregg提供的Linux系统的参考示例[清单](http://www.brendangregg.com/USEmethod/use-linux.html)

### 1.5.2 Google但四个黄金指标

Google的四个黄金指标来自Google SRE手册，采用与USE类似的方法，指定需要监控的通用指标类型。但此方法中的指标类型主要关注的不是
系统级别的时间序列数据，更多是针对应用程序或面向用户的部分：

- 延迟：服务请求所花费的时间，需要区分成功请求和失败请求。例如，失败请求可能会以非常低的延迟返回错误结果
- 流量：针对系统，例如，每秒HTTP请求数，或者数据库系统的事务
- 错误：请求失败的速率，要么是HTTP 500错误等显式失败，要么是返回错误内容或无效内容等隐式失败，或者基于策略原因导致的失败——例如，强制要求响应时间超过30ms的请求视为错误
- 饱和度：应用程序有多“满”，或者受限的资源，如内存或IO。这还包括即将饱和的部分，例如正在快速填充的磁盘

Weaveworks团队开发了一个名为RED（Rate、Error和Duration）的[相关框架](https://rancher.com/red-method-for-prometheus-3-key-metrics-for-monitoring)

# 2 Prometheus简介

## 2.1 Prometheus起源

Prometheus的灵感来自谷歌的Borgmon。最初由前Google SRE Matt T.Proud开发。在Proud加入SoundCloud后，他与另一位工程师Julius Volz
合作开发了Prometheus，并最终于2015年1月将其发布。

与Borgmon一样，Prometheus主要用于提供近实时的、基于动态云环境和容器的微服务、服务和应用程序的内省监控。

**Prometheus专注于现在正在发生的事情，而不是追踪数周或数月前的数据**。它基于这样一个前提，即大多数监控查询和警报都是从最近的（通常是一天内的）数据中生成的。Facebook在其内部时间序列数据库Gorilla的论文[插图]中验证了这一观点。Facebook发现85%的查询是针对26小时内的数据。Prometheus假定你尝试修复的问题可能是最近出现的，因此最有价值的是最近时间的数据，这反映在强大的查询语言和通常有限的监控数据保留期上。

## 2.2 Prometheus数据模型

Prometheus收集时间序列数据，它使用一个多维时间序列数据模型。这个时间序列模型结合了时间序列名称和标签(label)的键/值对，
这些标签提供了维度。每个时间序列由时间序列名称和标签的组合唯一 标识。

### 2.3.1 指标名称

时间序列名称，描述数据的一般性质——如website_visits_total网站访问的总数。

名称包含ASCII字符、数字、下划线和冒号。

### 2.3.2 标签

标签为数据模型提供了维度。为特定时间序列添加上下文。如total_website_visits时间
序列可以使用网站名称、请求IP或其他特殊标识的标签。Prometheus课在一个时间序列、一组时间序列或所有相关的时间序列上进行查询。

#### 标签种类

- 插桩标签(instrumentation label)：来自被监控的资源——如对于与HTTP相关的时间序列，标签可能会显示所使用的特定HTTP动词。这些标签在诸如客户端或exporter抓取前会被添加到时间序列中。
- 目标标签(target label):更多地与架构相关——它们可能会识别时间序列所在的数据中。目标标签由Prometheus在抓取期间和之后添加。

时间序列由名称和标签标识。尽管从技术上讲，名称本身也是名为__name__的标签。如果在
时间序列中添加或更改标签，则Prometheus会将其视为新的时间序列。

label可理解为键/值形式的标签，并且新的标签会创建新的时间序列。标签名称可以包含
ASCII字符、数字和下划线。

带有__前缀的标签名称保留给Prometheus内部使用。

### 2.3.3 采样数据

时间序列的真实值是采样(sample)的结果，包含两部分：
- 一个float64类型的数值
- 一个毫秒精度的时间戳

### 2.3.4 符号表示

Prometheus时间序列的符号表示如下：

```
<time_series_name>{<label_name>=<label_value>, ...}
```

例如，带有标签的total_website_visits时间序列可能如下：

```
total_website_visits{site="MegaApp", location="NJ", instance="webserver", job="web"}
```

如上所示，首先是时间序列名称，后面跟着一组键/值对标签。通常所有时间序列都有一个
instance标签(标识源主机或应用程序)以及一个job标签(包好抓取特定时间序列的作业名称)。

这与OpenTSDB使用的符号大致相同，受到了Borgmon的影响。

### 2.3.5 保留时间

**Prometheus专为短期监控和报警需求而设计**。默认情况下时序数据保留15天。如果要
保留更长时间，建议将数据发送到远程的第三方平台。

# 3 Prometheus安装与配置

## 3.1 配置

Prometheus通过yaml文件来配置。默认配置大致如下：

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager: 9093

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090]
```

默认配置文件中定义了4个YAML块：global alerting rule_files scrape_configs

### 3.2.1 global

global控制Prometheus服务器行为的全局配置：

#### scrape_interval

指定应用程序或服务抓取数据的时间间隔。这个值是时间序列的颗粒度，即该序列中每个数据点所覆盖的时间段。

从特定位置收集指标时，可能会覆盖这个全局抓取间隔。强烈不建议这么做。**建议保持一个全局抓取时间，这确保所有时间序列具有相同的颗粒度**，并可以组合在一起计算。但使用了不同的数据间隔来收集数据时，则可能产生不和逻辑的结果。

>WARNING:仅设置抓取间隔为全局参数以保持颗粒度一致。

#### evaluation_interval

指定Prometheus评估规则的频率。目前主要有两种规则：记录规则(recording rule)和报警规则(alerting rule)

- recording rule:允许预先计算使用频繁且开销大的表达式，并将结果保存为一个新的时间序列数据。

- alerting rule:允许定义报警条件

Prometheus根据evaluation_interval的设置，每隔15s(重新)评估这些规则。

### 3.2.2 alerting

用来设置Prometheus的警报。警报是由名为Alertmanager的独立工具进行管理的。Alertmanager是一个可以集群化的独立警报管理工具。

在上面的默认配置中,alerting包含服务器的警报配置，其中alertmanagers块列出Prometheus使用的每个Alertmanager,static_configs块标识手动指定targets数组中配置的Alertmanager.

>NOTE:Prometheus还指出Alertmanager的服务发现功能，如通过查询外部源(Consul)来返回可用的Alertmanager列表，而不是手动指定。

### 3.2.3 rule_files

指定包含recording rule和alerting rule的文件列表

### 3.2.4 scrape_configs

指定Prometheus抓取的所有目标。Prometheus将抓取指标的数据源称为端点。为了抓取端点的数据，Prometheus定义了一个目标，目标包含的信息是抓取数据所必需的，如用到的标签，建立连接所需的身份验证，或其他定义数据抓取的信息。若干目标构成的组成为作业，作业里的每个目标都有一个名为实例(instance)的标签，用来唯一标识这个目标。

## 3.4 第一个指标

通过浏览http://localhost:9090/metrics并查看返回的内容

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summay
go_gc_duration_seconds{quantile="0"} 1.6166e-05 
go_gc_duration_seconds{quantile="0.25"} 3.8655e-05 
go_gc_duration_seconds{quantile="0.5"} 5.3416e-05 
....
```


go_gc_duration_seconds{quantile="0.5"} 5.3416e-05 
这个指标的名称是go_gc_duration_seconds,里面有一个标签quantile="0.5",表示这衡量的是第50百分位数，后面的数字是这个指标的值。

## 3.7 容量规划

Prometheus的性能很难估计，因为它在很大程度上取决于你的配置、所收集的时间序列的数量以及服务器上规则的复杂性。一般容量规划关注两个问题：**内存和磁盘**

### 3.7.1 内存

Prometheus在内存中做了很多工作。每个收集的时间序列、查询和记录规则都会消耗进程内存。关于Prometheus的容量规划的参考数据并不多，**但一个有用的、粗略的经验法则是将每秒收集的样本数诚意样本的大小**。使用以下查询语句来查看收本收集率：

```
rate(prometheus_tsdb_head_samples_append_total[1m])
```

这将显示最后一分钟添加到数据库的每秒样本率。如果想知道收集的指标数量，则使用以下语句：

```
sum(count by (__name__)({__name__=\~"\.\+"}))
```

这里使用sum聚合来计算所有匹配的指标的计数和，使用=~运算符和.+的正则表达式来匹配所有的指标。

每个样本的大小通常为1到2字节，采用保守的估计，按照2个字节计算。假设在12小时内每秒收集100000个样本，则可采用如下方式计算内容使用情况：

```
100000 * 2 bytes * 43200 seconds
```
结果大概是8.64GB的内存。

还需要考虑在查询和记录规则方面的内存使用情况。这个不太好计算，并且依赖于许多其他变量，建议根据内存使用情况灵活调整。你可以通过检查process_resident_memory_bytes指标来查看Prometheus进程的内存使用情况。

### 3.7.2 磁盘

磁盘使用量收存储的时间序列数量和这些时间序列的保留时间限制。默认情况下，指标会在本地时间序列数据库中存储15天。数据库的位置和保留时间由命令行选项控制：

- storage.tsdb.path:控制时间序列数据库位置
- storage.tsdb.retention:控制时间序列的保留期，默认值为15d，表示15天。

**建议采用SSD作为时间序列数据库的磁盘**

对于每秒10万个样本的示例，按时间序列收集的每个样本在磁盘上占用大约1到2个字节，假设每个样本有2个字节，那么保留15天的时间序列意味着要大约259GB的磁盘。


# 4 监控主机和容器

为了确定要监控的内容，回顾USE监控方法，以帮助确定正确的监控指标。

## 4.1 节点监控

Node-Exporter，用go编写，提供了一个可用于收集各种助剂指标数据的库。它还有一个textfile收集器，允许你导出静态指标，这对发送有关节点的信息很有帮助，此外他还可以从批处理作业导出指标。

### 4.1.1 安装node exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz
tar -xzf node_export-*
sudo cp node_exporter-*/node_exporter /usr/local/bin/
```

在centos和Fedora平台上，可以通过COPR软件包安装。

>注意：使用配置管理工具是运行和安装exporter的最佳方式。这种方式可以轻松管理配置并提供自动化和服务管理。

### 4.1.2 配置node exporter

通过参数来对node_exporter进行配置，使用--help查看完整的参数列表

默认情况下，node_exporter在端口9100上运行，并在路径/metrics上暴露指标。你可以通过--web.listen-address和--web.telemetry-path参数来设置端口和路径，如下所示：

```bash
$ node_exporter --web.listeb-address=":9600" --web.telemetry-path="/node_metrucs"
```

还有些参数用于控制收集器的启用和关闭，许多收集器默认是启用的，使用no-前缀来修改状态。如，暴露/proc/net/arp统计信息的arp收集器默认是启用的，要禁用此收集器，需要运行如下命令：

```bash
$ node_exporter --no-collector.arp
```

### 4.1.3 配置textfile收集器

textfile收集器允许我们暴露自定义指标，这些自定义指标可能是批处理或cron作业无法抓取的，可能是没有exporter的源，甚至可能是为助剂提供上下文的静态指标。

收集器通过扫描指定目录中的文件，提取所有格式为Prometheus指标的字符串，然后暴露它们以便抓取。

现在设置收集器，首先创建一个目录来保存指标定义文件。

```bash
mkdir -p /var/lib/node_exporter/textfile_collector
```

在上面的目录中创建一个新的指标，指标在以.prom结尾的文件中定义，并使用Prometheus特定文本格式。
特定文本格式允许我们指定Prometheus支持的所有指标类型：计数型、测量型、计时型等

使用此格式创建一个包含有关此主机的元数据指标。

```
metadata{role="docker_server",datacenter="NJ"} 1
```
可以看到它包含一个指标名称（metadata）和两个标签。标签role定义节点的角色。标签的值为docker_server。标签datacenter定义主机的地理位置。最后，指标的值为1，因为它不是计数型、测量型或计时型的指标，而是提供上下文。

将这个指标添加到textfile_collector目录下的metadata.prom文件中。

```bash
$ echo 'metadata{role="docker_server",datacenter="NJ"} 1' | sudo tee /var/lib/node_exporter/textfile_collector/metadata.prom
```
在这里，将指标传递到名为metadata.prom的文件中。

>在真实环境中，建议使用配置管理工具来编辑该文件。例如，在配置新主机时，可以从模板创建元数据指标，这可以自动对主机和服务进行分类。

要启用textfile收集器，不需要配置参数，它默认就会被加载。但需要指定textfile_exporter目录，以便Node Exporter知道在哪里可以找到自定义指标。为此，需要指定--collector.textfile.directory参数.

### 4.1.4 启用systemd收集器

systemd收集器记录systemd中的服务和系统状态。它收集了很多指标，但我们并不想收集systemd管理的所有内容，而只想收集某些关键服务。因此，我们可以将特定服务列入白名单，只收集以下服务的指标：
- docker.service
- ssh.service
- rsyslog.service

使用--collector.systemd.unit-whitelist参数进行配置，它会匹配systemd的正则表达式。

### 4.1.5 运行node exporter

```bash
$ node_exporter --collector.textfile.directory /var/lib/node_exporter/textfile_collector --collector.systemd --collector.systemd.unit-whitelist="(docker|ssh|rsyslog).service"
```

### 4.1.6 抓取node exporter

要获取新数据，需要为配置添加另一个作业。这里给新作业起名为node，并将继续使用static_configs来添加单个目标，而不是使用服务发现。

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['138.197.26.39:9100','138.197.30.147:9100','138.197.30.163:9100']
```
### 4.1.7 过滤收集器

Node Exporter可以返回很多指标，也许你并不想把它们全部收集上来。除了通过本地配置来控制Node Exporter在本地运行哪些收集器之外，Prometheus还提供了一种方式来限制收集器从服务器端实际抓取的数据，尤其是在你无法控制正抓取的主机的配置时，这种方式非常有帮助。
Prometheus通过添加特定收集器列表来实现作业配置。


```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['138.197.26.39:9100','138.197.30.147:9100','138.197.30.163:9100']
    params:
      collect[]:
        - cpu
        - meminfo
        - diskstats
        - netdev
        - netstat
        - filefd
        - filesystem
        - xfs
        - systemd
```
将被抓取的指标限制在上面的收集器列表中，使用params块中的collect[]列表指定，然后将它们作为URL参数传递给抓取请求。

```bash
$ curl -g -X GET http://138.197.26.39:9100/metrics?collect[]=cpu
```
这将返回Node Exporter基本指标，如在Prometheus服务器上看到的Go指标，以及CPU收集器生成的指标，所有其他指标都将被忽略。

## 4.2 监控docker

Prometheus提供了几种方法来监控Docker，包括一些自定义exporter。然而，这些exporter一般都不会用到，推荐的方法是使用Google的cAdvisor工具。cAdvisor作为Docker容器运行，单个cAdvisor容器返回针对Docker守护进程和所有正在运行的容器的指标。Prometheus支持通过它导出指标，并将数据传输到其他各种存储系统，如InfluxDB、Elasticsearch和Kafka。

### 4.2.1 运行cAdvisor

由于cAdvisor只是Docker主机上的另一个容器，因此我们可以使用docker run命令启动它

```bash
$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker:/var/lib/docker:ro \
  --volume=/dev/disk:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  google/cadvisor:latest
```
在容器内挂载了几个目录。目录分为两种类型，第一种是只读的，cAdvisor将从中收集数据，例如/sys目录.第二种类型是可读写的，是Docker套接字的挂载，通常位于/var/run目录中。
将容器内部的8080端口映射到主机上的8080端口，如果浏览端口8080上的路径/metrics，则将看到暴露的内置Prometheus指标。

### 4.2.2 抓取cAdvisor

再次使用static_configs添加单个目标，而不是使用任何类型的服务发现。

```yaml
scrape_configs:
....
  - job_name: 'docker'
    static_configs:
      - targets: ['138.197.26.39:8080','138.197.30.147:8080','138.197.30.163:8080']
```
但重要的是，在继续之前，需要首先了解抓取的工作原理以及标签的生命周期。

## 4.3 抓取的生命周期

先看看抓取本身的生命周期，然后再转到标签的生命周期。**在每个scrape_interval期间（示例中是15秒），Prometheus都会检查执行的作业。这些作业将生成目标列表，即服务发现过程**。到目前为止，我们的示例都是已手动指定的静态配置的主机，以及一些其他服务发现机制，例如从文件加载目标或查询API等。

服务发现返回一个目标列表，其中包含一组称为元数据的标签，这些标签以__meta_为前缀。每个服务发现机制都有不同的元数据，例如，AWS EC2的发现机制会返回名为__meta_ec2_availability_zone的标签中的实例所在可用区（availability zone）。

服务发现还会根据目标的配置来设置其他标签，这些标签带有__的前缀和后缀，包括__scheme__、__address__和__metrics_path__。这些标签包含目标的模式（http或https）、目标的地址以及指标的具体路径。

每个标签通常都有一个默认值。例如，__metrics_path__默认为/metrics，__scheme__默认为http。此外，如果路径中存在任何URL参数，则它们的前缀会设置为__param_*。

配置标签会在抓取的生命周期中重复利用以生成其他标签。例如，指标上instance标签的默认内容是__address__标签的内容.

**为什么到现在我们都没有在界面上看到任何带__前缀和后缀的标签呢？这是因为有些标签在生命周期的后期被删除了，并且所有这些标签都被专门排除掉，不在Web UI上显示。**

目标列表和标签会返回给Prometheus，其中一些标签可以在配置中被覆盖，例如，通过metrics_path参数设置的指标路径以及通过scheme参数指定的模式。

```yaml
scrape_configs:
  - job_name: 'node'
    schema: https
    metric_path: /moremetrics
    static_configs:
      - targets: ['138.197.26.39:9100','138.197.30.147:9100','138.197.30.163:9100']
```
这里，我们将schema覆盖为https，将metrics_path覆盖为/moremetrics。

Prometheus提供了可以重新标记目标的机会，并可能使用你的服务发现所添加的一些元数据。你还可以过滤目标，以删除或保留特定条目。

然后就是真正的数据抓取，以及指标返回。当指标被抓取时，你将拥有最后一次机会在将它们保存到服务器之前重新标记并过滤。

整个抓取的生命周期总结如下：

服务发下 ---> 配置 ---> 重新标记relabel_configs ---> 抓取 ---> 重新标记metric_relabel_configs

## 4.4 标签

标签提供了时间序列的维度。它们可以定义目标，并为时间序列提供上下文。但最重要的是，结合指标名称，它们构成了时间序列的标识，如果它们改变了，那么时间序列的标识也会跟着改变。更改或添加标签会创建新的时间序列。

这意味着应该明智地使用标签并尽可能保持不变。如果不遵守这一规定，则可能产生新的时间序列，从而创建出一个动态的数据环境，使监控的数据源难以跟踪。假如你配置了依赖于指标的标签来评判的警报。如果更改或添加标签，那么警报将变为无效。这同样适用于历史时间序列数据：更改或添加标签，先前时间序列的跟踪数据会丢失，破坏已有的图像和表达式，并可能导致混乱。

### 4.4.1 标签分类

那么，何时应该添加标签以及应该添加哪些标签呢？一个设定分类标准的好办法是利用拓扑标签和模式标签。

拓扑标签（topological label）通过其物理或逻辑组成来切割服务组件，例如我们在上面看到的datacenter标签。每个指标都天然带有两个拓扑标签：job和instance。标签job是根据抓取配置中的作业名称设置的。我们倾向于使用job来描述正在监控的事物的类型。在之前的Node Exporter作业中，我们将其命名为node。这会为每个Node Exporter指标打上job标签node。instance标签可以标识目标，它通常是目标的IP地址和端口，并且来自__address__标签。

模式标签（schematic label）是url、error_code或user之类的东西，它们允许你将拓扑中同一级别的时间序列匹配在一起，例如创建数据间的比率。

如果你需要添加额外的标签，则可考虑如下的层次结构：

数据中心/位置/区域 ---> 环境：开发/预发布/生产 ---> 服务/应用程序

### 4.4.2 重新标记

既然已经精心设计了标签，为什么还需要重新标记（relabel）呢？用一个词来回答就是：控制。在一个集中的复杂监控环境中，有时你无法控制监控的所有资源以及所有暴露的监控数据。通过重新标记，可以控制、管理并标准化环境中的指标。一些最常见的用例是：

- 删除不必要的指标
- 从指标中删除敏感或不需要的标签
- 添加、编辑或修改指标的标签值或标签格式

请记住，我们有两个阶段可以重新标记。第一个阶段是对来自服务发现的目标进行重新标记，这对于将来自服务发现的元数据标签中的信息应用于指标上的标签来说非常有用。这是在作业内的relabel_configs块中完成的，我们将在下一章介绍更多相关内容。

第二个阶段是在抓取之后且指标被保存于存储系统之前。这样，我们就可以确定哪些指标需要保存、哪些需要丢弃以及这些指标的样式。这是在我们作业内的metric_relabel_configs块中完成的。

**在抓取之前使用relabel_configs，在抓取之后使用metric_relabel_configs**

cAdvisor收集了大量数据，但并非所有数据都是有用的。因此，如何在它们进入存储之前删除其中一些指标，以避免占用不必要的空间。

例如下面的配置通过重新标记来删除指标

```yaml
  - job_name: 'docker'
    static_configs:
      - targets: ['138.197.26.39:8080','138.197.30.147:8080','138.197.30.163:8080']
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: '(container_tasks_state|container_memory_failures_total)'
        action: drop
```

这里，设置了名为docker的作业。在static_configs块之后，添加了一个新的块metric_relabel_configs，其中可以指定一系列相关操作.

#### 删除指标

使用source_labels参数选择要操作的指标，并且还需要一组标签名称。在示例中我们使用__name__标签，__name__标签是表示指标名称的预留标签。因此，docker作业的源标签是所有从cAdvisor抓取的指标的连接名称。

多个标签通过分隔符连接在一起，默认分隔符为“;”，也可以使用separator参数覆盖分隔符配置。

例如定义__name__标签值使用“,”进行分隔：

```yaml
    metric_relabel_configs:
      - source_labels: [__name__]
        separator: ','
        regex: '(container_tasks_state|container_memory_failures_total)'
        action: drop
```

接下来，指定一个正则表达式来搜索连接后的指标名称并进行匹配。正则表达式使用RE2表达式语法,这也是Go的正则表达式库RegExp使用的语法。

网上有一些很好的正则表达式测试工具可以参考：http://www.regexplanet.com/advanced/golang/index.html

正则表达式包含在regex参数中，它匹配并捕获两个指标：

- container_tasks_state
- container_memory_failures_total

如果指定了多个源标签，则使用分隔符隔开每个正则表达式，如：

```
regex1;regex2;regex3
```

然后执行action参数中指定的操作。在示例中，这两个指标都包含大量的时间序列，但是可能仅有部分数据有价值，所以我们使用了drop操作，这将在数据存储之前删除指标。其他操作（如keep）则会保留与正则表达式匹配的指标，并删除所有其他指标。

#### 替换标签值

许多cAdvisor指标都有一个id标签，其中包含正在运行的进程的名称。如果该进程是一个容器，那么会看到类似如下的信息：

```
id="/docker/6fca30023..."
```

上面的内容有些冗长，想要从中获取容器ID，然后将其放入一个新标签container_id中，可通过重新标记实现：

```yaml
metric_relabel_configs:
- source_labels: [id]
  regex: '/docker/([a-z0-9]+);'
  replacement: '$1'
  target_label: container_id
```

>NOTE:重新标记是按顺序执行的，使用配置文件中自上而下的顺序。

源标签是id。然后，指定匹配并捕获容器ID的正则表达式（regex）。replacement字段包含新标签值，在示例中使用的是匹配的结果$1。然后，在target_label参数中指定捕获信息的目标，此处为container_id。

你可能会注意到我们没有为此指定action。这是因为默认操作是replace，如果没有指定操作，那么Prometheus将假定你要执行替换操作。

Prometheus还有一个参数honor_labels，当你尝试覆盖并添加已存在的标签时，这个参数会控制冲突行为。假设你已经抓取的数据有一个名为job的标签。默认情况下honor_labels为false，Prometheus将通过在其前面添加exported_前缀来重命名现有标签。所以我们的job标签将变成exported_job。如果honor_labels被设置为true，则Prometheus会保留标签，并忽略服务器上的任何重新标记行为。

#### 删除标签

删除标签通常用于隐藏敏感信息或简化时间序列。示例中将删除kernelVersion标签，隐藏docker助剂的内核版本。

```yaml
metric_relabel_configs:
  - regex: 'kernelVersion'
    action: labeldrop
```
上面的配置将删除与正则表达式匹配的所有标签。此操作还有一个对应的反向操作labelkeep,它将保留与正则表达式匹配的标签，并删除所有其他标签。

>WARNING:标签是时间序列的唯一性约束。如果你删除标签并导致时间序列重复，那么系统可能会出现问题！

## 4.5 node exporter和cAdvisor指标

### 4.5.1 USE方法

USE方法建议收集并关注使用率、饱和度和错误指标，以帮助进行性能诊断。

#### CPU使用率

从Prometheus Web界面查询该指标，导航到http://localhost:9090/graph，从指标下拉列表中选择node_cpu_seconds_total，然后单击Execute,得到node_cpu_seconds_total的列表。

这个指标列表并不是特别有用。对于任何性能分析，都需要使用PromQL将其转化为更有用的指标。这里真正想要的是获得每个实例的CPU使用百分比，但要实现这一点，需要稍微处理下指标，我们可以通过一系列PromQL计算来实现这一结果。

首先计算每种CPU模式的每秒使用率。PromQL有一个名为irate的函数，用于计算范围向量中时间序列增加的每秒即时速率。让我们对node_cpu_seconds_total指标使用irate函数，在查询框中输入：

```
irate(node_cpu_seconds_total{job="node"}[5m])
```
然后单击Execute，将从node作业返回每个CPU在每种模式下的列表，表示为5分钟范围内的每秒速率。但这仍然没有太大帮助，还需要聚合不同CPU和模式间的指标。

为此，可以使用avg或average运算符以及我们在第3章介绍的by子句。

```
avg(irate(node_cpu_seconds_total{job="node"}[5m])) by (instance)
```
这将生成三个新指标，分别表示每台主机的平均CPU使用率。但这个指标还是不太准确，它仍然包括idle的值，并且它没有表示成百分比的形式。我们将查询每个实例的idle使用率，因为它已经是一个比率，将它乘以100可以转换为百分比。

```
avg (irate(node_cpu_seconds_total{job="node",mode="idle"}[5m])) by (instane) * 100
```

在这里，为irate查询添加了一个值为idle的mode标签，表示仅查询idle数据。我们按实例对结果进行了平均，并且还乘上100以转换为百分比的形式，这样就得到了每个主机5分钟范围内idle使用率的平均百分比。用100减去这个值，结果就是CPU使用率的百分比，如下所示：

```
100 - avg (irate(node_cpu_seconds_total{job="node",mode="idle"}[5m])) by (instane) * 100
```
现在我们有三个指标，每台主机一个，展示了5分钟范围内平均使用CPU的百分比.

#### CPU饱和度

在主机上获得CPU饱和的一种方法是跟踪平均负载，实际上它是将主机上的CPU数量考虑在内的一段时间内的平均运行队列长度。平均负载少于CPU的数量通常是正常的，长时间内超过该数字的平均值则表示CPU已饱和。

要查看主机的平均负载，我们可以使用node_load*指标，它们显示了1分钟、5分钟和15分钟的平均负载。我们将使用1分钟的平均负载：node_load1。

这里我们用idle的mode来计算node_cpu_seconds_total时间序列的出现次数，然后使用by子句从结果向量中删除instance之外的所有标签，这会为我们提供一个包含每台主机CPU数量的主机列表:

```
count by (instance)(node_cpu_seconds_total{mode="idle"})
```
可以看到3个节点各有2个CPU.

>NOTE:由于我们也在收集Docker指标，因此也可以使用cAdvisor的指标machine_cpu_cores来快速获取CPU数量。

然后我们可以将此计数与node_load1指标结合起来，如下所示：

```
node_load1 > on (instance) 2 * count by (instance)(node_cpu_seconds_total{mode="idel"})
```

这里我们查询的是1分钟的平均负载超过主机CPU数量的两倍的结果。这并不一定是一个问题，但是我们将在第6章介绍如何将它转换为警报，以便在出现问题时及时通知。

>NOTE:我们将跳过USE中的CPU错误，因为我们不太可能收集到任何有用的数据。

#### 内存使用率

重点关注node_memory指标的一个子集以获取使用率：
- node_memory_MemTotal_bytes：主机上的总内存。
- node_memory_MemFree_bytes：主机上的可用内存。
- node_memory_Buffers_bytes：缓冲缓存中的内存。
- node_memory_Cached_bytes：页面缓存中的内存。

所有这些指标都是以字节为单位来表示的。

将node_memory_MemFree_bytes、node_memory_Cached_bytes和node_memory_Buffers_bytes指标的值相加，这代表我们主机上的可用内存。使用这个值和node_memory_MemTotal_bytes指标来计算可用内存的百分比，使用以下查询：

```
(node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_momery_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
```
这将生成三个指标，显示每个主机使用的内存百分比.在这里，不需要使用by子句来保留不同的维度，因为指标具有相同的维度标签，每个指标都会依次应用查询。

#### 内存饱和度

通过检查内存和磁盘的读写来监控内存饱和度。可以使用从/proc/vmstat收集的两个Node Exporter指标：

- node_vmstat_pswpin：系统每秒从磁盘读到内存的字节数。
- node_vmstat_pswpout：系统每秒从内存写到磁盘的字节数。

两者都是自上次启动以来的字节数，以KB为单位。

为了获得饱和度指标，对每个指标计算一分钟的速率，将两个速率相加，然后乘以1024以获得字节数。创建一个查询来执行此操作。
```
1024 * sum by (instance) ((rate(node_vmstat_pgpgin[1m]) + rate(node_vmstat_pgpgout[1m])))
```

然后，我们可以对此设置图形化展示或者警报，以识别行为不当的应用程序主机。

#### 磁盘使用率

对于磁盘，我们只测量磁盘使用情况而不是使用率、饱和度或错误。这是因为在大多数情况下，它是对可视化和警报最有用的数据。Node Exporter的磁盘使用指标位于以node_filesystem为前缀的指标列表中。

但是，与内存指标不同，我们在每个主机上的每个挂载点都有文件系统指标。所以我们添加了mountpoint标签，特别是根文件系统“/”挂载。这将在每台主机上返回该文件系统的磁盘使用指标。

如果想要或需要监控特定挂载点，那么我们可以为其添加查询。要监控/data挂载点，我们可以使用：

```
(node_filesystem_size_bytes{mountpoint="/data"} - node_filesystem_free_bytes{mountpoint="/data"}) / node_filesystem_size_bytes{mountpoint="/data"} * 100
```
或者可以使用正则表达式来匹配多个挂载点。

```
(node_filesystem_size_bytes{mountpoint=~"/|/run"} - node_filesystem_free_bytes{mountpoint=~"/|/run"}) / node_filesystem_size_bytes{mountpoint=~"/|/run"} * 100
```

>NOTE:不能使用正则表达式来匹配空字符串。

将操作符从=更改为=~，这告诉Prometheus右侧值是正则表达式。然后我们匹配/run和/根文件系统。正则表达式中还有表示不匹配的运算符。

这仍然是一个相当经典的磁盘使用量指标，它告诉我们文件系统当前使用的百分比。在许多情况下，这些数据是无用的。例如，如果磁盘以每年1%的速度增长，那么对于1 GB文件系统来说80%的使用可能根本不是一个问题。但是对于1 TB文件系统，如果以每10分钟10%的速度增长，那么10%的使用则可能面临磁盘写满的严重风险。对于磁盘空间，我们确实需要了解指标的趋势和方向。我们通常要回答的问题是：“考虑到现在磁盘的使用情况，以及它的增长，我们会在什么时候耗尽磁盘空间？”

实际上Prometheus提供了一种机制，通过一个名为predict_linear的函数，我们可以构造一个查询来回答这个问题。我们来看一个例子：

```
predict_linear(node_filesystem_free_bytes{mountpoint="/"}[1h], 4 * 3600) < 0
```

这里指定的是根文件系统node_filesystem_free_bytes{mountpoint="/"}，也可以通过指定作业名称或使用正则表达式来选择所有文件系统。


```
predict_linear(node_filesystem_free_bytes{job="node"}[1h], 4 * 3600) < 0
```
我们选择一小时的时间窗口[1h]，并将此时间序列快照放在predict_linear函数中。该函数使用简单的线性回归，根据以前的增长情况来确定文件系统何时会耗尽空间。该函数参数包括一个范围向量，即一小时窗口，以及未来需要预测的时间点。这些都是以秒为单位的，因此这里使4*3600秒，即四小时。最后<0过滤出小于0的值，即文件系统空间不足。

因此，如果基于最后一小时的增长历史记录，文件系统将在接下来的四个小时内用完空间，那么查询将返回一个负数，然后我们可以使用它来触发警报。我们将在第6章看到这个警报是如何工作的。

### 4.5.2 服务状态

systemd收集器的数据展示了主机上的服务状态和其他各种systemd配置。服务的状态在node_systemd_unit_state指标中暴露出来。对于收集的每个服务和服务状态都有一个指标。在示例中，我们只收集Docker、SSH和RSyslog守护进程的指标。

下面的查询为每个潜在的服务和状态（failed、inactive、active等）的组合生成指标，其中表示当前服务状态的指标的值为1。

```
node_systemd_unit_state{{name="docker.service"}}
```

可以通过添加state标签来进一步缩小搜索范围，仅返回active状态的数据。


```
node_systemd_unit_state{{name="docker.service",state="active"}
```

或者搜索值为1的所有指标，这将返回当前服务的状态。

```
node_systemd_unit_state{name="docker.service"} == 1
```
**只有处于active状态的指标的值为1**

使用比较运算符“==”，查询将返回值为1、name标签为docker.service的所有指标。
**将使用systemd指标来监控主机上的服务可用性**，例如Docker守护进程。第6章会介绍如何对此发出警报。

### 4.5.3 可用性和up指标

对于每个实例抓取，Prometheus都会存储下面的时间序列样本：

```
up{job="<job-name>", instance="<instance-id>"}
```

如果实例是健康的，则指标设置为1，即数据抓取成功返回。如果抓取失败，则设置为0。指标使用作业名称job和时间序列的实例instance来进行标记。
许多exporter都有特定的指标，旨在确定最后一次成功的数据抓取。

>NOTE:不能重新标记自动填充的指标，如up指标，因为它们是在重新标记阶段之后生成的。

### 4.5.4 metadata指标

前面使用Node Exporter的textfile收集器创建了metadata指标

```
metadata{role="docker_server",datacenter="NJ"} 1
```

该指标提供资源的上下文信息，如角色docker_server、主机的位置datacenter。尽管这些数据本身很有用，但为什么又要创建一个单独的指标而不是仅将这些作为标签添加到主机的指标中呢？

**更改标签或添加新标签都会创建新的时间序列。**这意味着我们应该谨慎地使用标签，并且应尽可能保持不变。因此，我们不是使用一组完整的标签集来修饰每个时间序列，而是创建一个时间序列，我们可以使用它来查询特定类型或类别的资源。

如何利用该指标上的标签呢。假设我们只想从某个特定的数据中心或一组数据中心选择指标。我们可以快速找到所有主机，例如，查询不在新泽西（NJ）的数据中心：

```
metadata{datacenter != "NJ"}
```
## 4.6 查询持久性

到目前为止，只是在表达式浏览器中运行查询。虽然查看该查询的输出很方便，但结果仍然是临时存储在Prometheus服务器上。可以通过以下三种方式使查询持久化：

- 记录规则：根据查询创建新指标。
- 警报规则：从查询生成警报。
- 可视化：使用Grafana等仪表板可视化查询。

### 4.6.1 记录规则

记录规则是一种根据已有时间序列计算新时间序列（特别是聚合时间序列）的方法，这样做的目的是：

- 跨多个时间序列生成聚合。
- 预先计算消耗大的查询。
- 产生可用于生成警报的时间序列。

### 4.6.2 配置记录规则

记录规则存储在Prometheus服务器上，位于Prometheus服务器加载的文件中。规则是自动计算的，频率则由prometheus.yml配置文件的global块中的evaluation_interval参数控制。

规则文件在Prometheus配置文件的rules_files块中指定。

在prometheus.yml文件的同一文件夹中创建一个名为rules的子文件夹，用于保存我们的记录规则。我们还将为节点指标创建一个名为node_rules.yml的文件。Prometheus规则与Prometheus配置一样是通过YAML编写的。

>WARNING:YAML规则已在Prometheus 2.0中更新。早期的版本使用了不同的结构，一些旧的规则文件在Prometheus 2.0或更高版本中会失效，可以使用promtool来升级旧规则文件。关于升级过程参考链接：https://www.robustperception.io/converting-rules-to-the-prometheus-2-0-format/

创建一个记录规则文件node_rules.yml

```bash
$ mkdir -p rules
$ cd rules
$ touch node_rules.yml
```

在rule_configs块中添加这个文件

```
rule_files:
  - "rules/node_rules.yml"
```
### 4.6.3 添加记录规则

让我们把CPU、内存和磁盘计算转换为记录规则。我们有很多要监控的主机，所以我们要对所有节点预先计算这三个指标的查询，这样就可以将这些计算作为指标，然后可以设置警报或者通过Grafana等仪表板进行可视化。

首先是CPU计算的记录规则：

```
groups:
- name: node_rules
  rules:
  - record: instance:node_cpu:avg_rate5m
    expr: 100 - avg (irate(node_cpu_seconds_total{job="node",mode="idle"}[5m])) by (instance) * 100
```

**记录规则在规则组中定义**，这里的规则组叫作node_rules。**规则组名称在服务器中必须是唯一的**。规则组内的规则以固定间隔顺序执行。默认情况下，这是通过全局evaluate_interval来控制的，但你可以使用interval子句在规则组中覆盖。

规则组内规则执行的顺序性质意味着你可以在后续规则中使用之前创建的规则。这允许你根据规则创建指标，然后在之后的规则中重用这些指标。这仅在规则组内适用，规则组是并行运行的，因此不建议跨组使用规则。

>NOTE:这意味着你可以将记录规则用作参数，例如，你可能希望创建一个带有阈值的规则。然后，你可以在规则中设置一次阈值并重复使用多次。如果你需要更改阈值，则只需更改一处即可。

下面的配置示例更新规则组为每10秒运行一次，而不是全局定义的15秒：

```
groups:
- name: node_rules
  interval: 10s
  rules:
```

名为rules的YAML块包含该组的记录规则。每条规则都包含一条记录，告诉Prometheus将新的时间序列命名为什么。你应该仔细命名规则，以便快速识别它们代表的内容。一般推荐的格式是：

```
level:metric:operations
```

- level表示聚合级别以及规则输出的标签
- metric是指标名称，除使用rate()或irate()函数剥离_total计数器之外，应该保持不变。
- operations是应用于指标的操作列表，一般最新的操作放在前面。

>NOTE:在Prometheus文档中有一些命名的[最佳实践](https://prometheus.io/docs/practice/naming)

然后，指定一个expr字段来保存生成新时间序列的查询。

还可以添加labels块以向新时间序列添加新标签。根据规则创建的时间序列会继承用于创建它们的时间序列的相关标签，但你也可以添加或覆盖标签。例如：

```
groups:
- name: node_rules
  rules:
  - record: instance:node_cpu:avg_rate5m
    expr: 100 - avg (irate(node_cpu_seconds_total{job="node",mode="idle"}[5m])) by (instance) * 100
    labels:
      metric_type: aggregation
```
让我们再创建一些查询，并添加它们。

```
groups:
- name: node_rules
  rules:
  - record: instance:node_cpu:avg_rate5m
    expr: 100 - avg (irate(node_cpu_seconds_total{job="node",mode="idle"}[5m])) by (instance) * 100
  - record: instance:node_cpus:count
    expr: count by (instance)(node_cpu_seconds_total{mode="idle"})
  - record: instance:node_cpu_saturation_load1
    expr: node_load1 > on (instance) 2 * count by (instance)(node_cpu_seconds_total{mode="idle"})
  - record: instance:node_memory_usage:percentage
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
  - record: instance:node_memory_swap_io_bytes:sum_rate
    expr: 1024 * sum by (instance) (
                 (rate(node_vmstat_pgpgin[1m])
                 + rate(node_vmstat_pgpgout[1m]))
          )
  - record: instance:root:node_filesystem_usage:percentage
    expr: (node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"}) / node_filesystem_size_bytes{mountpoint="/"} * 100
```
在需要重启Prometheus服务器或进行SIGHUP以激活新规则。这将为每个规则创建一个新的时间序列。

通过将SIGHUP信号发送到Prometheus进程，可以在运行时重新加载规则文件。重新加载仅在规则文件格式良好时才有效。Prometheus服务器附带一个名为promtool的实用程序，可以帮助检测规则文件。

你可以在Web界面的/rules路径中查看当前服务器上定义的规则。它会提供一些有用的信息，如每个规则的执行时间，可以帮助你调试可能需要优化的规则。

# 5 服务发现

在抓取配置中手动列出了IP地址和端口这种方法在主机较少时还可以，但不适用于规模较大的集群，尤其不适用于使用容器和基于云的实例的动态集群，这些实例经常会出现变化、创建或销毁的情况。

Prometheus通过使用服务发现解决了这个问题：通过自动化的机制来检测、分类和识别新的和变更的目标。服务发现可以通过以下几种机制实现：

- 从配置管理工具生成的文件中接收目标列表
- 查询API（例如Amazon AWS API）以获取目标列表。
- 使用DNS记录以返回目标列表。

## 5.1 静态配置的局限性

根据之前提到的数据抓取的生命周期，当Prometheus开始作业时，第一步就是服务发现，这将生成作业将要抓取的目标和元数据标签列表。

之前的配置中，服务发现机制是在static_configs块中定义的。不难看出，在繁杂的工作中维护一长串主机列表并不是一个可扩展的任务（HUP的Prometheus服务器也不是每次都可以优雅地启动）。尤其对于大多数环境的动态特性，以及被监控主机、应用程序和服务的规模来说，这种局限性更为明显。

因此需要更成熟的服务发现方式，我们将探索以下几种服务发现方案：

- 基于文件的方式
- 基于云的方式
- 基于DNS的方式

## 5.2 基于文件的服务发现

基于文件的发现只比静态配置更先进一小步，但它非常适合配置管理工具。借助基于文件的服务发现，Prometheus会使用文件中指定的目标。这些文件通常由另一个系统生成，例如Puppet、Ansible或Chef等配置管理系统，或者从其他源（如CMDB）查询。定期执行脚本或进行查询可以（重新）生成这些文件。Prometheus会按指定的时间计划从这些文件重新加载目标。

这些文件可以是YAML或JSON格式，包含定义的目标列表，就像我们在静态配置中定义它们一样。让我们从将现有作业迁移到基于文件的服务发现开始。

基于文件的服务发现配置示例如下：

```yaml
- job_name: node
  file_sd_configs:
    - files:
      - targets/nodes/*.json
      refresh_interval: 5m

- job_name: docker
  file_sd_configs:
    - files:
      - targets/docker/*.json
      refresh_interval: 5m
```

用file_sd_configs块替换prometheus.yml文件中的static_configs块。在这些块中，已经指定了文件列表，并包含在files列表中。我们在父目录targets下为每个作业指定了对应的文件，并为每个作业创建了一个子目录。你可以创建适合你的任何文件结构。

每次作业运行或这些文件发生变化时，Prometheus都会重新加载文件的内容。以防万一，我们还指定了refresh_interval选项，该选项将在每个间隔结束时加载文件列表中的目标——对这个示例来说是5分钟。

还有一个名为prometheus_sd_file_mtime_seconds的指标将告诉你文件的上次更新时间。你可以监控这个指标以识别数据过期问题。

创建目标文件目录以及json文件

```bash
$ cd /etc/prometheus
$ mkdir -p targets/{node,docker}
$ touch targets/nodes/nodes.json
$ touch targets/docker/daemons.json
$ cat targets/nodes/nodes.json
[{
  "targets": [
    "138.197.26.39:9100",
    "138.197.30.147:9100",
    "138.197.30.163:9100"
  ]
}]
$ cat targets/docker/daemons.json
[{
  "targets": [
    "138.197.26.39:8080",
    "138.197.30.147:8080",
    "138.197.30.163:8080"
  ]
}]
```
也可以为这些目标添加标签,如：


[{
  "targets": [
    "138.197.26.39:8080",
    "138.197.30.147:8080",
    "138.197.30.163:8080"
  ],
  "labels": {
      "datacenter": "nj"
  }
}]

向Docker守护进程目标添加了值为nj的标签datacenter。基于文件的服务发现会在重新标记阶段自动给每个目标添加一个元数据标签__meta_filepath，它包含配置目标的文件路径和文件名。

你可以访问https://localhost:9090/service-discovery，通过Web界面查看服务发现目标的完整列表及其元数据标签。

## 5.3 基于API的服务发现

原生的服务发现集成在某些工具和平台上提供，它们内置支持Prometheus。这些服务发现插件使用工具和平台现有的数据存储或API来返回目标列表。

监控Kubernetes上运行的应用程序会用到Kubernetes的服务发现kubernetes_sd_configs

## 5.4 基于DNS的服务发现

如果基于文件的服务发现不适合你，或者你的源或服务不支持任何现有的服务发现工具，则可以选择基于DNS的服务发现。DNS服务发现允许你指定DNS条目列表，然后查询这些条目中的记录以发现目标列表。它依赖于A、AAAA或SRV DNS记录查询。

>NOTE:DNS记录将由Prometheus服务器上本地定义的DNS服务器解析。例如，Linux上的/etc/resolv.conf。

基于DNS服务发现作业配置示例如下：

```yaml
- job_name: webapp
  dns_sd_configs:
    - names: ['_prometheus._tcp.example.com']
```
当Prometheus查询目标时，它会通过DNS服务器查找example.com域。然后，它将在该域中搜索名为_prometheus._tcp.example.com的SRV记录，并返回该条目中的服务记录。

还可以使用DNS服务发现来查询单个A或AAAA记录。为此，我们需要为抓取明确指定查询类型和端口。之所以需要指定端口，是因为A和AAAA记录只返回主机，而不是像SRV记录那样返回主机和端口组合。


# 6 警报管理

## 6.1 警报

一个好警报的关键是能够在正确的时间、以正确的理由和正确的速度发送，并在其中放入有用的信息。
警报方法中最常见的反模式是发送过多的警报。对于监控来说，过多的警报相当于“狼来了”这样的故事。收件人会对警告变得麻木，然后将其拒之门外，而重要的警报常常被淹没在无关紧要的更新中。

通常发送过多警报的原因可能包括：

- 警报缺少可操作性，它只是提供信息。你应关闭所有这些警报，或将其转换为计算速率的计数器，而不是发出警报。
- 故障的主机或服务上游会触发其下游的所有内容的警报。你应确保警报系统识别并抑制这些重复的相邻警报。
- 对原因而不是症状（symptom）进行警报。

症状是应用程序停止工作的迹象，它们可能是由许多原因导致的各种问题的表现。API或网站的高延迟是一种症状，这种症状可能由许多问题导致：高数据库使用率、内存问题、磁盘性能等。对症状发送警报可以识别真正的问题。仅对原因（例如高数据库使用率）发出警报也可能识别出问题（但通常很可能不会）。对于这个应用程序，高数据库使用率可能是完全正常的，并且可能不会对最终用户或应用程序造成性能问题。作为一个内部状态，发送警报是没有意义的。这种警报可能会导致工程师错过更重要的问题，因为他们已经对大量不可操作且基于原因的警报变得麻木。你应该关注基于症状的警报，并依赖你的指标或其他诊断数据来确定原因。

第二种最常见的反模式是警报的错误分类。有时，这也意味着重要的警报会隐藏在其他警报中。但有时候，警报会被发送到错误的地方或者带有不正确的紧急情况说明。

第三种反模式是发送无用的警报.例如：告警信息：磁盘/data目录剩余空间9%，这个通知似乎提供了一些有用的信息，但事实并非如此。这是由存储空间突增导致的？还是逐渐增长的结果？增长速度是多少？正如我们在第1章提到的，1 GB分区上9%的可用磁盘空间与1 TB磁盘上9%的可用磁盘空间完全不同。我们可以忽略或静音这类通知吗？还是需要立即采取行动？

良好的警报应该具备以下几个关键特征：

1. 适当数量的警报，关注症状而不是原因。噪声警报会导致警报疲劳，最终警报会被忽略。修复警报不足比修复过度警报更容易。
2. 应设置正确的警报优先级。如果警报是紧急的，那么它应该快速路由到负责响应的一方。如果警报不紧急，那么我们应该以适当的速度发送警报，以便在需要时做出响应。
3. 警报应包括适当的上下文，以便它们立即可以使用。

>NOTE:Google SRE手册中有一个很棒的关于警报的章节。

## 6.2 Alertmanager如何工作

Alertmanager处理从客户端发来的警报，客户端通常是Prometheus服务器，还可以接收来自其他工具的警报。Alertmanager对警报进行去重、分组，然后路由到不同的接收器，如电子邮件、短信或SaaS服务（PagerDuty等）。你还可以使用Alertmanager管理维护。

在Prometheus服务器上编写警报规则，这些规则将使用我们收集的指标并在指定的阈值或标准上触发警报。当指标达到阈值或标准时，会生成一个警报并将其推送到Alertmanager。警报在Alertmanager上的HTTP端点上接收。一个或多个Prometheus服务器可以将警报定向到单个Alertmanager，或者你可以创建一个高可用的Alertmanager集群。

收到警报后，Alertmanager会处理警报并根据其标签进行路由。一旦路径确定，它们将由Alertmanager发送到外部目的地，如电子邮件、短信或聊天工具。

## 6.3 Alertmanager安装

Alertmanager是一个独立的Go二进制文件。

### 6.3.1 在Linux上安装Alertmanager

```bash
$ cd /tmp
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.15.2/alertmanager-0.15.2.linux-amd64.tar.gz
$ tar -zxf alertmanager-0.15.2.linux-amd64.tar.gz
$ sudo cp alertmanager-0.15.2.linux-amd64/alertmanager /usr/local/bin
$ sudo cp alertmanager-0.15.2.linux-amd64/amtool /usr/local/bin
```

amtool二进制文件用于帮助管理Alertmanager，并可以从命令行安排维护窗口。

## 6.4 配置Alertmanager

与Prometheus一样，Alertmanager配置也是基于YAML的配置文件.下面的配置示例通过电子邮件发送收到的警报。

```bash
$ sudo mkdir -p /etc/alertmanager/
$ sudo touch /etc/alertmanager/alertmanager.yml
```
在alertmanager.yml文件中写入以下内容：
```yaml
global:
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.com'
  smtp_require_tls: false

templates:
- '/etc/alertmanager/template/*.tmpl'

route:
  receiver: email

receivers:
- name: 'email'
  email_configs:
  - to: 'alerts@example.com'
```

此配置文件包含一个基本设置，用于处理警报并通过电子邮件将其发送到一个地址。

第一个块global包含Alertmanager的全局配置。这些选项为所有其他块设置默认值，并在这些块中作为覆盖生效。在示例中，我们只是配置一些电子邮件和SMTP设置：发送电子邮件的邮件服务器、发送电子邮件的地址，并且我们禁用自动使用TLS的要求。上述配置假设SMTP服务器在localhost端口25上运行。

template块包含保存警报模板的目录列表。由于Alertmanager可以发送到各种目的地，因此你通常需要能够自定义警报的外观及其包含的数据。

route块告诉Alertmanager如何处理特定的传入警报。警报根据规则进行匹配然后采取相应的操作。你可以把路由想象成有树枝的树，每个警报都从树的根（基本路由或基本节点）进入。除了基本节点之外，每个路由都有匹配的标准，这些标准应该匹配所有警报。然后，你可以定义子路由或子节点，它们是树的分支，对某些特定的警报感兴趣，或者会采取某些特定的操作。例如，来自特定集群的所有警报可能由特定的子路由处理。在当前的配置中，我们只定义了基本路由，即树的根节点。在本章后面，我们将利用路由来确保警报具有正确的容量、频率和目的地。这里只定义了一个参数receiver。这是我们警报的默认目的地，在示例中是email电子邮件。接下来我们将定义该接收器。

receivers指定警报目的地。你可以通过电子邮件发送警报，或者发送到PagerDuty和VictorOps等服务，以及Slack和HipChat等聊天工具。这里我们只定义了一个目的地：一个电子邮件地址。每个接收器都有一个名称和相关配置。这里我们已经命名了接收器email。对于电子邮件警报，我们使用email_configs块来指定电子邮件选项，例如接收警报的地址。我们还可以指定SMTP设置（这将覆盖全局设置），并添加其他条目（例如邮件标头）。

>NOTE:有一个称为Webhook接收器的内置接收器，你可以使用它将警报发送到Alertmanager中没有特定接收器的其他目的地。

## 6.5 允许Alertmanager

Alertmanager作为一个Web服务运行，默认端口为9093。

```bash
$ alertmanager --config.file alertmanager.yml
```

你可以使用https://localhost:9093的web界面来查看当前警报并管理维护窗口，以及警报抑制（在Prometheus术语中称为silence）。

>NOTE:随Alertmanager一起附带的还有一个命令行工具amtool，允许你查询警报、管理silence和使用Alertmanager服务器等。

接下来配置Prometheus以找到Alertmanager。

## 6.6 为Prometheus配置Alertmanager

在prometheus.yml配置文件中使用了默认的Alertmanager配置，它包含在alerting块中。默认配置如下：
```yaml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager: 9093
```

alerting块包含允许Prometheus识别一个或多个Alertmanager的配置。为此，Prometheus使用与查找抓取目标时相同的发现机制，在默认配置中是static_configs。与监控作业一样，它指定目标列表，此处是主机名alertmanager加端口9093（Alertmanager默认端口）的形式。该列表假定你的Prometheus服务器可以解析alertmanager主机名为IP地址，并且Alertmanager在该主机的端口9093上运行。

### 6.6.1 Alertmanager服务发现

由于我们可以使用服务发现机制，因此也可以使用这些机制来识别一个或多个Alertmanager。让我们添加一条DNS SRV记录，让Prometheus发现Alertmanager。创建Alertmanager SRV记录：

```
_alertmanager._tcp.example.com. 300 IN SRV 10 1 9093
alertmanager1.example.com.
```

在这里，我们以SRV记录的形式指定了一个名为_alertmanager的TCP服务。我们的记录返回主机名alertmanager1.example.com和端口号9093，Prometheus将在此处找到正在运行的Alertmanager。让我们配置Prometheus服务器，以便在此进行搜索。

```yaml
alerting:
  alertmanagers:
  - dns_sd_configs:
    - names: ['_alertmanager._tcp.example.com']
```

Prometheus将查询_alertmanager._tcp.example.com SRV记录以获得Alertmanager的主机名。此外，我们也可以使用其他服务发现机制，让Prometheus识别Alertmanager。

### 6.6.2 监控ALertmanager

与Prometheus一样，Alertmanager暴露了自身的相关指标。让我们创建一个Prometheus作业来监控Alertmanager。

```yaml
- job_name: 'alertmanager'
  static_configs:
    - targets: ['localhost:9093']
```

这将从http://localhost:9093/metrics收集指标并抓取一系列以alertmanager_为前缀的时间序列数据。这些数据包括按状态分类的警报计数、按接收器分类的成功和失败通知的计数（例如，所有向email接收器发送失败的通知）。它还包含Alertmanager集群状态指标.

## 6.7 添加告警规则

与记录规则一样，警报规则在Prometheus服务器配置中加载的规则文件内也使用YAML语句定义。让我们在rules目录中创建一个新文件node_alerts.yml，以保存节点警报规则。

>NOTE:可以在同一文件中同时保存记录规则和警报规则，但为了功能清晰明确，建议将它们放在单独的文件中。

创建报警规则文件：

```bash
$ cd rules
$ touch node_alerts.yml
```
不需要单独将此文件添加到prometheus.yml配置文件中的rule_files块，可以使用globbing通配符加载该目录中以_rules.yml或_alerts.yml结尾的所有文件。

在rule_files块中添加globbing通配符:

```yaml
rule_files:
  - "rules/*_rules.yml"
  - "rules/*_alerts.yml"
```
需要重新启动Prometheus服务器，以加载这个新的警报规则文件。

### 6.7.1 添加第一条警报规则

创建一个警报，如果我们创建的CPU查询（5分钟内的节点平均CPU使用率）在至少60分钟内超过80%，则会触发警报。

报警规则配置如下：

```yaml
groups:
- name: node_alerts
  rules:
  - alert: HighNodeCPU
    expr: instance:node_cpu:avg_rate5m > 80
    for: 60m
    labels:
      serverity: warning
    annotations:
      summary: High Node CPU for 1 hour
      console: You migth want to check the Node Dashboard at http://grafana.example.com/dashboard/db/node-dashboard
```

与记录规则一样，警报规则也可以组合在一起。我们已经指定了一个组名node_alerts，该组中的规则包含在rules块中。每个规则都有一个名称，在alert子句中指定，这里是HighNodeCPU。在每个警报组中，警报名称都必须是唯一的。

在expr子句中指定触发警报的测试或表达式,测试表达式使用我们在第4章用记录规则创建的instance:node_cpu:avg_rate5m指标。

for子句控制在触发警报之前测试表达式必须为true的时间长度。在示例中，指标instance:node_cpu:avg_rate5m需要在触发警报之前的60分钟内大于80%。这限制了警报误报或是暂时状态的可能性。

最后，使用标签（label）和注解（annotation）来装饰警报。警报规则中时间序列上的所有标签都会转移到警报。labels子句允许我们指定要附加到警报的其他标签，这里我们添加了一个值为warning的severity标签，很快我们就会看到如何使用这个标签。

警报上的标签与警报的名称相结合，构成警报的标识。这与时间序列相同，其中指标名称和标签构成时间序列的标识。

annotations子句允许我们指定展示更多信息的标签，如描述、运行手册的链接或处理警报的说明。我们添加了一个名为summary的标签来描述警报，还添加了一个名为console的注解，它指向了展示节点指标的Grafana仪表板。这是一个用注解提供上下文的很好的例子。

重新启动Prometheus后，你将能够在Prometheus Web界面http://localhost:9090/alerts中看到新的警报。

### 6.7.2 警报触发

Prometheus以一个固定时间间隔来评估所有规则，这个时间由evaluate_interval定义，我们将其设置为15秒。在每个评估周期，Prometheus运行每个警报规则中定义的表达式并更新警报状态。

警报可能有以下三种状态：

- Inactive：警报未激活。
- Pending：警报已满足测试表达式条件，但仍在等待for子句中指定的持续时间。
- Firing：警报已满足测试表达式条件，并且Pending的时间已超过for子句的持续时间。

Pending到Firing的转换可以确保警报更有效，且不会来回浮动。没有for子句的警报会自动从Inactive转换为Firing，只需要一个评估周期即可触发。带有for子句的警报将首先转换为Pending，然后转换为Firing，因此至少需要两个评估周期才能触发。

警报的生命周期如下：

- 节点的CPU不断变化，每隔一段由scrape_interval定义的时间被Prometheus抓取一次，对我们来说是15秒。
- 根据每个evaluation_interval的指标来评估警报规则，对我们来说还是15秒。
- 当警报表达式为true时（对于我们来说是CPU超过80%），会创建一个警报并转换到Pending状态，执行for子句。
- 在下一个评估周期中，如果警报测试表达式仍然为true，则检查for的持续时间。如果超过了持续时间，则警报将转换为Firing，生成通知并将其推送到Alertmanager。
- 如果警报测试表达式不再为true，则Prometheus会将警报规则的状态从Pending更改为Inactive。

### 6.7.3 Alertmanager的警报

我们的警报现在处于Firing状态，并且已将通知推送到Alertmanager。可以在Prometheus Web界面http://localhost:9090/alerts中看到这个警报及其状态。

Prometheus将为Pending和Firing状态中的每个警报创建指标，这个指标被称为ALERT，并且会像HighNodeCPU警报示例那样构建。

```
ALERTS{alertname="HighNodeCPU",alertsstate="firing",serverity=warning,instance="138.197.26.39:9100"}
```

每个alert指标都具有固定值1，并且在警报处于Pending或Firing状态期间存在。在此之后，它将不接收任何更新，并且最终会过期。

通知将被发送到Prometheus配置中定义的Alertmanager，在示例中是alertmanager主机和端口9093，通知被推送到如下HTTP端点：

```
http://alertmanager:9093/api/v1/alerts
```
假设一个HighNodeCPU警报被触发了，我们将能在Alertmanager Web控制台http://alertmanager:9093/#/alerts上看到该警报.
在当前Alertmanager配置中，警报将立即发送到email接收器.如果我们有很多团队，或者不同严重程度的警报，那么上面的内容似乎不太实用。这就是Alertmanager路由可以提供帮助的地方。

### 6.7.4 添加新警报和模板

为了有更多的警报可以路由，让我们快速添加一些其他警报规则到node_alerts.yml警报规则文件中。

```yaml
groups:
- name: node_alerts
  rules:
.....
  - alert: DiskWillFillIn4Hours
    expr: predict_linear(node_filesystem_free_bytes{mountpoint="/"}[1h], 4 * 3600) < 0
    for: 5m
    labels:
      serverity: critical
    annotations:
      summary: Disk on {{ $labels.instance }} will fill in approxymately 4 hourts.
  - alert: InstanceDown
    expr: up{job="node"} == 0
    for: 10m
    labels:
      serverity: critical
    annotations:
      summary: Host {{ $labels.instance }} of {{ $labels.job }} is down!
```
第一个警报复制了我们在第4章看到的predict_linear磁盘预测。这里，如果线性回归预测/根文件系统的磁盘空间将在4小时内耗尽，则会触发警报。你可能还会注意到，我们已在summary注解中添加了一些模板值。

#### 模板

模板（template）是一种在警报中使用时间序列数据的标签和值的方法，可用于注解和标签。模板使用标准的Go模板语法，并暴露一些包含时间序列的标签和值的变量。标签以变量$labels形式表示，指标的值则是变量$value。

>NOTE:变量$labels和$value分别是底层Go变量.Labels和.Value的名称。

要在summary注解中引用instance标签，我们使用{{$labels.instance}}。如果想要引用时间序列的值，那么我们会使用{{$value}}。Prometheus还提供了一些其他功能，你可以在[模板参考](https://prometheus.io/docs/prometheus/latest/configuration/template_reference/)中看到。其中一个例子是humanize函数，它使用指标前缀将数字转换为更易于阅读的形式。例如：

```
...
annotations:
  summary: High Node CPU of {{ humanize $value }}% for 1 hour
```
这会将指标值显示为两位小数的百分比，例如88.23%。

#### Prometheus警报

始终牢记：Prometheus服务器也可能出问题。让我们添加一些规则来识别问题并对它们发出警告。我们将在rules目录中创建一个新文件prometheus_alerts.yml以保存它们。因为这符合我们的glob规则，它也会被Prometheus加载。

```bash
$ touch rules/prometheus_alerts.yml
```
文件内容如下：

```yaml
groups:
- name: prometheus_alerts
  rules:
  - alert: PrometheusConfigReloadFailed
    expr: prometheus_config_last_reload_successful == 0
    for: 10m
    labels:
      severity: warning
    annotations:
      description: Reloading Prometheus configuration has failed on {{ $labels.instance }}
  - alert: PrometheusNotConnectedToAlertmanagers
    expr: prometheus_notifications_alertmanagers_discovered  < 1
    for: 10m
    labels:
      severity: warning
    annotations:
      description: Prometheus {{ $labels.instance }} is not connected to any Alertmanagers
```

上面添加了两个新规则。第一个是PrometheusConfigReloadFailed，它让我们知道Prometheus配置重新加载是否失败。如果上次重新加载失败，则使用指标prometheus_config_last_reload_successful，且指标的值为0。

第二条规则确保Prometheus服务器可以发现Alertmanager。这使用prometheus_notifications_alertmanagers_discovered指标，该指标是服务器找到的Alertmanager计数。如果小于1，则表面Prometheus没有发现任何Alertmanager，并且这个警报将会触发。由于没有任何Alertmanager，因此它只会显示在Prometheus控制台的/alerts页面上。

#### 可用性警报

如果我们在节点上监控的服务不再活动，则会生成一个警报。节点服务警报配置如下：

```yaml
- alert: NodeServiceDown
  expr: node_systemd_unit_state{state="active"} != 1
  for: 60s
  labels:
    serverity: critical
  annotations:
    summary: Service {{ $labels.name }} on {{ $labels.instance }} is no longer active!
    description: Werner Heisenberg says - "OMG Where's my service?"
```
如果带有active标签的node_systemd_unit_state指标值为0，则会触发此警报，表示服务故障至少60秒。

下一个警报使用up指标，该指标对于监控主机的可用性非常有用。但是它并不完美，因为我们真正监控的是作业对目标的抓取是否成功。然而，知道实例是否已停止响应抓取也是很有用的，这可能表明存在更严重的问题。为此，警报会检测up指标的值是否为0，如果是0则表示抓取失败。

```
up{job="node"} == 0
```
在许多情况下，知道单个实例宕机实际上并不是非常重要。相反，我们也可以测试多个失败的实例，例如，失败实例的百分比：

```
avg(up) by (job) <= 0.50
```

这个测试表达式计算出up指标的平均值然后按job聚合，并在该值低于50%时触发。如果作业中50%的实例无法完成抓取，则会触发警报。另一种方法可能是：

```
sum by job (up) / count(up) <= 0.8
```

这里，我们根据job对up指标求和，然后将其除以计数，如果结果大于或等于0.8，或者特定作业中20%的实例未启动，则触发警报。

通过确定目标何时消失，我们可以使up警报稍微健壮一些。例如，如果从服务发现中删除我们的目标，那么它的指标将不再更新。如果所有目标都从服务发现中消失，则不会记录任何指标，因此up警报不会被触发。Prometheus有一个功能叫absent，可检测是否存在缺失的指标。

```yaml
- alert: InstancesGone
  expr: absent(up{job="node"})
  for: 10s
  labels:
    severity: critical
  annotations:
    summary: Host {{ $labels.instance }} is no longer reporting!
    description: Werner Heisenberg says - "OMG Where's my service?"
```

在这里，我们的表达式使用absent函数来检测节点作业中的up指标是否消失，如果消失，那么它将触发警报。

>NOTE:另一种可用性监控方法是通过HTTP、HTTPS、DNS、TCP和ICMP来探测端点，我们将在第10章看到更多内容。

## 6.8 路由

现在我们已经选择了一些具有不同属性的警报，需要将它们路由到不同的目的地。我们之前发现路由是一棵树。顶部的默认路由总会被配置，并匹配任何子路由不匹配的内容。回到Alertmanager配置，让我们在alertmanager.yml文件中添加一些路由配置。

```yaml
route:
  group_by: ['instance']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: email
  routes:
  - match:
      severity: critical
    receiver: pager
    routes:
      - match:
          service: application1
        receiver: support_team
  - match_re:
      severity: ^(informational|warning)$
    receiver: support_team
receivers:
- name: 'email'
  email_configs:
  - to: 'alerts@example.com'
- name: 'support_team'
  email_configs:
  - to: 'support@example.com'
- name: 'pager'
  email_configs:
  - to: 'alert-pager@example.com'
```

可以看到，我们为默认路由添加了一些新选项。第一个选项group_by控制的是Alertmanager分组警报的方式。默认情况下，所有警报都组合在一起，但如果我们指定了group_by和任何标签，则Alertmanager将按这些标签对警报进行分组。例如，我们已经指定了instance标签，这意味着来自特定实例的所有警报将被分在一起。如果列出多个标签，则会在每个指定的标签值匹配时对警报进行分组，例如：
```yaml
route:
  group_by: ['service','cluster']
```
这里，service和cluster标签的值需要与要分组的警报匹配。

>NOTE:这仅适用于标签，不适用于注解。

分组还会更改Alertmanager的行为。如果引发了新警报，那么Alertmanager将等待下一个选项group_wait中指定的时间段，以便在触发警报之前查看是否收到该组中的其他警报。你可以将其视为警报缓冲。在我们的例子中，这个等待时间是30秒。

在发出警报后，如果收到来自该分组的下一次评估的新警报，那么Alertmanager将等待group_interval选项中指定的时间段（即5分钟），然后再发送新警报。这可以防止警报分组的警报泛滥。

我们还指定了repeat_interval。这个暂停并不适用于我们的警报组，而是适用于单个警报，并且是等待重新发送相同警报的时间段，我们指定为3个小时。

#### 路由表

第一条路由使用了我们定义的新接收器pager，这会将此路由上的警报发送到新的电子邮件地址。它使用match选项查找要发送的特定警报。这里有两种匹配方法：标签匹配和正则表达式匹配。match选项执行简单的标签匹配。标签匹配配置如下：
```yaml
match:
  severity: critical
```
在这里，我们将所有severity标签与critical值匹配，并将它们发送到pager接收器。由于路由都是分支，因此，如果需要我们也可以再次分支路由。路由分支配置如下：

```yaml
routes:
- match:
    severity: critical
  receiver: pager
  routes:
    - match:
        service: application1
    receiver: support_team
```
可以看到我们的新routes块嵌套在已有的route块中。要触发此路由，我们的警报首先需要severity标签为critical，并且service标签是application1。如果这两个条件都匹配，那么我们的警报将被路由到接收器support_team。

我们也可以根据需要尽可能地嵌套路由。默认情况下，与该路由匹配的任何警报都由该路由处理。但是，我们可以使用continue选项来覆盖此行为，该选项控制警报是否先遍历路由，然后再返回以遍历路由树。

>NOTE:Alertmanager路由是后序遍历的。

```yaml
routes:
- match:
    serverity: critical
  receiver: pager
  continue: true
```

continue选项默认为false，但如果设置为true，则警报将在此路由中触发（如果匹配），并继续执行下一个相邻路由。有时这对于向两个地方发送警报很有用，但更好的解决方法是在接收器中指定多个端点，接收器中的多个端点的配置如下：

```yaml
receivers:
- name: 'email'
  email_configs:
  - to: 'alerts@examples.com'
  pagerduty_configs:
  - service_key: TEAMKEYHERE
```

这会添加第二个pagerduty_configs块，该块既可以将警报发送到PagerDuty，也可以发送电子邮件。我们可以指定任何可用的接收器，例如，可以向Slack等聊天服务发送电子邮件和消息。

>NOTE:你是否习惯查看已解决的警报？这些是警报条件解决后生成的警报。通过在接收器配置中将send_resolved选项设置为true，可以使用Alertmanager发送它们。通常不建议发送这些已解决的警报，因为其可能导致“错误警报”的循环，进而导致警报疲劳，所以在启用之前要仔细考虑。

第二个路由使用match_re选项将正则表达式与标签匹配，正则表达式使用severity标签。

```yaml
- match_re:
    severity: ^(informational|warning)$
  receiver: support_team
```
它匹配severity标签中的informational或warning值。

## 6.9 接收器和通知模板

现在让我们添加一个非电子邮件的接收器。我们将添加Slack接收器，它会消息发送到Slack实例。在alertmanager.yml配置文件中查看新接收器配置。首先，我们将为pager接收器添加一个Slack配置。

```yaml
receivers:
- name: 'pager'
  email_configs:
  - to: 'alert-pager@example.com'
  slack_configs:
  - api_url: https://hooks.slack.com/services/ABC123/ABC123/EXAMPLE
    channel: #monitoring
```

现在，任何向pager接收器发送警报的路由都将被发送到Slack的#monitoring频道，并通过电子邮件发送到alert-pager@example.com。

Alertmanager发送给Slack的一般警报消息非常简单。你可以在其[源代码](https://github.com/prometheus/alertmanager/blob/master/template/default.tmpl)中看到Alertmanager使用的默认模板，该模板包含电子邮件和其他接收器的默认值，但是我们可以为许多接收器覆盖这些值。例如，可以在Slack警报中添加文本行。

```yaml
  slack_configs:
  - api_url: https://hooks.slack.com/services/ABC123/ABC123/EXAMPLE
    channel: #monitoring
    text: '{{ .CommonAnnotations.summary }}'
```
Alertmanager自定义通知使用Go模板语法。警报中包含的数据也通过变量暴露。我们使用CommonAnnotations变量，该变量包含一组警报通用的注解集。我们使用summary注解作为Slack通知的文本。

>NOTE:可以在Alertmanager[文档](https://prometheus.io/docs/alerting/notifications)中找到通知模板变量的完整参考。

还可以使用Go template函数来引用外部模板，从而避免在配置文件中嵌入较长且复杂的字符串。我们在本章前面引用了模板目录，目录位于/etc/alertmanager/templates/。让我们在这个目录中创建一个模板。

```bash
$ touch /etc/alertmanager/templates/slack.tmpl
```

slack.tmpl的文件内容如下：

```
{ define "slack.example.text" {}}{{ .CommonAnnotations.summary }}{{ end}}
```

这里使用define函数定义了一个新模板，以end结尾，并取名为slack.example.text，然后在模板内的text中复制内容。现在可以在Alertmanager配置中引用该模板。

```yaml
  slack_configs:
  - api_url: https://hooks.slack.com/services/ABC123/ABC123/EXAMPLE
    channel: #monitoring
    text: '{{ template "slack.example.text" . }}'
```

我们使用了template选项来指定模板的名称。现在将使用模板通知来填充text字段，这对于使用上下文装饰通知很有用。Alertmanager文档中还有一些其他的通知模板示例.

## 6.10 silence和维护

通常我们需要让警报系统知道我们已经停止服务以进行维护，并且不希望触发警报。或者，当上游出现问题时，我们需要将下游服务和应用程序“静音”。Prometheus称这种警报静音为silence。silence可以设定为特定时期，例如一小时，或者是一个时间窗口（如直到今天午夜）。这是silence的到期时间或到期日期。如果需要，我们也可以提前手动让silence过期（如果我们的维护比计划提前完成）。

可以使用以下两种方法来设置silence：

- 通过Alertmanager Web控制台；
- 通过amtool命令行工具。

### 6.10.1 通过Alertmanager控制silence

第一种方法是使用Web界面，单击New Silence按钮如图6-7所示。silence指定开始时间、结束时间或持续时间。通过使用标签匹配警报来识别要静音的警报，就像警报路由一样。你可以使用直接匹配，例如匹配具有特定值的标签的每个警报，或者可以使用正则表达式匹配。你还需要指定silence的创建者和注释，以说明警报被静音的原因。

### 6.10.2 通过amtool控制silence

第二种方法是使用amtool命令行。amtool二进制文件随Alertmanager安装tar包附带。

```bash
$ amtool --alertmanager.url=http://localhost:9093 silence add alertname=InstancesGone service=application1
```
这将在Alertmanager的http://localhost:9093上添加一个新silence，它将警报与两个标签匹配：自动填充包含警报名称的alertname标签；以及我们设置的service标签。

>NOTE:使用amtool创建的silence被设置为一小时后自动过期，可以使用--expires和--expire-on参数来指定更长的时间或窗口。

这将返回一个silence ID，你可以使用该ID，以在稍后处理silence。示例中ID为：

```bash
784ac68d-33ce-4e9b-8b95-431a1e0fc268
```

使用query子命令来查询当前silence列表:

```bash
$ amtool --alertmanager.url=http://localhost:9093 silence query
```
这将返回silence列表及其配置，你可以通过ID来使特定的silence过期:

```bash
$ amtool --alertmanager.url=http://localhost:9093 silence expire 784ac68d-33ce-4e9b-8b95-431a1e0fc268
```
这将使Alertmanager上的silence失效。

你可以为某些选项创建一个YAML配置文件，而不必每次都指定--alertmanager.url参数。amtool查找的默认配置文件路径是$HOME/.config/amtool/config.yml或/etc/amtool/config.yml。我们来看一个示例文件。

```
alertmanager.url: "http://localhost:9093"
author: sre@example.com
comment_required: true
```
可以看到我们已经添加了一个Alertmanager，同时还指定了author属性，这是silence的创建者，它的默认值为本地用户名，你可以使用这样的方式来编辑author，或者在命令行上使用-a或--author参数。comment_required参数控制silence是否需要注释来说明它的作用。

你可以在配置文件中指定所有amtool参数，但有些参数可能没有太大意义.

回到silence的创建，在创建silence时，你还可以使用正则表达式作为标签值。

```bash
$ amtool silence add --comment "App1 maintenane" alertname=~'Instance.*' service=application1
```

这里我们使用=~来表示标签匹配是一个正则表达式，并在所有警报上匹配一个以Instance开头的alertname。我们还使用了--comment参数来添加有关警报的信息。

还可以控制silence的更多细节，如下所示：

```bash
$ amtool silence add - author "James" --duration "2h" alertname=InstancesGone service=application1
```
这里，我们用--author参数覆盖了silence的创建者，并将持续时间指定为两小时，而不是默认的一小时。

amtool还允许我们使用Alertmanager并验证其配置文件，以及其他有用的任务。你可以通过使用--help参数查看命令行参数的完整列表，还可以通过amtoolsilence --help获得特定子命令的帮助。此外，你可以使用Github上的[说明](https://github.com/prometheus/alertmanager/blob/master/cmd/amtool/README.md)完成命令行自动补全和帮助页生成。

# 7 可靠性和可扩展性

到目前为止，Prometheus都是以单个服务器运行，同时也仅包含一个Alertmanager。这种方式适用于许多监控场景，尤其是在团队监控自己的资源时，但它通常无法扩展到多个团队。此外，这种方式也不够有弹性、不够健壮。如果Prometheus服务器或Alertmanager负载过高，或者出现故障，那么我们的监控或警报也将失效。

把以上内容分为两个问题进行考虑：

- 可靠性和容错性
- 可扩展性

针对每个问题，Prometheus有不同的解决方案，我们也会看到如何通过架构优化来同时解决这两个问题。在本章中，我们将讨论Prometheus处理每个问题的理念和方法，并了解如何构建更具可扩展性和可靠性的Prometheus实.

## 7.1 可靠性和容错性

Prometheus解决容错问题的方法考虑到了操作和技术的复杂性。在多数情况下，监控服务的容错性可以通过使监控服务高可用来解决，通常的实现方式是构建集群。但是，集群解决方案需要相对复杂的网络，并且需要解决集群中节点之间的状态管理问题。

Prometheus专注于实时监控，通常会保留一段时间内的数据，并且我们假设配置是由配置管理工具管理的。从可用性的角度来看，单个Prometheus服务器通常被认为是不可靠的。Prometheus架构认为，实现集群所需的投入以及维护集群节点之间数据一致性的成本要高于数据本身的价值。

但是Prometheus并没有忽视解决容错问题的必要性。实际上，Prometheus推荐的容错解决方案是并行运行两个配置相同的Prometheus服务器，并且这两个服务器同时处于活动状态。该配置生成的重复警报可以交由上游Alertmanager使用其分组（及抑制）功能进行处理。一个推荐的方法是尽可能使上游Alertmanager高度容错，而不是关注Prometheus服务器的容错能力.

### 7.1.1 重复的Prometheus服务器

这里不会介绍构建两个重复的Prometheus服务器的细节，使用配置管理工具可以相对容易地实现这一点。建议按照第3章中的安装步骤，选用一个配置管理解决方案。

### 7.1.2 设置Alertmanager集群

Alertmanager包含由HashiCorp Memberlist库提供的集群功能。Memberlist是一个Go语言库，使用基于gossip的协议来管理集群成员和成员故障检测，其也是SWIM协议的扩展。

要配置集群首先需要在多个主机上安装Alertmanager。在示例中，在3台主机（am1、am2和am3）上运行它。在每个主机上安装Alertmanager。使用am1主机来启动集群。在am1上：

```bash
am1$ alertmanager --config.file alertmanager.yml --cluster.listen-address 172.19.0.10:8001
```

运行alertmanager二进制文件来指定一个配置文件（可以使用在第6章中创建的文件），以及一个集群监听地址和端口。你需要在集群中的每个节点上使用相同的配置，这样可以确保对警报的处理是相同的，并且确保集群的一致性。

>WARNING:所有Alertmanager应使用相同的配置！如果不相同，那么集群实际上并不是高可用的。

我们指定了am1主机的IP地址172.19.0.10和8001端口。Alertmanager集群中的其他节点将使用这个地址和端口连接到集群，因此该端口需要在Alertmanager集群节点之间的网络上保持可访问状态。

>NOTE:如果未指定集群监听地址，则默认为0.0.0.0的9094端口。

然后在其他两台主机上运行Alertmanager，监听它们的本地IP地址，并引用刚刚创建的集群节点的IP地址和端口。

```bash
am2$ alertmanager --config.file alertmanager.yml --cluster.listen-address 172.19.0.20:8001 --cluster.peer 172.19.0.10:8001
am3$ alertmanager --config.file alertmanager.yml --cluster.listen-address 172.19.0.30:8001 --cluster.peer 172.19.0.10:8001
```
在另外两个Alertmanger主机（am2和am3）上同样运行alertmanager二进制文件，并使用各自的IP地址和8001端口来为每台主机指定一个集群监听地址。我们还使用集群cluster.peer参数指定am1节点的IP地址和端口作为peer，以便它们可以加入集群。

你将不会看到任何指示集群已启动的特定消息（即便使用--debug参数可获得更多输出），但你可以在Alertmanager的控制台状态页面/status上进行确认。我们来看看主机am1上显示的集群状态https://172.19.0.10:9093/status.

在am1上可以看到Alertmanager集群中的三个节点：am1本身，以及am2和am3。

你可以在一个Alertmanager上设置silence并查看配置是否复制到其他Alertmanager节点，以此来测试集群是否正常工作。为此，请单击am1上的NewSilence按钮并设置silence，然后检查am2和am3上的/silences路径，应该可以看到所有主机上都复制了相同的silence配置。

现在集群正常运行，接下来我们需要告诉Prometheus所有的Alertmanager信息。

### 7.1.3 为Prometheus配置Alertmanager集群

考虑到弹性，我们必须为Prometheus服务器标识所有的Alertmanager。这样，如果集群中的某个Alertmanager发生故障，Prometheus便可以找到另一个来发送警报。Alertmanager集群本身负责与集群的其他活动成员共享所有收到的警报，并处理数据去重（如果需要的话）。因此，你不应该为Alertmanager设置负载平衡，因为Prometheus会帮你处理。

使用静态配置来定义所有的Alertmanager，如下所示：

```yaml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - am1:9093
      - am2:9093
      - am3:9093
```

通过上述配置，Prometheus服务器将连接到所有3个Alertmanager，前提是Prometheus服务器可以解析每个Alertmanager的DNS条目。另一个更加智能的方法是使用服务发现来查找所有Alertmanager。例如，像我们在第6章中看到的那样使用基于DNS的服务发现，我们可以为每个Alertmanager添加DNS SRV记录。

```
_alertmanager._tcp.example.com. 300 IN SRV 10 1 9093 am1.example.com.
_alertmanager._tcp.example.com. 300 IN SRV 10 1 9093 am2.example.com.
_alertmanager._tcp.example.com. 300 IN SRV 10 1 9093 am3.example.com.
```

在这里，我们以SRV记录的形式指定了名为_alertmanager的TCP服务。我们的记录返回三个主机名am1、am2和am3，以及端口号9093（Prometheus可以在这里找到一个正在运行的Alertmanager）。让我们配置Prometheus服务器来发现它们。

```yaml
alerting:
  alertmanagers:
  - dns_sd_configs:
    - names: ['_alertmanager._tcp.example.com']
```
在这里，Prometheus将查询alertmanager.example.com SRV记录以返回Alertmanager列表。我们可以使用其他服务发现机制，让Prometheus识别集群中的所有Alertmanager。

现在，当有警报时，它将被发送到所有已发现的Alertmanager。Alertmanager接收警报、处理数据去重并共享状态。这为上游提供了容错功能，可以确保警报被正常传递。

## 7.2 可扩展性

除了容错功能外，我们还将介绍Prometheus的扩展选项。大多数选项基本上都是手动的，涉及选择特定Prometheus服务器上运行的特定工作负载。
Prometheus环境扩展通常有两种形式：功能扩展或水平扩展。

### 7.2.1 功能扩展

功能扩展使用分片将监控内容分布到不同的Prometheus服务器上。例如，可以通过地理位置或者逻辑域来拆分服务器.

或者可以通过特定功能，将所有基础设施监控发送到一台服务器，而将所有应用程序监控发送到另一台服务器.

设置在每台Prometheus服务器上运行的特定作业是一个相对简单的过程，不过最好是使用配置管理工具来确保创建服务器以及在其上运行特定作业是一个自动化过程。

从这里开始，如果需要对某些区域或功能进行整体视图查看，那么你可以使用联合功能（federation，稍后会详细介绍）将时间序列提取到集中的Prometheus服务器。Grafana支持从多个Prometheus服务器提取数据来构建图形,详情参考[这里](http://docs.grafana.org/guides/whats-new-in-v2-5/#mix-different-data-sources)，这可能对你很有帮助，它允许在可视化级别联合来自多个服务器的数据，前提是收集的时间序列具有一定的一致性。

### 7.2.2 水平分片

某些时候，通常是在大规模部署中，垂直分片的容量和复杂性将会出现问题，当单个作业包含数千个实例时尤其如此。这种情况下，你可以考虑另一种方案：水平分片。水平分片使用一系列工作节点（worker），每个节点都抓取一部分目标。然后，我们在工作节点上汇总感兴趣的特定时间序列。例如，若我们正在监控主机指标，则可能会汇总这些指标的子集。然后，主节点（primary）使用Prometheusfederation API来抓取每个工作节点的聚合指标.

主节点不仅可以提取聚合指标，还可以为Grafana等工具暴露指标或者作为可视化的默认数据源。如果需要更深入或进一步扩展，则可以添加分层的主节点和工作节点。一个很好的例子是基于区域的主节点和工作节点，可能是故障域或者像AWS可用区这样的逻辑区域，将基于区域的主节点视为全局的工作节点，然后向全局的主节点进行报告。

需要注意的是，这种扩展方式存在风险和限制，最显而易见的是，你需要从工作节点中抓取一部分指标，而不是大量或正在收集的所有指标。这是一个类似金字塔的层级结构，而不是分布式的层级结构。此外，你还需要考虑主节点对工作节点的抓取请求负载。

接下来，你会在你的环境中创建更复杂的Prometheus服务器层级结构，还需要担心主节点与工作节点之间的连接，而不仅仅是工作节点与目标之间的连接。这可能会降低解决方案的可靠性。

最后，数据的一致性和正确性也可能会降低。工作节点正在根据设定的间隔抓取目标，而你的主节点也要抓取工作节点。这会导致到达主节点的结果出现延迟，并可能导致数据倾斜或警报延迟。

这两个问题的后果是，在主节点上集中警报可能不是一个好主意。相反，应该将警报推送到工作节点上，在那里更有可能识别出问题（例如目标缺失），或者减少识别警报条件和触发警报之间的滞后。

>NOTE:水平分片通常是最后的选择。我们希望在你需要以这种方式扩展之前，每个目标都有数万个目标或大量时间序列。


# 8 监控应用程序

# 9 日志监控

# 10 探针监控

# 11 推送指标和Pushgateway

# 12 监控kubernetes

# 13 监控Tornado
