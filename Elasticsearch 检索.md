# Elasticsearch 检索

#### Elasticsearch 针对不同数据类型有不同的数据检索方式，每种检索方式都有不同的应用场景和特点

- **"profile": true类似于mysql explain**
- **ES 默认评分算法：BM25(默认：k1(词频)=1.2, b(文档长度归一化)=0.75)**

#### 自定义评分方式：内置函数评分(90%场景使用field_value_factor)、自定义脚本评分、自定义BM25评分、两阶段评分
    
#### 检索模板：跟索引模板功能类似，模板引擎：mustache

text数据类型检索，应用场景：文本内容搜索、模糊匹配、相关性排序(默认按_score从高到低排序)，特点：不区分大小写、自动分词、计算相关性、词之间默认是"或"关系
```
match：标准全文搜索(自动分词)，示例：{"match": {"field": "elasticsearch 教程"}}
multi_match[best_fields(默认，按搜索词取字段最高分并排序)、most_fields、cross_fields]：多字段搜索，示例：{"multi_match": {"query": "elasticsearch", "fields": ["field_1^2", "field_2"]}}
match_phrase：短语匹配(保持词序)，示例：{"match_phrase": {"field": "elasticsearch 教程"}}
match_phrase_prefix：短语匹配+最后一个词项前缀匹配，示例：{"match_phrase_prefix": {"field": "elasticsearch 教程"}}
match_bool_prefix：所有词项都必须出现，最后一个词可前缀匹配，不要求顺序和连续，示例：{"match_bool_prefix": {"field": "elastic sea"}}
query_string：支持复杂查询，但性能差、安全注入风险高，示例：{"query_string": {"default_field": "field_name", "query": "(elasticsearch OR kibana) AND 教程"}}
simple_query_string：query_string的简单查询，不支持模糊匹配、正则，不检查语法是否正确，示例：{"simple_query_string": {"default_field": "field_name", "query": "(elasticsearch OR kibana) AND 教程"}}
```

非text数据类型的精确检索，应用场景：精确值匹配、状态过滤、分类查询，语法特点：严格区分大小写、不分词，性能：term、terms优于match、multi_match
```
term：精确值匹配（单值），示例：{"term": {"field": "elasticsearch"}}
terms：多值任一值精确匹配，示例：{"terms": {"field": ["active", "pending"]}}
range、date_range：范围查询，示例：{"range": {"field": {"gte": 100, "lte": 500}}}
exists：字段存在性检查，返回字段非null、字段不存在的对应文档，示例：{"exists": {"field": "field_name"}}
prefix：前缀匹配，只支持keyword数据类型，示例：{"prefix": {"field": "elas"}}
wildcard：通配符匹配，只支持keyword数据类型，示例：{"wildcard": {"field": "*search*"}}
terms_set：有数量要求的多值精确匹配，只支持keyword数据类型，示例："terms_set": { "field": {"terms": ["词项1", "词项2", "词项3"],"minimum_should_match": 2 } }
fuzzy：纠错容错检索，大于5个字符时使用auto默认允许2个字符错误，示例："match": { "field": { "query": "词项1","fuzziness": "AUTO" } }
regexp：正则匹配检索，只支持keyword数据类型，非必要不使用，性能差，示例：{  "regexp": {  "sku": {   "value": "PROD.*"    }  } }
ids：专为文档ID检索，示例："ids": {"values": ["1", "2", "100"]}
```

组合检索，应用场景：复杂查询、多条件组合，语法特点：minimum_should_match(默认): 0，constant_score(默认)："boost": 1.0，filter可以使用缓存
```
bool：组合检索，示例：{"bool": {"must": [匹配条件], "should": [可选条件], "must_not": [排除条件], "filter": [过滤条件]}}
constant_score：固定分数查询，跳过内置评分器，效率最高，示例：{"constant_score": {"filter": {...}, "boost": 1.0}}
```

