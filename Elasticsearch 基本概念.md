# Elasticsearch 基本概念

#### 本文主要针对Elasticsearch 基本概念进行详细讲解，包括参数默认值、分词器、预处理管道等概念和应用场景进行展开介绍

#### Elasticsearch 默认值
- **索引默认分片数1，索引最大分片数1024**
- **索引默认刷新频率1S**
- **索引默认最大搜索结果1W条**
- **索引默认字段最多1000个**
- **索引默认最大文档大小5M**
- **批量处理最大数据量100MB**
- **nested数据类型默认深度20层，默认字段最多50个**
- **ES默认分词器：standard分词器**

#### Elasticsearch 数据类型
- **基础类型：boolean、keyword、number(integer、long、float、double)、date、text**
- **特殊类型：数组、json、nested、join、flattened**
- **nested定义：将json数据作为一个整体存储和索引，进行CRUD**
- **join：同一索引中通过父子关系实现类似于MySQL join效果，生产环境中严禁使用，应用层多索引关联解决**
- **flattened：将json对象内的多个字段映射为一个字段，避免字段膨胀场景**

- **倒排索引格式：字段内容+【文档ID+文档ID内出现的次数+文档ID内的位置指针】**
- **正排索引(doc_values)：文档ID+字段内容**

#### 索引模板方式：普通模板、组件模板

#### _id生成方式：uuid base64后，共计20位

#### Elasticsearch 基本定义
- **text：可以全文检索，分词字段必须，keyword：聚合、排序、精准匹配**
- **fielddata：text数据类型中早期版本执行聚合、排序、脚本时的解决方案，现已弃用，但仍保留在ES中，默认禁用**
- **_source：存储原始JSON文档数据，默认开启**
- **store：针对字段数据是否存储，通常在_source定义为false时使用**
- **null_value：将null值替换成指定的值**

#### Elasticsearch 分词器
- **分词器类型：中文分词器、英文分词器，主流分词器：IK 分词、HanLP 分词**
- **分词器工作原理：将什么字符过滤掉 char_filter[html_strip]->基本什么文本切分分词 tokenizer["ambiguity" "synonym" ]->分词后再过滤 filter["lowercase"]**
- **分词使用原则：索引时细粒度["analyzer": "ik_max_word"]，搜索时粗粒度["search_analyzer": "ik_smart"]，提高召回率**
- **分词使用语法：analysis：[索引时细粒度{char_filter、tokenizer、filter}+搜索时粗粒度{char_filter、tokenizer、filter}]、tokenizer索引时和细粒度的具体定义**
- **验证分词效果：GET /_analyze {"text": "中华人民共和国","analyzer": "standard"}**
- **ngram：机械切割分词器，不理解语义，应用场景：搜索提示，edge_ngram：前缀切割分词器，不理解语义，应用场景：自动补全**

#### 字段高亮：对文本中搜索词加颜色、字体类的标识，一眼识别出来，应用场景：长文本搜索

#### ETL：抽取(extract)、转换(transform)、加载(load)

#### ingest 预处理管道
- **ingest工作流程：1、创建预处理管道  2、写入任务关联预处理管道  3、写入数据**
- **ingest类型：数据美化(append、enrich)、数据转换(convert、lowercase、trim)、数据过滤(drop、remove)、流水线处理(fail、reroute)、数据和JSON处理(json、sort)**
- **创建预处理管道："PUT _ingest/pipeline/ingest_bbt {"processors":[{"append":{"field":"currency","value":["aaa"]}}]}"**
- **使用预处理管道："PUT test/_doc/1?pipeline=ingest_name"**
- **预处理管道应用范围：创建索引、创建模板、迁移索引、更新索引、写入索引、CCR**

##### 至此，Elasticsearch 基本概念就介绍完成了，请开始你的表演吧！

&nbsp;

**有兴趣的小伙伴，可加联系方式：Vx：+86 183 1026 3857&nbsp;&nbsp;&nbsp;&nbsp;Telegram：+86 189 1127 2119**  
