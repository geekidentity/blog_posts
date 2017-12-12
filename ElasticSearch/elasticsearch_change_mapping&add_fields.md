---
categories: elasticsearch

tags: 
  - Elasticsearch

title: Elasticsearch修改Mpping和在索引上添加field

date: 2017-07-26
---

我们有时候需要修改ES的Mapping，ES的mapping修改非常简单，和创建新的mapping一样，重发发一次请求，ES会更新指定的mapping，例如：

```
curl -XPUT localhost:9200/_template/temaplate_name?pretty -d'{
	"template": "index_*",
	"aliases": {
		"{index}_alias": {
			
		}
	},
	"mappings": {
		"mapping_name": {
			"_all": {
				"enabled": false
			},
			"properties": {
				"fields_1": {
					"type": "keyword"
				},
				"fields_2": {
					"type": "keyword"
				},
			}
		}
	}
}
```

**注意：** mapping修改后对已创建的索引不会生效，只对新创建的索引生效。

如果对已存在的索引添加filed，可用下面的请求：

```
curl -XPUT localhost:9200/index_name/_mapping/mapping_name?pretty -d'{
			"properties": {
				"new_feild": {
					"type": "keyword"
				}	
			}
}'
```