地理检索：应用场景：位置搜索、距离计算、地理围栏
```
geo_distance：圆形区域搜索，示例：{"geo_distance": {"distance": "10km", "location": {...}}}
geo_bounding_box：矩形区域搜索，示例：{"geo_bounding_box": {"location": {"top_left": {...}, "bottom_right": {...}}}}
geo_polygon：多边形区域搜索，示例：{"geo_polygon": {"location": {"points": [{lat, lon}, ...]}}}
```

嵌套检索，应用场景：嵌套文档一对多查询、数组内对象搜索
```
nested：嵌套结构检索，示例：{"nested": {"path": "comments", "query": {...}}}
has_child：父子文档关联查询，示例：{"has_child": {"type": "comment", "query": {"term": {"status": "approved"}}}}
has_parent：子文档关联查询，示例：{"has_parent": {"parent_type": "article", "query": {"term": {"category": "technology"}}}}
```

特殊检索，应用场景：特殊需求、高级功能
```
more_like_this：相似文档推荐，示例：{"more_like_this": {"fields": ["title", "content"], "like": "Elasticsearch 搜索教程", "min_term_freq": 1}}
script：自定义脚本查询，示例：{"script": {"script": {"source": "doc['sales'].value > params.min_sales", "params": {"min_sales": 100}}}}
```

高亮、排序、分页
```
高亮：检索结果突出显示，应用场景：内容检索、日志分析，语法特点：fragment_size(每个片段高亮字符长度：100)、number_of_fragments(高亮片段数量：5)、高亮标签：<em>，示例：{ "query": { /* 查询条件 */ }, "highlight": { "fields": { "高亮字段名": { /* 具体配置 */ } }} }
排序：按某一字段排序，禁用内置评分，示例：{ "query": { "match": { "fields": "electronics" } },  "sort": [  { "price": { "order": "asc" } }, { "rating": { "order": "desc" } } ] }
分页类型："from:0,size:10"、search_after、scroll、PIT(Point in Time)+search_after

"from:0,size:10"：浅分页
示例：{ "from": 0, "size": 10,  "query": {...}}
应用场景：小数据量最大小于10000条(max_result_windows默认10000条)、支持随机分页
语法特点："from:1000,size:10"：需要加载1010条文档，没有类似于MYSQL ICP功能

scroll：深度分页，基于快照非实时，不能随机分页
示例："POST /index_name/_search?scroll=1m { "size": 100, "query": {...}, "sort": [ {"_shard_doc": "asc"}  // 特殊排序，高效遍历 ] }"-->"POST /_search/scroll {"scroll": "1m","scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVY..."}"-->"DELETE /_search/scroll { "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZ2twU2h1NnJYc1F6VDRpQQ=="}"
应用场景：数据导出、全量处理
语法特点："scroll=1m"：快照保持时间，类似于ORACLE 游标

search_after：深度分页，实时数据，不能随机分页
示例：GET /index/_search  {   "size": 10,   "query": {...},   "sort": [     {"field1": "asc"},     {"_id": "asc"}  // 必须包含唯一字段保证确定性   ],   "search_after": [last_sort_value1, last_id] } 
应用场景：深度分页
语法特点：必须包括"_id"字段，使用上一页最后一条排序值作为下一页起点

PIT(Point in Time)+search_after：数据一致性的search_after，不能随机分页
示例：POST /products/_pit?keep_alive=1m"-->"GET /_search {   "size": 10,   "query": {...},   "pit": {     "id": "pit_id",     "keep_alive": "1m"   },   "sort": [     {"_shard_doc": "asc"}  // 特殊排序，高效遍历   ],   "search_after": [125]  // 上一页的_shard_doc值 }
应用场景：数据一致性的深度分页
语法特点：数据一致，全局有序
```

##### 至此，Elasticsearch 检索就介绍完成了，请开始你的表演吧！

&nbsp;

**有兴趣的小伙伴，可加联系方式：Telegram：@dean_code**  

