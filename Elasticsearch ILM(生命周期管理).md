# Elasticsearch ILM(生命周期管理)

#### Elasticsearch ILM功能是从6.6版本引入，并在6.7版本正式推出的原生索引生命周期管理功能，解决了之前需要程序处理所带来的各种繁琐问题

标准3节点集群配置方案，ILM(索引生命周期管理)基于data_hot、data_warm、data_cold节点创建
```
节点1: node.roles: [data_content, data_hot, master, ingest]
节点2: node.roles: [data_content, data_warm, master, ingest]
节点3: node.roles: [data_content, data_cold, master, ingest]
```

手动触发滚动索引
```
POST test-400/_rollover
{
  "conditions": {
    "max_age": "7d",
    "max_docs": 5,
    "max_primary_shard_size": "50gb"
  }
}
```

shrink(压缩索引)前提条件：
```
索引只读："index.blocks.write":true
索引副本设置为0："index.number_of_replicas":0
所有主分片必须在同一节点："index.routing.allocation.include._tier_preference":"data_hot"
目标分片数必须是原分片数的约数(例如，源分片8个可以压缩为4、2、1个)

PUT test-400-20260107/_settings
{
  "settings": {
    "index.blocks.write": true,
    "index.number_of_replicas": 0,
    "index.routing.allocation.include._tier_preference": "data_hot"
  }
}

POST test-400-20260107/_shrink/test-400-20260107-new
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 1,
    "index.codec": "best_compression"
  }
}

POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "test-400-20260107",
        "alias": "test-400-new"
      }
    },
    {
      "add": {
        "index": "test-400-20260107-new",
        "alias": "test-400-new"
      }
    }
  ]
}
```

ILM(索引生命周期管理)阶段：热阶段、温阶段、冷阶段、删除阶段
ILM默认检查时间：10m
```
查看索引有无异常：GET test-400/_ilm/explain

创建ILM：
PUT _ilm/policy/policy-test-400
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "set_priority": {
            "priority": 100
          },
          "rollover": {
            "max_age": "30d",
            "max_docs": 5,
            "max_primary_shard_size": "50gb"
          }
        }
      },
      "warm": {
        "min_age": "1m",
        "actions": {
          "readonly": {},
          "shrink": {
            "number_of_shards": 1,
            "allow_write_after_shrink": false
          },
          "forcemerge": {
            "max_num_segments": 1,
            "index_codec": "best_compression"
          },
          "set_priority": {
            "priority": 50
          },
          "migrate": {
            "enabled": false
          },
          "allocate": {
            "number_of_replicas": 0
          }
        }
      },
      "delete": {
        "min_age": "10m",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      }
    }
  }
}

创建模板：
PUT _index_template/test-400-template
{
  "index_patterns": [
    "test-400-*"
  ],
  "priority": 100,
  "template": {
    "mappings": {
      "properties": {
        "title": {
          "type": "keyword"
        }
      }
    },
    "settings": {
      "index": {
        "number_of_replicas": 1,
        "number_of_shards": 5,
        "lifecycle": {
          "name": "policy-test-400",
          "rollover_alias": "test-400"
        }
      }
    }
  },
  "_meta": {
    "description": "Template for test-400 indices with ILM integration"
  }
}

创建索引，以便引用模板
PUT test-400-20260109
{
  "aliases": {
    "test-400": {
      "is_write_index": true
    }
  }
}

写入数据，验证滚动是否生效
PUT test-400/_bulk
{"index":{"_id":1}}{"title":"test 1"}
PUT test-400/_bulk
{"index":{"_id":2}}{"title":"test 2"}
PUT test-400/_bulk
{"index":{"_id":3}}{"title":"test 3"}
PUT test-400/_bulk
{"index":{"_id":4}}{"title":"test 4"}
PUT test-400/_bulk
{"index":{"_id":5}}{"title":"test 5"}
```

