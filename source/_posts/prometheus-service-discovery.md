---
title: Prometheus Service Discovery
date: 2018-03-27 16:31:33
tags: prometheus
---

# Prometheus Service Discovery

## Service Discovery

- Build-in Service Discovery
- JSON file Service Discovery
- Consul Service Discovery

<!-- more -->

### (1) Build-in Discovery

> prometheus/retrieval/targetmanager.go

- Sources() []string，返回当前provider的标示。target manager将返回的string作为target group的ID

- Run(up chan<- config.TargetGroup, done <-chan struct{})，启动target provider。provider会将最新的target group信息输出到up这个channel中，以通知target manager。


> prometheus/config/config.go

- Targets，会被prometheus解析，用于获得target metrics的访问信息，会被添加到metrics记录上；
- Labels，普通的label，会被添加到metrics记录上；
- Source，等同于ID。


#### Kubernetes sd_config

```
- job_name: kubelets
  scrape_interval: 1m
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: https
  kubernetes_sd_configs:
  - api_server: null
    role: node
    namespaces:
      names: []
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    insecure_skip_verify: true
```

### (2) JSON file service discovery

```
scrape_configs:
  - job_name: 'dummy'  # This is a default value, it is mandatory.
    file_sd_configs:
      - files:
        - targets.json
```

```
- job_name: 'test'
  scrape_interval: '1s'
  file_sd_configs:
    - files:
      - 'sd_configs/*.json'
```

```
[
  {
    "targets": [
      "localhost:10003"
    ],
    "labels": {
      "job": "ExporterInc"
    }
  }
]
```

```
INFO[0000] Starting target manager...                    source="targetmanager.go:63"
```

### (3) Consul Discovery

```
scrape_configs:  
 - job_name: consul
   consul_sd_configs:
     - server: 'localhost:8500'
   relabel_configs:
     - source_labels: [__meta_consul_tags]
       regex: .*,monitor,.*
       action: keep
     - source_labels: [__meta_consul_service]
       target_label: service
```

##### With ACL Setting
```
scrape_configs:
  - job_name: 'consul_sd'
    consul_sd_configs:
      - server: 'localhost:8500'
        token: '63c7ce81-1e3b-9081-dff5-d91ede002ceb'
    relabel_configs:
      - source_labels: ['__meta_consul_service']
        regex: prometheus\-(.+)
        replacement: '${1}'
        target_label: 'job'
      - source_labels: ['__meta_consul_service']
        regex: consul
        action: drop
      - source_labels: ['__meta_consul_node']
        target_label: 'node'
```

#### snmp_config 

```
- job_name: snmp_exporter-hwswitch20232
  params:
    module:
    - hwswitch
  scrape_interval: 1m
  scrape_timeout: 10s
  metrics_path: /snmp
  scheme: http
  static_configs:
  - targets:
    - 10.200.3.233
    labels:
      business: 6f289f7e-e1c7-4e9a-b217-5580cd61f397
      sub_business: d185b50e-cba2-40b8-8901-1181ecc63ad3
  relabel_configs:
  - source_labels: [__address__]
    separator: ;
    regex: (.*)
    target_label: __param_target
    replacement: $1
    action: replace
  - source_labels: [__param_target]
    separator: ;
    regex: (.*)
    target_label: instance
    replacement: $1
    action: replace
  - source_labels: []
    separator: ;
    regex: (.*)
    target_label: __address__
    replacement: 127.0.0.1:20232
    action: replace
```

> snmp file_sd_config:

```
  - job_name: snmp_exporter
    params:
      module:
      - hwswitch
    scrape_interval: 1m
    scrape_timeout: 10s
    metrics_path: /snmp
    scheme: http
    
    file_sd_configs:
      - files:
        - 'snmp_config/*.json'
    
    relabel_configs:
    - source_labels: [__address__]
      separator: ;
      regex: (.*)
      target_label: __param_target
      replacement: $1
      action: replace
    - source_labels: [__param_target]
      separator: ;
      regex: (.*)
      target_label: instance
      replacement: $1
      action: replace
    - source_labels: []
      separator: ;
      regex: (.*)
      target_label: __address__
      replacement: 10.200.0.185:20232
      action: replace
```

> snmp.json

```
[
  {
    "targets": [ "10.200.3.233" ],
    "labels": {
      "job": "snmp_h3c"
    }
  }
]

```

## Refers

- [Prometheus对Swarm的服务发现插件](http://www.jianshu.com/p/2fb7208c8152)

- [Finding Consul services to monitor with Prometheus](https://www.robustperception.io/finding-consul-services-to-monitor-with-prometheus/)

- [Using JSON file service discovery with Prometheus](https://www.robustperception.io/using-json-file-service-discovery-with-prometheus/)
