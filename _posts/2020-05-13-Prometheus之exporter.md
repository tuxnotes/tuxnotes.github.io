---
layout: post
date: 2020-05-13
title: Prometheus之exporter
author: tux
tags: prometheus exporter
---

### Exporter

本文转自https://songjiayang.gitbooks.io/prometheus/content/exporter/

在prometheus中负责数据汇报的程序统一叫Exporter，而不同的Exporter负责不同的业务。它们具有统一的命名格式，即xx_exporter,例如负责主机信息收集的叫node_exporter. Prometheus社区已经提供了很多exporter，详情参考[这里][https://prometheus.io/docs/instrumenting/exporters/#exporters-and-integrations]。

### 文本格式

因为一个Exporter本质上就是将收集的数据，转化为对应的文本格式，并提供http请求。所以先介绍一下Prometheus文本数据格式。

Exporter将收集到的数据转化后的文本内容以行(\n)为单位，空行将被忽略，文本内容最后一行为空行。

#### 注释

文本内容如果以`#`开头通常表示注释：

- 以`# HELP`开头表示`metric`帮助说明；
- 以`# TYPE`开头表示定义`metric`类型，包含`conter`, `gauge`, `histogram`, `summary`和`untyped`类型；
- 其他表示一般注释，供阅读使用，将被Prometheus忽略。

#### 采样数据

内容如果不以`#`开头，表示采样数据。它通常紧挨着类型定义行，满足一下格式：

```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```

下面是个完整的例子：

```
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

不过需要注意的是，假设采样数据 metric 叫做 `x`, 如果 `x` 是 `histogram` 或 `summary` 类型必需满足以下条件：

- 采样数据的总和应表示为`x_sum`。
- 采样数据的总量应表示为`x_count`。
- `summary`类型的采样数据的quantile应表示为`x{quantile="y"}`。
- `histogram`类型的采样分区统计数据将表示为`x_bucket{le="y"}。
- `histogram`类型的采样必须包含`x_bucket{le="+Inf"}`,它的值等于`x_count`的值。
- `summary`和`histogram`中的`quantile`和`le`必须按从小到大顺序排列。

### Sample Exporter

既然一个exporter就是将收集的数据转化为文本格式，并提供http请求即可，那很容易自己实现一个简单的exporter.

下面使用`golang`实现一个简单的`sample_exporter`，代码大致如下：

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, exportData)
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}

var exportData string = `# HELP sample_http_requests_total The total number of HTTP requests.
# TYPE sample_http_requests_total counter
sample_http_requests_total{method="post",code="200"} 1027 1395066363000
sample_http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
sample_msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
sample_metric_without_timestamp_and_labels 12.47

# A histogram, which has a pretty complex representation in the text format:
# HELP sample_http_request_duration_seconds A histogram of the request duration.
# TYPE sample_http_request_duration_seconds histogram
sample_http_request_duration_seconds_bucket{le="0.05"} 24054
sample_http_request_duration_seconds_bucket{le="0.1"} 33444
sample_http_request_duration_seconds_bucket{le="0.2"} 100392
sample_http_request_duration_seconds_bucket{le="0.5"} 129389
sample_http_request_duration_seconds_bucket{le="1"} 133988
sample_http_request_duration_seconds_bucket{le="+Inf"} 144320
sample_http_request_duration_seconds_sum 53423
sample_http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP sample_rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE sample_rpc_duration_seconds summary
sample_rpc_duration_seconds{quantile="0.01"} 3102
sample_rpc_duration_seconds{quantile="0.05"} 3272
sample_rpc_duration_seconds{quantile="0.5"} 4773
sample_rpc_duration_seconds{quantile="0.9"} 9001
sample_rpc_duration_seconds{quantile="0.99"} 76656
sample_rpc_duration_seconds_sum 1.7560473e+07
sample_rpc_duration_seconds_count 2693
`
```

运行此程序，访问 `http://localhost:8080/metrics`, 将看到返回的文本内容。

### 与Prometheus集成

利用 Prometheus 的 static_configs 来收集 `sample_exporter` 的数据。打开 `prometheus.yml` 文件, 在 `scrape_configs` 中添加如下配置：

```yaml
- job_name: "sample"
    static_configs:
      - targets: ["127.0.0.1:8080"]
```

重启加载配置，然后到 Prometheus Console 查询，你会看到 `simple_exporter` 的数据。

