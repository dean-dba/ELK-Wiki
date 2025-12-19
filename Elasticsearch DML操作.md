# Elasticsearch DML操作

#### Elasticsearch DML操作主要针对创建索引、删除索引、插入文档、更新文档、删除文档、批量操作这几块内容展开

创建索引：生产环境中通过模板创建
```
PUT aaaa
{
  "mappings": {
    "properties": {
      "currency": {
        "type": "keyword"
      },
      "customer_birth_date": {
        "type": "date"
      },
      "sales": {
        "type": "long"
      }
    }
  }
}
```

删除索引
```
DELETE aaaa
```

单条写入文档：存在即更新(PUT 操作须指定文档ID，不指定文档ID须用POST)
```
PUT test/_doc/1
{"currency": "abcd", "customer_birth_date": "2025-12-19",'sales':100}

POST test/_doc
{"currency": "abcd", "customer_birth_date": "2025-12-19","sales":100}
```

批量写入，index：存在即更新，create：存在会报错
```
POST _bulk
{"index":{"_index":"test","_id":100}}
{"currency": "ccc", "customer_birth_date": "2025-12-18"}
{ "index" : { "_index" : "test","_id":200 } }
{"currency": "aaa", "customer_birth_date": "2025-12-18"}
{ "index" : { "_index" : "test","_id":300 } }
{"currency": "bbb", "customer_birth_date": "2025-12-18"}
```

删除文档，默认内部会标记删除，段合并时释放磁盘空间
```
DELETE test/_doc/1

POST test/_delete_by_query
{"query":{"term":{"currency":"abcd"}}}
```

更新文档：索引必须设置_source:true(默认配置)
```
POST test/_update/100
{"doc":{"currency": "hhh"}}

POST test/_update_by_query
{
  "script": {
    "lang": "painless",
	"source": "ctx._source.currency = params.new_value",
    "params": {
      "new_value": "USA"
    }
  },
  "query": {
    "term": {
      "currency": "CN"
    }
  }
}
```

索引间同步数据，_reindex支持跨集群同步数据
```
POST _reindex
{
  "source": {
    "index": "index_1"
  },
  "dest": {
    "index": "index_2"
  }
}
```

ES 默认脚本：painless，ctx._source(context source)：当前文档的原始数据
```
  "script": {	
    "lang": "painless",
    "source": "ctx._source.currency = params.new_value",
    "params": {"new_value": "USA"}
```

##### 至此，Elasticsearch DML操作就介绍完成了，请开始你的表演吧！

&nbsp;

**有兴趣的小伙伴，可加联系方式：Vx：+86 183 1026 3857&nbsp;&nbsp;&nbsp;&nbsp;Telegram：+86 189 1127 2119**  
