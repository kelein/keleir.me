---
title: Prometheus Storage
date: 2018-01-27 15:59:47
tags: prometheus
---

# Prometheus Storage

On average, Prometheus uses only around 1-2 bytes per sample. Thus, to plan the capacity of a Prometheus server, you can use the rough formula:

```
needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample
```

## Concepts

-  chunks this data together in chunks of constant size (`1024 bytes`)
- keeps all the currently used (incomplete) chunks in memory
- batch up write operations to store chunks on disk

## Optimize

### Memory Options

- `storage.local.memory-chunks`

> how many chunks can Prometheus keep in memory. Remember, it’s the number of chunks, not the size in bytes.   
> Suggested value: `<total memory in bytes> / 1024 / 6`.

- `storage.local.max-chunks-to-persist`

> how many chunks can be waiting to be written to the disk.
> Suggested value: `memory-chunks / 2`

### The rushed mode

> When the number of chunks in memory, waiting to be persisted to disk, grows too much, Prometheus enters `the rushed mode` and speed up persisting chunks. 

Prometheus calculate an urgency score, as the number of chunks waiting for persistence in relation to `max-chunks-to-persist` and on how much the number of chunks in memory exceeds the `memory-chunks`.

- urgency_score > 0.8: Enter Rushed Mode
- urgency_score < 0.7: Leave Rushed Mode

## Snapshot

### Snapshot API

> Prometheus V2.0 提供了snapshot机制用于TSDB Backend数据备份, 创建快照API开启方式:  `--web.enable-admin-api` 



```
$ curl -XPOST http://localhost:9090/api/v2/admin/tsdb/snapshot

$ curl -X POST http://prometheus.in.dataengine.com/api/v2/admin/tsdb/snapshot


{
  "name": "2017-12-05T06:42:03Z-32d47b4db4c3e108"
}
```

##### Data Directory

```
data
├── 01C0JGQ905RK2ZZCQS8F7VSNV7
│   ├── chunks
│   ├── index
│   ├── meta.json
│   └── tombstones
├── 01C0JQJTVZ6JC5MQJFSG145979
│   ├── chunks
│   ├── index
│   ├── meta.json
│   └── tombstones
├── lock
├── snapshots
│   ├── 2017-12-05T06:42:03Z-32d47b4db4c3e108
│   └── 2017-12-05T06:42:44Z-6be0c2980c1989c5
└── wal
    ├── 000001
    └── 000002
```

##### Snapshot Directory

```
2017-12-05T06:42:44Z-6be0c2980c1989c5
└── 01C0K07PSNB97D6TXJQ7YE9ST1
    ├── chunks
    │   └── 000001
    ├── index
    ├── meta.json
    └── tombstones
```

> meta.json

```
{
  "version": 1,
  "ulid": "01C0JPHXJ8BBG194HPNZFP7AR7",
  "minTime": 1512446400000,
  "maxTime": 1512456122548,
  "stats": {
    "numSamples": 321304,
    "numSeries": 604,
    "numChunks": 3020
  },
  "compaction": {
    "level": 1,
    "sources": [
      "01C0JPHXJ8BBG194HPNZFP7AR7"
    ]
  }
}
```

- **maxTime - minTime = storage.tsdb.min-block-duration**  (default: 2h)
- meta.json 文件存储了 series 起止时间 / metircs 数量 / block 目录名称;
- 每个snapshot 由一个或多个 Chunk Block 组成, 这些Block 共同存储了Prometheus TSDB 的全量数据; 
- Chunk Block 生成的数量由 storage.tsdb.min-block-duration 和 storage.tsdb.max-block-duration 控制; 

 

```
caller=compact.go:361 component=tsdb msg="compact blocks" count=1
mint=1512532800000 maxt=1512539893781
```



> Prometheus Version: 1.7.0
```
time="2017-11-20T03:58:23+08:00" level=error msg="Storage needs throttling. Scrapes and rule evaluations will be skipped." chunksToPersist=92938 memoryChunks=493334 source="storage.go:1007" urgencyScore=1 
```


## Refers

- ##### [KubeCon 2017 - Prometheus Takeaways](https://pracucci.com/kubecon-2017-prometheus-takeaways.html)
- ##### [How much RAM does my Prometheus need for ingestion?](https://www.robustperception.io/how-much-ram-does-my-prometheus-need-for-ingestion/)
- ##### [Writing a Time Series Database from Scratch](https://fabxc.org/tsdb/)
- ##### [Continuous Improvement in Monitoring with Prometheus 2.0](https://www.opcito.com/continuous-improvement-in-monitoring-with-prometheus-2-0/)

- ##### [剖析Prometheus的内部存储机制](http://www.cnblogs.com/vovlie/p/7709312.html)

- ##### [LevelDB 实现分析](http://blog.jobbole.com/111792/)

## Research

- Prometheus WAL Log

- Chunk Data 

- Snapshot 
