---
layout: post
date: 2021-03-10
title: 使用Python实现prometheus-exporter的两个示例
author: tux
tags: prometheus
---

示例转自网络，仅用于个人学习，原文连接在reference部分给出。

# 示例1

```python
import psutil
import socket
from prometheus_client import Gauge,start_http_server
from time import sleep

g = Gauge('cup_use_percent_test_metric', 'Description of gauge',['hostip'])
host_ip = socket.gethostbyname(socket.getfqdn(socket.gethostname()))      #获取本机IP

def get_cup_use():
    cup_use_percent = psutil.cpu_percent(0.5)            #获取CPU使用率
    g.labels(hostip=host_ip).set(cup_use_percent)       #本机IP传入labels，CPU使用率传入value

if __name__ == '__main__':
    start_http_server(8006)           #8006端口启动
    while True:
        get_cup_use()
        sleep(10)

```
设置标签

```python
from prometheus_client import Counter
c = Counter('my_requests_total', 'HTTP Failures', ['method', 'endpoint'])
c.labels('get', '/').inc()
c.labels('post', '/submit').inc()
#Labels can also be passed as keyword-arguments:

from prometheus_client import Counter
c = Counter('my_requests_total', 'HTTP Failures', ['method', 'endpoint'])
c.labels(method='get', endpoint='/').inc()
c.labels(method='post', endpoint='/submit').inc()

#Gauge example:
from prometheus_client import Gauge
g = Gauge('my_inprogress_requests', 'Description of gauge',['mylabelname'])

g.labels(mylabelname='labelname').set(30)

```
remove方法

只能移除含有label的metric，remove里参数全部label values

```python
g1 = Gauge('my_requests_total', 'HTTP Failures', ['method', 'endpoint'])
g1.labels(method='get', endpoint='/').set(random.random())
g1.remove('get','/')
```

# 示例2

```python
import prometheus_client
from prometheus_client import Gauge
from prometheus_client.core import CollectorRegistry
from flask import Response, Flask
from nmap import nmap
from flask_apscheduler import APScheduler
from psutil import virtual_memory
from psutil import cpu_times

app = Flask(__name__)

scan_data = []

def scan_port():
    host_list = ['192.168.80.100', '192.168.80.101', '192.168.80.102', '192.168.80.103', '192.168.80.104',
                 '192.168.10.11']
    for i in host_list:
        nm = nmap.PortScanner()
        port = '22'
        nm.scan(i, port)
        name = nm[i]['tcp'][int(port)]['name']
        port_state = nm[i]['tcp'][int(port)]['state']
        version = nm[i]['tcp'][int(port)]['version']
        scan_data.append({'host': i, 'port': port, 'port_state': port_state, 'name': name, 'version': version})

class SchedulerConfig(object):
    JOBS = [
        {
            'id': 'update_job',
            'func': '__main__:update_job',
            'args': None,
            'trigger': 'interval',
            'seconds': 10,
        }
    ]

def update_job():
    scan_port()
    print("数据更新完成")

# 为实例化的flask引入定时任务配置
app.config.from_object(SchedulerConfig())

REGISTRY = CollectorRegistry(auto_describe=False)

host_info = Gauge("host_info", "主机信息扫描", ["host", "port", "port_state", "name", "version"], registry=REGISTRY)

mem_percent = Gauge("system_memory_percent", "内存使用率", registry=REGISTRY)

cpu_percent = Gauge("system_cpu_percent", "CPU使用率", registry=REGISTRY)

@app.route("/metrics")
def metrics():
    mem_percent.set(virtual_memory().percent)
    cpu_percent.set(cpu_times().system)

    for i in scan_data:
        host = i.get('host')
        port = i.get('port')
        port_state = i.get('port_state')
        name = i.get('name')
        version = i.get('version')
        host_info.labels(host, port, port_state, name, version)
    return Response(prometheus_client.generate_latest(REGISTRY), mimetype="text/plain")

if __name__ == "__main__":
    # 实例化APScheduler
    scheduler = APScheduler()
    # 把任务列表载入实例flask
    scheduler.init_app(app)
    # 启动任务计划
    scheduler.start()
    # 在debug模式下，Flask的重新加载器将加载两次,因此flask总共有两个进程,会导致定时任务重复执行,禁用重新加载器可解决此问题，或者debug设为false
    app.run(host="0.0.0.0", port=8000, debug=True, use_reloader=False)
```

# Reference

- https://blog.csdn.net/specter11235/article/details/89198032
- https://mikuac.com/archives/742/