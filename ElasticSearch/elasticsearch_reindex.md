---
categories: elasticsearch

tags: 
  - Elasticsearch

title: Elasticsearch不停服务重新索引

date: 2017-10-25
---

ES已经有的字段其类型是无法修改的，因此当要修改字段类型时，必须重建索引。

ES [Reindex API(EN)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) [中文文档](http://cwiki.apachecn.org/display/Elasticsearch/Reindex+API)

要注意，reindex API不会建立目标索引，同时也不会复制原索引的设置，因此在运行_reindex操作之前要先创建目标索引，包括设置映射，分片计数，副本等所有必要设置。

reindex最简单的功能是将文档从一个索引复制到另一个索引：将文档从twitter索引复制到new_twitter索引中

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
'

```

返回如下所示结果：

```JSON
{
  "took" : 147,
  "timed_out": false,
  "created": 120,
  "updated": 0,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

就像`_update_by_query`一样，_reindex获取源索引的快照，但其目标必须是不同的索引，因此不会发生版本冲突。  dest元素可以像index API一样进行配置，以控制乐观并发控制。 只需将version_type（如上所述）或将其设置为internal 则Elasticsearch会将文档全部转储到目标索引中，并覆盖具有相同类型和ID的任何内容：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
'

```
将version_type设置为external则Elasticsearch从源索引中保留版本，并在目标索引中创建缺少的文档，并更新在目标索引中具有比源索引中更旧版本的任何文档：