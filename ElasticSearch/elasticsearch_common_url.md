---
categories: elasticsearch

tags: 
  - Elasticsearch

title: Elasticsearch常用请求

---

# 查看ES状态

集群健康

```
curl -XGET 'localhost:9200/_cat/health?pretty'
```
响应如下
```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

颜色 | 说明
---|---
green | 表示一切正常，集群中所有副本完好
yellow | 表示所有数据可用，数据没有丢失，但有些副本丢失了，该情况一般是由于某些结点宕机导致
red | 发生了数据丢失，会有部分数据查询不到，此时应该修复集群


集群结点

```
curl -XGET 'localhost:9200/_cat/nodes&pretty'
```
响应如下
```
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

列出所有索引

```
curl -XGET 'localhost:9200/_cat/indices&pretty'
```

# 查看数据

# 创建修改数据

# ES设置