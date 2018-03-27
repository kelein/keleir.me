---
title: Prometheus in Practice
date: 2018-01-27 15:46:35
tags: prometheus
---

# Prometheus 

![Archture](https://ordina-jworks.github.io/img/prometheus/prometheus-architecture.svg)

## docker-composer

> docker-compose.yml

```
version: '2'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '-config.file=/etc/prometheus/prometheus.yml'
    ports:
      - '9090:9090'
```

> prometheus.yml

```
global:
  scrape_interval: 5s
  external_labels:
    monitor: 'my-monitor'
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```


```
docker-compose up
```

> http://localhost:9090/status    
> http://localhost:9090/metrics


## HTTP API

- http://prometheus.in.dataengine.com/api/v1/query?query=node_cpu{instance="bj-rc-dptd-bluesharp-app-1-v-test-1:19000",mode="idle"}

- http://prometheus.in.dataengine.com/api/v1/query?query=irate(node_network_transmit_packets{device=%22lo%22,instance=%22bj-rc-devops-sftp-v-online-1:19000%22,job=%22node_exporter%22}[5m])

```
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [{
      "metric": {
        "device": "lo",
        "instance": "bj-rc-devops-sftp-v-online-1:19000",
        "job": "node_exporter"
      },
      "value": [1507878801.765, "0"]
    }]
  }
}
```

> "value": [ 1507878801.765, "0" ]    
> "value": [ 当前时间戳，当前值  ]


#### Query Series 

> Query Series by label     
>
> `GET` /api/v1/series?match[]={job=`jboName`}
- http://prometheus.in.dataengine.com/api/v1/series?match[]={job="zookeeper_exporter"}

```
{
  "status": "success",
  "data": [
    {
      "__name__": "zk_znode_count",
      "exported_instance": "10.200.3.88:2181",
      "instance": "10.200.0.192:9120",
      "job": "zookeeper_exporter"
    },
    {
      "__name__": "zookeeper_MaxClientCnxnsPerHost",
      "instance": "10.200.10.42:7071",
      "job": "zookeeper_exporter",
      "replicaId": "1"
    }
  ]
}
```

#### Query labels
- http://prometheus.in.dataengine.com/api/v1/label/job/values
```
{
  "status": "success",
  "data": [
    "apache_exporter",
    "ceph_exporter",
    "zookeeper_exporter"
  ]
}
```

#### Query Range

- curl -s 'http://prometheus.in.dataengine.com/api/v1/query_range?query=up&start=2017-11-01T20:10:30.781Z&end=2017-11-08T20:11:00.781Z&step=1h' | jq


#### API List

```
http://prometheus.in.dataengine.com/api/v1/label/__name__/values


```
- 

## Node Exporter

> docker-compose.yml  

```
version: '2'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '-config.file=/etc/prometheus/prometheus.yml'
    ports:
      - '9090:9090'
  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - '9100:9100'

```

> prometheus.yml

```
global:
  scrape_interval: 5s
  external_labels:
    monitor: 'my-monitor'
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

## Reload Config

#### (1) POST
```
curl -i -v -X POST -H 'Accept: application/json; indent=4' http://localhost:9090/-/reload
```

#### (2) SIGHUP
```
kill -HUP pid
```

## Push Gateway

![arch](http://7sbqda.com1.z0.glb.clouddn.com/Screen%20Shot%202017-07-11%20at%205.47.35%20PM.png)

`$ cat docker-compose.yml`

```
version: '2'
services:
  pushgateway:
    image: prom/pushgateway:latest
    ports:
      - "9091:9091"
```

`$ cat prometheus.yml`
```
  - job_name: 'pushgateway'
    static_configs:
      - targets: ['10.200.3.98:9091']
```

### Push Metrics

`/metrics/job/<JOBNAME>{/<LABEL_NAME>/<LABEL_VALUE>}`

```
# Send metrics to pushgateway
curl -s http://$EXPORTER_ADDR/$EXPORTER_METRIC | \ 
curl --data-binary @- http://$PGW_ADDR/metrics/job/$PGW_JOB/instance/$PGW_INSTANCE
```

> e.g:

```
curl -s 10.200.3.98:9100/metrics | 
curl --data-binary @- http://10.200.3.98:9091/metrics/job/node_exporter/instance/"10.200.3.98:9100"
```

> Add label when push metrics
```
curl -s 10.200.3.98:9100/metrics | 
curl --data-binary @- http://10.200.3.98:9091/metrics/job/node_exporter/instance/"10.200.3.98:9100"/group/kallen/platform/openstack/service/mysql
```

```
echo "kallen_dev_metric 20" | curl --data-binary @- http://10.205.16.15:9091/metrics/job/kallen_exporter/instance/10.205.16.15:8888 -v
```

```
echo "kallen_dev_metric 20" | curl --data-binary @- http://10.205.16.15:9091/metrics/job/kallen_exporter/instance/10.205.16.15:8888 -v

* About to connect() to 10.205.16.15 port 9091 (#0)
*   Trying 10.205.16.15...
* Connected to 10.205.16.15 (10.205.16.15) port 9091 (#0)
> POST /metrics/job/kallen_exporter/instance/10.205.16.15:8888 HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 10.205.16.15:9091
> Accept: */*
> Content-Length: 21
> Content-Type: application/x-www-form-urlencoded
> 
* upload completely sent off: 21 out of 21 bytes
< HTTP/1.1 202 Accepted
< Date: Fri, 03 Nov 2017 10:01:54 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host 10.205.16.15 left intact

```

## Alert Manager

> prometheus.yml

```
alerting:
  alertmanagers:
  - scheme: https
    static_configs:
    - targets:
      - "1.2.3.4:9093"
      - "1.2.3.5:9093"
      - "1.2.3.6:9093"
```

### AlertManager API
- http://10.200.3.98:9093/api/v1/alerts
- http://10.200.3.98:9093/api/v1/alerts/groups
- http://10.200.3.98:9093/api/v1/status
- http://10.200.3.98:9093/api/v1/silences


### Receivers

```
route:
  group_by: [ alertname ]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 2h
  receiver: 'slacker'
```

> Multiple Receiver

```
route:
  group_by: [ alertname ]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 2h
  receiver: 'slacker'
  
  routes:
    - match_re:
        job: .+
      receiver: 'alerta'

receivers:
  - name: 'alerta'
    webhook_configs:
      - url: 'http://10.203.16.119:8181/api/webhooks/prometheus'
        send_resolved: true

  - name: slacker
    ...
```

### AlertManager HA

> alertmanager
```
# On a machine named "am-1":
wget https://github.com/prometheus/alertmanager/releases/download/v0.7.1/alertmanager-0.7.1.linux-amd64.tar.gz
tar -xzf alertmanager-*.linux-amd64.tar.gz
cd alertmanager-*
./alertmanager -config.file simple.yml -mesh.peer=am-2:6783


# On a machine named "am-2":
wget https://github.com/prometheus/alertmanager/releases/download/v0.7.1/alertmanager-0.7.1.linux-amd64.tar.gz
tar -xzf alertmanager-*.linux-amd64.tar.gz
cd alertmanager-*
./alertmanager -config.file simple.yml -mesh.peer=am-1:6783
```
> prometheus.yml
```
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['am-1:9093', 'am-2:9093']
```

#### Experment on Docker

```
version: '2'
services:
  alertmanager_peer1:
    image: prom/alertmanager:latest
    volumes:
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      - ./simple.yml:/etc/alertmanager/simple.yml
    command: [
      '-config.file=/etc/alertmanager/simple.yml', 
      '-mesh.peer=alertmanager_peer2:6783'
    ]
    ports:
      - '9093:9093'
    hostname: 'alertmanager_peer1'
    links:
      - alertmanager_peer2
  
  alertmanager_peer2:
    image: prom/alertmanager:latest
    volumes:
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
      - ./simple.yml:/etc/alertmanager/simple.yml
    command: [
      '-config.file=/etc/alertmanager/simple.yml', 
      '-mesh.peer=alertmanager_peer1:6783'
    ]
    ports:
      - '9094:9093'
    hostname: 'alertmanager_peer2'

```

```
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
      - targets: [
        "10.200.3.98:9093",
        "10.200.3.98:9094"
      ]
```

### Alert Stats

```
inactive --> pending --> firing
```

```
Connections:

Address: 172.18.0.4:59634
Info: none 02:42:ac:12:00:04(alertmanager_peer1)
State: pending

Address: 172.18.0.2:6783
Info: none 02:42:ac:12:00:02(alertmanager_peer3)
State: pending

Address: 172.18.0.4:6783
Info: Multiple connections to 02:42:ac:12:00:04(alertmanager_peer1) added to 02:42:ac:12:00:03(alertmanager_peer2), retry: 2018-01-31 16:55:58.292680256 +0800 CST m=+105335.286293642
State: failed
```

## PromQL 

```
topk(10, count by (__name__)({__name__=~".+"}))
topk(10, count by (job)({__name__=~".+"}))

sum(sort_desc(sum_over_time(ALERTS{alertstate=`firing`}[24h]))) by (alertname)

count(ALERTS{alertstate=`firing`}) by (alertname)

sum(rate(container_cpu_user_seconds_total{beta_kubernetes_io_os="linux"}[10m])) by (instance)

topk(5, sum(rate(container_cpu_user_seconds_total{}[1h])) by (instance, pod_name))
```

> Or query

```
up{instance=~"192.168.156.30:9102|192.168.38.25:9102"}

up{job="mysql_exporter"} or up{job="redis_exporter"}
```

## Prometheus Metric

```
MustNewConstMetric(desc *Desc, valueType ValueType, value float64, labelValues ...string)

NewDesc(fqName, help string, variableLabels []string, constLabels Labels)

BuildFQName(namespace, subsystem, name string)
```

> e.g: 

```
ch <- prometheus.MustNewConstMetric(
	prometheus.NewDesc(
		prometheus.BuildFQName(namespace, mountsInfoSubsystem, "state"),
		"Mounts State Info, via /proc/mounts (ro-Readonly, rw-ReadWrite)",
		[]string{"device", "mountpoint", "fstype", "mountstate"},
		nil,
	),
	prometheus.GaugeValue, value,
	minfo["device"], minfo["mountpoint"], minfo["fstype"], minfo["mountstate"],
)
```

## Prometheus Federation

```
- job_name: 'federate'
  scrape_interval: 15s

  honor_labels: true
  metrics_path: '/federate'

  params:
    'match[]':
      - '{job="prometheus"}'
      - '{__name__=~"job:.*"}'

  static_configs:
    - targets:
      - 'source-prometheus-1:9090'
      - 'source-prometheus-2:9090'
      - 'source-prometheus-3:9090'
```

### Hierarchical federation

```
global:
  external_labels:
    datacenter: global  # In a HA setup, this would be global1 or global2

scrape_configs:
  - job_name: datacenter_federation
    honor_labels: true
    metrics_path: /federate
    params:
      match[]:
        - '{__name__=~"^job:.*"}'
    static_configs:
      - targets:
        - eu-west-1-prometheus:9090

etc.
```

> sharding mode 

```
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prome-master'

scrape_configs:
  - job_name: federate
    honor_labels: true
    metrics_path: /federate
    params:
      'match[]':
          - '{__name__=~".+"}'
    static_configs:
      - targets: [
        "prome-salve-01:9090",
        "prome-slave-02:9090",
        "prome-slave-03:9090" 
      ]
```



![arch](https://stefanprodan.com/assets/prometheus-on-docker.png)


## Refers

- [Monitoring with Prometheus, Grafana & Docker](https://finestructure.co/blog/2016/5/16/monitoring-with-prometheus-grafana-docker-part-1) 

- [How to Setup Monitoring for Docker Containers using Prometheus](https://linoxide.com/containers/setup-monitoring-docker-containers-prometheus/)

- [PROMETHEUS CONFIGURATION](https://prometheus.io/docs/operating/configuration/)

- http://codeblog.dotsandbrackets.com/highly-available-kafka-cluster-docker/

- [Writing a Jenkins exporter in Python](https://www.robustperception.io/writing-a-jenkins-exporter-in-python/)

- ##### [Push Based Monitoring Service with Prometheus](http://zpjiang.me/2017/07/10/Push-based-monitoring-for-Prometheus/) 

- ##### [Prometheus Pushgateway](https://quay.io/repository/prometheus/pushgateway)


- #### [KubeCon 2017 - Prometheus Takeaways](https://pracucci.com/kubecon-2017-prometheus-takeaways.html)

- #### [Understanding and Extending Prometheus AlertManager](http://calcotestudios.com/talks/slides-understanding-and-extending-prometheus-alertmanager.html#/)

## Features

- [**Prometheus 2.0: New storage layer dramatically increases monitoring scalability for Kubernetes and other distributed systems**](https://coreos.com/blog/prometheus-2.0-storage-layer-optimization)

- [**Writing a Time Series Database from Scratch**](https://fabxc.org/blog/2017-04-10-writing-a-tsdb/)

- #### [System Properties Comparison InfluxDB vs. Prometheus](https://db-engines.com/en/system/InfluxDB%3BPrometheus)

## Solutions

- [360基于Prometheus的在线服务监控实践](http://dbaplus.cn/news-72-1462-1.html)
- [Prometheus在Kubernetes下的监控实践](http://yunlzheng.github.io/2017/07/04/prometheus-kubernates/)

- [基于etcd+confd对监控prometheus的服务注册发现](http://www.net-add.com/a/dockerzhuanti/fuwuzhucefaxian/2017/0421/42.html)

- [收集golangruntime指标](http://www.codedata.cn/cdetail/Go/other/go-runtime-metrics)

## New Found

- [Netsil](https://netsil.com/)
- Netdata
