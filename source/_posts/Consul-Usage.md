---
title: Consul Usage
date: 2017-01-27 16:48:32
tags: consul
---

# Consul

![consul](https://www.consul.io/assets/images/consul-arch-420ce04a.png)

<!-- more -->

- TCP/RPC 8300 Node
- Lan Gossip 8301 Client
- Wan Gossip 8302 Server
- Http 8500 API/UI
- DNS 8500 Dig

## Start Agent

```
consul agent -dev
```

## Members

```
consul members

Node		Address         Status  Type    Build  Protocol  DC   Segment
test-1  	127.0.0.1:8301  alive   server  1.0.0  2         dc1  <all>
```

```
curl localhost:8500/v1/catalog/nodes

[
  {
    "CreateIndex": 5,
    "Meta": {
      "consul-network-segment": ""
    },
    "Datacenter": "dc1",
    "ModifyIndex": 6,
    "Address": "127.0.0.1",
    "TaggedAddresses": {
      "lan": "127.0.0.1",
      "wan": "127.0.0.1"
    },
    "Node": "bj-rc-devops-dkl-v-test-1.host.dataengine.com",
    "ID": "0e6c609d-c54b-c28b-95b1-827bff85ee64"
  }
]
```

## Consul Cluster

#### Server Mode
```
nohup consul agent -server -bootstrap-expect=1 \
    -data-dir=/opt/Consul/data -node=kallen-1 -bind=10.200.3.98 \
    -enable-script-checks=true -config-dir=/opt/Consul/etc/consul.d > consul.log 2>&1 &
```

#### Agent Mode
```
nohup consul agent -data-dir=/opt/Consul/data -node=kallen-2 \
    -bind=10.200.3.100 -enable-script-checks=true -config-dir=/opt/Consul/etc/consul.d > consul.log 2>&1 &
```

#### Start WebUI

```
nohup consul agent -ui -data-dir=/opt/Consul/data -node=kallen-2 \
    -bind=10.200.3.100 -enable-script-checks=true -config-dir=/opt/Consul/etc/consul.d > consul.log 2>&1 &
```

```
nohup consul agent -ui -client=0.0.0.0 -data-dir=/opt/Consul/data -node=kallen-2 \
    -bind=10.200.3.100 -enable-script-checks=true -config-dir=/opt/Consul/etc/consul.d > consul.log 2>&1 &
```

>  -client 0.0.0.0  
>  Sets the address to bind for client access. This includes RPC, DNS,HTTP and HTTPS (if configured).

#### Join Cluster

```
consul join 10.200.3.100 

Successfully joined cluster by contacting 1 nodes.
```

#### Leave Cluster

```
consul leave

consul members
Node      Address            Status  Type    Build  Protocol  DC   Segment
kallen-1  10.200.3.98:8301   alive   server  1.0.0  2         dc1  <all>
kallen-2  10.200.3.100:8301  left    client  1.0.0  2         dc1  <default>
```

#### Service Register

```
PUT http://10.200.3.100:8500/v1/agent/service/register

{
  "address": "10.200.3.98",
  "id": "httpd_exporter_1",
  "name": "httpd_exporter_1",
  "port": 9413,
  "tags": [
    "kallen"
  ]
}
```

#### Check 

```
"check": {
    "name": "mysql_exporter_check",
    "http": "http://10.200.3.98:33109/metrics",
    "tls_skip_verify": false,
    "method": "GET",
    "interval": "60s",
    "timeout": "1s"
  }
```

#### Service Deregister

```
PUT http://10.200.3.100:8500/v1/agent/service/deregister/node_exporter_1

{
  "ID": "node_exporter_1"
}
```

> 基于ZooKeeper、Etcd等分布式键值对存储服务来建立服务发现系统在现在看起来也不是一种很好的方案，一方面是因为它们只能提供基本的数据存储功能，还需要在外围做大量的开发才能形成完整的服务发现方案。另一方面是因为它们都是强一致性系统，在集群发生分区时会优先保证一致性、放弃可用性，而服务发现方案更注重可用性，为了保证可用性可以选择最终一致性

## Refers

-  [服务发现系统consul-HTTP API](http://www.codeweblog.com/%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E7%B3%BB%E7%BB%9Fconsul-http-api/)

- [Raft算法演示](http://thesecretlivesofdata.com/raft/) 

- [在线支付公司Stripe的服务发现架构设计过程分享](http://www.infoq.com/cn/articles/service-discovery-at-stripe)

- [Consul 简介和快速入门](https://book-consul-guide.vnzmi.com/) 
