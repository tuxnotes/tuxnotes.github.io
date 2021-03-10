---
layout: post
title: 使用Go编写应用的prometheus-exporter的15个步骤[译]
date: 2021-03-10
author: tux
tags: prometheus
---

本文转自：https://medium.com/teamzerolabs/15-steps-to-write-an-application-prometheus-exporter-in-go-9746b4520e26。仅用于学习目的，转载请注明

示例代码地址：https://github.com/teamzerolabs/mirth_channel_exporter

# 1 背景

exporter是prometheus监控流水线中的核心和灵魂。如果你想使用的exporter没有，则需要自己编写一个。这里将包含编写exporter需要的几个步骤。

# 2 目标

- 使用Go写一个exporter
- exporter抓取的时候将调用应用(Mirth Connect in this example)的REST API
- exporter将返回的结果转换为metrics

# 3 步骤

## 3.1 搭建Go的环境和包的依赖

- Go环境的搭建参考：https://golang.org/doc/install
- 创建一个名为my_first_exporter的目录

```bash
go mod init my_first_exporter 
go get github.com/prometheus/client_golang 
go get github.com/joho/godotenv
--> creates go.mod file
--> Installs dependency into the go.mod file
```

## 3.2 创建入口文件并导入依赖

创建main.go文件，并写入如下代码：

```go
package main

import (
 "github.com/joho/godotenv"
 "github.com/prometheus/client_golang/prometheus"
 "github.com/prometheus/client_golang/prometheus/promhttp"
)
```
## 3.3 写入入口函数main()

```go
func main() {

}
```
## 3.4 添加prometheus metrics endpoint并监听服务端口

```go
func main() {
    http.Handle("/metrics", promhttp.Handler())
    log.Fatal(http.ListenAndServe(":9101", nil))
}
```

## 3.5 使用curl命令探索外部服务的API

这里要监控的应用是MirthConnect，这里将使用两个API调用：

1. 获取channel的统计信息
2. 获取channel的id和名称映射

```bash
curl -k --location --request GET 'https://apihost/api/channels/statistics' \
--user admin:admin

curl -k --location --request GET 'https://apihost/api/channels/idsAndNames' \
--user admin:admin
```
## 3.6 将curl调用转换为go的HTTP调用，并处理结果解析

本步是整个过程中最难的部分，比如我的端点返回的XML文本，这意味着我需要使用encoding/xml将对内容进行反序列化。

如果转换成功意味着go程序处理结果与curl是一样的。为了完成这个转换需要使用下面的包：

- crypto/tls = specify TLS connection options.指定TLS连接选项
- io/ioutil = reading the result payload from buffer into strings从Buffer中读取结果并转为string类型
- net/http = create transport and clients
- strconv = converting string to numbers like floating point/integer将字符串转换为数值型

你也可以使用log包将结果迭代后的值输出

## 3.7 声明prometheus metric description

在prometheus中，每个metric都由下面几部分组成：
- metric name：指标名称
- metric label value：指标标签的值
- metric help text：指标帮助文本
- metric type:指标类型
- measurement：测量值

例如：

```
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code=”200"} 1.829159e+06
promhttp_metric_handler_requests_total{code=”500"} 0
promhttp_metric_handler_requests_total{code=”503"} 0
```

对于应用的抓取器，我们会定义prometheus metric description，这包括指标名称/指标标签/指标帮助文本

```go
messagesReceived = prometheus.NewDesc(
 prometheus.BuildFQName(namespace, "", "messages_received_total"),
 "How many messages have been received (per channel).",
 []string{"channel"}, nil,
)
```
## 3.8 使用接口声明prometheus exporter的几个部分

自定义exporter需要4个部分：

1. A structure with member variables一个结构体
2. A factory method that returns the structure返回结构体的工厂方法
3. Describe function Describe函数
4. Collect function Collect函数

```go
type Exporter struct {
 mirthEndpoint, mirthUsername, mirthPassword string
}

func NewExporter(mirthEndpoint string, mirthUsername string, mirthPassword string) *Exporter {
 return &Exporter{
  mirthEndpoint: mirthEndpoint,
  mirthUsername: mirthUsername,
  mirthPassword: mirthPassword,
 }
}
func (e *Exporter) Describe(ch chan<- *prometheus.Desc) {
}
func (e *Exporter) Collect(ch chan<- prometheus.Metric) {
}
```

