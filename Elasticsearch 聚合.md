# Elasticsearch 聚合

#### 通过Elasticsearch提供的聚合函数可以轻松实现数据分析、数据挖掘的目的，主要应用于用户行为分析、电商分析、日志分析等场景，功能强大且灵活高效

#### 聚合函数
- **数字类型：sum、avg、min、max、count、stats、range、histogram、percentiles、cardinality**
- **keyword类型：terms、multi_terms、filters、significant_terms、diversified_sampler**
- **date类型：min、max、date_range、date_histogram、extended_stats**
- **地理类型：geo_bounds、geo_centroid、geo_distance、geohash_grid**

#### 聚合类型
- **分桶聚合(类似于MySQL group by，支持聚合所有数据类型)：terms(分桶聚合)、range(范围聚合)、histogram(间隔聚合)、date_histogram(时间间隔聚合)、date_range(自定义时间范围间隔聚合)、filters(满足给定条件聚合)**
- **指标聚合(类似于MySQL sum、avg、max，支持数字、date类型)：avg、sum、max、min、stats、top hits(求各外层桶的详情)、cardinality(去重)、value count(计数)**
- **管道子聚合(类似于MySQL 窗口函数，依赖于其它聚合输出)：兄弟聚合：[max_bucket(获取外层聚合最大值的桶，并输出桶值、键)、min_bucket(与max_bucket相反)、stats_bucket(获取外层聚合统计结果)、sum_bucket(获取外层聚合求和结果)]、父子聚合：[bucket_selector(对聚合结果进行再筛选、运算)、bucket_sort(使用聚合结果的任意字段进行排序，并输出排序后的桶列表)、bucket_script(根据聚合结果进行脚本运算，以生成新的聚合结果)]**
- **composite(组合聚合)：对复杂查询进行多维度聚合查询，不能与管道子聚合混合使用**

分桶聚合之terms
```
POST test_100/_search
{
  "size": 0,
  "aggs": {
    "hole_terms_agg": {
      "terms": {
        "field": "has_hole"
      }
    }
  }
}
```

指标聚合之max
```
POST test_100/_search
{
  "size": 0,
  "aggs": {
    "max_value_aggs": {
      "max": {
        "field": "size"
      }
    }
  }
}
```

管道子聚合之max_bucket
```
POST test_100/_search
{
  "size": 0,
  "aggs": {
    "hole_terms_agg": {
      "terms": {
        "field": "has_hole"
      },
      "aggs": {
        "max_value_aggs": {
          "max": {
            "field": "size"
          }
        }
      }
    },
    "max_hole_color_aggs": {
      "max_bucket": {
        "buckets_path": "hole_terms_agg>max_value_aggs"
      }
    }
  }
}
```

组合聚合之[terms+histogram]
```
POST test_200/_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 5,
        "sources": [
          {
            "brand_terms": {
              "terms": {
                "field": "brand",
                "order": "asc"
              }
            }
          },
          {
            "prices_histogram": {
              "histogram": {
                "field": "price",
                "interval": 50,
                "order": "asc"
              }
            }
          }
        ],
        "after": {
          "brand_terms": "brand_a",
          "prices_histogram": 500
        }
      }
    }
  }
}
```

分桶聚合之cardinality
```
POST test_200/_search
{
  "size": 0,
  "aggs": {
    "brand_count": {
      "cardinality": {
        "field": "brand"
      }
    }
  }
}
```

#### 去重显示：聚合函数+top_hits+size：基于评分显示、collapse+inner_hits+collapse：性能优于top_hits

##### 至此，Elasticsearch 聚合就介绍完成了，请开始你的表演吧！

&nbsp;

**有兴趣的小伙伴，可加联系方式：Telegram：@dean_code**  
