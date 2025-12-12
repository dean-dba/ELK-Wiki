# Elastic Stack 基本概念

#### 学习和了解Elastic Stack的基本概念，可以帮助我们更好的了解Elastic Stack设计理念、应用场景，以及如何更好的部署、规划、使用Elastic Stack，真正做到"知己知彼，百战不殆"

#### Elastic Stack产品矩阵
- **Elasticsearch：分布式数据存储和搜索引擎，可处理索引、查询和分析，是Elastic Stack的核心和发动机**
- **Kibana：仪表盘、可视化和管理工具的可视化界面**
- **Logstash：数据摄取和转换引擎，通常用于更复杂的 ETL（提取、转换、加载）管道**
- **Elastic Agent：一个轻量级的数据传输器，用于收集数据并将其转发到 Elasticsearch，现已替代Beats**
- **Beats：Elastic Agent没有出来之前的主流数据传输器，主要有Filebeat、Metricbeat、Auditbeat、Heartbeat、Packetbeat、Winlogbeat**
- **APM(应用性能监控)：它能够实时监控软件服务和应用程序，收集关于传入请求、数据库查询、缓存调用、外部 HTTP 请求等的详细性能信息，从而帮助您快速定位并修复性能问题**
- **Elasticsearch Hadoop：它是一个与Hadoop、Elasticsearch之间进行数据交互的一种工具**


#### Elasticsearch 基本概念
- **集群：一组由不同角色的Elasticsearch 节点组成的集合**
- **节点：不同角色的Elasticsearch 节点，一般生产集群最少3节点**
- **分片：存放索引数据的子集，比如一个索引有3个分片，那么每个分片存放1/3数据**
- **副本：分片的副本，分片高可用、只读数据的来源**
- **索引：存储、检索、组织数据的逻辑容器**
- **文档：Elasticsearch的基本存储单元，类似于MYSQL的行**
- **字段：Elasticsearch的最小数据单元，类似于MYSQL的列**
- **映射：Elasticsearch的存储数据结构，类似于MYSQL的表结构**
- **分词：构建倒排索引的基本，比如："我是中国人"，分词："我"、"是"、"中国"、"人"**
- **JSON：Elasticsearch的存储、返回格式**

#### Elasticsearch、Kibana查询语言(数据写入/更新/删除操作须通过PUT、POST _bulk、POST _update_by_query等接口操作)
- **Query DSL：原生的JSON风格语言，支持复杂查询**
- **ES|QL：用于过滤、转换和分析数据的一种管道查询语言，支持JOIN、支持跨集群**
- **EQL：基于时间序列、事件序列（例如日志、指标和跟踪）的查询语言**
- **Elasticsearch SQL：类似于SQL语法的查询，底层自动转换成Query DSL**
- **KQL(Kibana Query Language)：基于Kibana UI的查询语言，可以通过Kibana UI界面操作自动生成查询、过滤数据**
- **Lucene query syntax：基于Apache Lucene的基本搜索、简单过滤的原始查询语法**

#### Elasticsearch 常用数据类型：keyword、Number、Date、Text、Boolean、Object、Geo_point

#### Elasticsearch CRUD常用操作语法("POST _bulk API"是高效率写入的核心，可以在一个请求中混合执行索引、创建、更新、删除等多种操作)
- **查询：match、multi_match、term、terms、range、exists、bool、filter、geo_distance、geo_bounding_box等**
- **插入："PUT /index/_doc/1"、"POST _bulk"**
- **更新："POST /<index>/_update/<id>"、"POST /<index>/_update_by_query"、"POST _bulk"**
- **删除："DELETE /<index>/_doc/<id>"、"POST /<index>/_delete_by_query"、"POST _bulk"**

# Elasticsearch 倒排索引结构
![Elasticsearch 倒排索引数据结构](images/倒排索引数据结构.png)