## 3.9 在Describe函数中发送3.7的metric description

```go
func (e *Exporter) Describe(ch chan<- *prometheus.Desc) {
 ch <- up
 ch <- messagesReceived
 ch <- messagesFiltered
 ch <- messagesQueued
 ch <- messagesSent
 ch <- messagesErrored
}
```
## 3.10 将3.6步API的调用逻辑放到collect函数中

这里继续采用3.6步的逻辑。不同于3.6将内容输出到屏幕，而是将内容发送到prometheus.Metric通道：

```go
func (e *Exporter) Collect(ch chan<- prometheus.Metric) {
 channelIdNameMap, err := e.LoadChannelIdNameMap()
 if err != nil {
  ch <- prometheus.MustNewConstMetric(
   up, prometheus.GaugeValue, 0,
  )
  log.Println(err)
  return
 }
 ch <- prometheus.MustNewConstMetric(
  up, prometheus.GaugeValue, 1,
 )

 e.HitMirthRestApisAndUpdateMetrics(channelIdNameMap, ch)
}
```
当调用API的时候，确保使用prometheus.MustNewConstMetric(prometheus.Desc, metric type, measurement)发送到measurements

对于需要传入外部标签的场景，按下面的做法在参数列表的末尾添加外部的标签

```go
channelError, _ := strconv.ParseFloat(channelStatsList.Channels[i].Error, 64)
ch <- prometheus.MustNewConstMetric(
 messagesErrored, prometheus.GaugeValue, channelError, channelName,
)
```
## 3.11 在main函数中声明exporter并注册

```go
exporter := NewExporter(mirthEndpoint, mirthUsername, mirthPassword)
prometheus.MustRegister(exporter)
```

到此exporter就可以使用了，每次请求metrics路由的时候，将调用API，并以prometheus文本文件的格式返回结果。下面的步骤是使得exporter的部署更加容易。

## 3.12 将硬编码的API路径转换为flags的形式

目前，应用的base url，metric route url，exporter端口都是采用硬编码的方式。可以使用命令行参数的方式将程序变得更加灵活

```go
var (
listenAddress = flag.String("web.listen-address", ":9141",
 "Address to listen on for telemetry")
metricsPath = flag.String("web.telemetry-path", "/metrics",
 "Path under which to expose metrics")
)
func main() {
   flag.Parse()
   ...
   http.Handle(*metricsPath, promhttp.Handler())
   log.Fatal(http.ListenAndServe(*listenAddress, nil))
}
```
## 3.13 将加密信息移到环境变量中

如果应用的endpoint或者登录密码等发生变化怎么办？我们可以从环境变量中加载。在本示例中，使用godotenv包来帮助存储变量值：

```go
import (
  "os"
)
func main() {
 err := godotenv.Load()
 if err != nil {
  log.Println("Error loading .env file, assume env variables are set.")
 }
 mirthEndpoint := os.Getenv("MIRTH_ENDPOINT")
 mirthUsername := os.Getenv("MIRTH_USERNAME")
 mirthPassword := os.Getenv("MIRTH_PASSWORD")
}
```
## 3.14 编写Makefile用于不同平台的快速构建

Makefile可以让你在开发过程中节省大量的输入。对于需要在多平台上构建的exporter(testing on windows/mac, running in Linux),可以使用下面的内容：

```
linux:
   GOOS=linux GOARCH=amd64 go build
mac:
   GOOS=darwin GOARCH=amd64 go build
```
因此简单的make mac或make linux就会生成不同的执行文件

## 3.15 编写service文件让程序在后台运行

取决于exporter的运行环境，你可以写个service文件或者是Dockerfile。下面是一个Centos 7 service文件的示例：

```
[Unit]
Description=mirth channel exporter
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
WorkingDirectory=/mirth/mirthconnect
EnvironmentFile=/etc/sysconfig/mirth_channel_exporter
ExecStart=/mirth/mirthconnect/mirth_channel_exporter

[Install]
WantedBy=multi-user.target
```
到这里就全部结束了。整个过程中只有第6步比较困难，如果你直到哪些API可用，并且直到如何解析它们，剩下的步骤就水到渠成了。

完成代码如下：

```go
// A minimal example of how to include Prometheus instrumentation.
package main

import (
	"crypto/tls"
	"encoding/xml"
	"flag"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"strconv"

	"github.com/joho/godotenv"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

/*
<map>
  <entry>
    <string>101af57f-f26c-40d3-86a3-309e74b93512</string>
    <string>Send-Email-Notification</string>
  </entry>
</map>
*/
type ChannelIdNameMap struct {
	XMLName xml.Name       `xml:"map"`
	Entries []ChannelEntry `xml:"entry"`
}
type ChannelEntry struct {
	XMLName xml.Name `xml:"entry"`
	Values  []string `xml:"string"`
}

/*
<list>
  <channelStatistics>
    <serverId>c5e6a736-0e88-46a7-bf32-5b4908c4d859</serverId>
    <channelId>101af57f-f26c-40d3-86a3-309e74b93512</channelId>
    <received>0</received>
    <sent>0</sent>
    <error>0</error>
    <filtered>0</filtered>
    <queued>0</queued>
  </channelStatistics>
</list>
*/
type ChannelStatsList struct {
	XMLName  xml.Name       `xml:"list"`
	Channels []ChannelStats `xml:"channelStatistics"`
}
type ChannelStats struct {
	XMLName   xml.Name `xml:"channelStatistics"`
	ServerId  string   `xml:"serverId"`
	ChannelId string   `xml:"channelId"`
	Received  string   `xml:"received"`
	Sent      string   `xml:"sent"`
	Error     string   `xml:"error"`
	Filtered  string   `xml:"filtered"`
	Queued    string   `xml:"queued"`
}

const namespace = "mirth"
const channelIdNameApi = "/api/channels/idsAndNames"
const channelStatsApi = "/api/channels/statistics"

var (
	tr = &http.Transport{
		TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
	}
	client = &http.Client{Transport: tr}

	listenAddress = flag.String("web.listen-address", ":9141",
		"Address to listen on for telemetry")
	metricsPath = flag.String("web.telemetry-path", "/metrics",
		"Path under which to expose metrics")

	// Metrics
	up = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "up"),
		"Was the last Mirth query successful.",
		nil, nil,
	)
	messagesReceived = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "messages_received_total"),
		"How many messages have been received (per channel).",
		[]string{"channel"}, nil,
	)
	messagesFiltered = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "messages_filtered_total"),
		"How many messages have been filtered (per channel).",
		[]string{"channel"}, nil,
	)
	messagesQueued = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "messages_queued"),
		"How many messages are currently queued (per channel).",
		[]string{"channel"}, nil,
	)
	messagesSent = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "messages_sent_total"),
		"How many messages have been sent (per channel).",
		[]string{"channel"}, nil,
	)
	messagesErrored = prometheus.NewDesc(
		prometheus.BuildFQName(namespace, "", "messages_errored_total"),
		"How many messages have errored (per channel).",
		[]string{"channel"}, nil,
	)
)

type Exporter struct {
	mirthEndpoint, mirthUsername, mirthPassword string
}

func NewExporter(mirthEndpoint string, mirthUsername string, mirthPassword string) *Exporter {
	return &Exporter{
		mirthEndpoint: mirthEndpoint,
		mirthUsername: mirthUsername,
		mirthPassword: mirthPassword,
	}
}

func (e *Exporter) Describe(ch chan<- *prometheus.Desc) {
	ch <- up
	ch <- messagesReceived
	ch <- messagesFiltered
	ch <- messagesQueued
	ch <- messagesSent
	ch <- messagesErrored
}

func (e *Exporter) Collect(ch chan<- prometheus.Metric) {
	channelIdNameMap, err := e.LoadChannelIdNameMap()
	if err != nil {
		ch <- prometheus.MustNewConstMetric(
			up, prometheus.GaugeValue, 0,
		)
		log.Println(err)
		return
	}
	ch <- prometheus.MustNewConstMetric(
		up, prometheus.GaugeValue, 1,
	)

	e.HitMirthRestApisAndUpdateMetrics(channelIdNameMap, ch)
}

func (e *Exporter) LoadChannelIdNameMap() (map[string]string, error) {
	// Create the map of channel id to names
	channelIdNameMap := make(map[string]string)

	req, err := http.NewRequest("GET", e.mirthEndpoint+channelIdNameApi, nil)
	if err != nil {
		return nil, err
	}

	// This one line implements the authentication required for the task.
	req.SetBasicAuth(e.mirthUsername, e.mirthPassword)
	// Make request and show output.
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}

	body, err := ioutil.ReadAll(resp.Body)
	resp.Body.Close()
	if err != nil {
		return nil, err
	}
	// fmt.Println(string(body))

	// we initialize our array
	var channelIdNameMapXML ChannelIdNameMap
	// we unmarshal our byteArray which contains our
	// xmlFiles content into 'users' which we defined above
	err = xml.Unmarshal(body, &channelIdNameMapXML)
	if err != nil {
		return nil, err
	}

	for i := 0; i < len(channelIdNameMapXML.Entries); i++ {
		channelIdNameMap[channelIdNameMapXML.Entries[i].Values[0]] = channelIdNameMapXML.Entries[i].Values[1]
	}

	return channelIdNameMap, nil
}

func (e *Exporter) HitMirthRestApisAndUpdateMetrics(channelIdNameMap map[string]string, ch chan<- prometheus.Metric) {
	// Load channel stats
	req, err := http.NewRequest("GET", e.mirthEndpoint+channelStatsApi, nil)
	if err != nil {
		log.Fatal(err)
	}

	// This one line implements the authentication required for the task.
	req.SetBasicAuth(e.mirthUsername, e.mirthPassword)
	// Make request and show output.
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}

	body, err := ioutil.ReadAll(resp.Body)
	resp.Body.Close()
	if err != nil {
		log.Fatal(err)
	}
	// fmt.Println(string(body))

	// we initialize our array
	var channelStatsList ChannelStatsList
	// we unmarshal our byteArray which contains our
	// xmlFiles content into 'users' which we defined above
	err = xml.Unmarshal(body, &channelStatsList)
	if err != nil {
		log.Fatal(err)
	}

	for i := 0; i < len(channelStatsList.Channels); i++ {
		channelName := channelIdNameMap[channelStatsList.Channels[i].ChannelId]

		channelReceived, _ := strconv.ParseFloat(channelStatsList.Channels[i].Received, 64)
		ch <- prometheus.MustNewConstMetric(
			messagesReceived, prometheus.GaugeValue, channelReceived, channelName,
		)

		channelSent, _ := strconv.ParseFloat(channelStatsList.Channels[i].Sent, 64)
		ch <- prometheus.MustNewConstMetric(
			messagesSent, prometheus.GaugeValue, channelSent, channelName,
		)

		channelError, _ := strconv.ParseFloat(channelStatsList.Channels[i].Error, 64)
		ch <- prometheus.MustNewConstMetric(
			messagesErrored, prometheus.GaugeValue, channelError, channelName,
		)

		channelFiltered, _ := strconv.ParseFloat(channelStatsList.Channels[i].Filtered, 64)
		ch <- prometheus.MustNewConstMetric(
			messagesFiltered, prometheus.GaugeValue, channelFiltered, channelName,
		)

		channelQueued, _ := strconv.ParseFloat(channelStatsList.Channels[i].Queued, 64)
		ch <- prometheus.MustNewConstMetric(
			messagesQueued, prometheus.GaugeValue, channelQueued, channelName,
		)
	}

	log.Println("Endpoint scraped")
}

func main() {
	err := godotenv.Load()
	if err != nil {
		log.Println("Error loading .env file, assume env variables are set.")
	}

	flag.Parse()

	mirthEndpoint := os.Getenv("MIRTH_ENDPOINT")
	mirthUsername := os.Getenv("MIRTH_USERNAME")
	mirthPassword := os.Getenv("MIRTH_PASSWORD")

	exporter := NewExporter(mirthEndpoint, mirthUsername, mirthPassword)
	prometheus.MustRegister(exporter)

	http.Handle(*metricsPath, promhttp.Handler())
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(`<html>
             <head><title>Mirth Channel Exporter</title></head>
             <body>
             <h1>Mirth Channel Exporter</h1>
             <p><a href='` + *metricsPath + `'>Metrics</a></p>
             </body>
             </html>`))
	})
	log.Fatal(http.ListenAndServe(*listenAddress, nil))
}
```

