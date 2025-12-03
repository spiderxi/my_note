# 1. ES 简介

## 1. 概述

_**Lucene 和 ES 的关系?**_

```
ES底层使用Lucene全文搜索引擎
```

_**什么是 Inverted Index?**_

```
将文档分词, 每插入一个文档时维护"词语到文档id列表"的索引结构:
词语a -> [文档id1, 文档id2, ...]
词语b -> [文档id2, 文档id3, ...]
```

**_常见数据库的类型有哪些?_**

```
🌟 结构型数据库(如MySQL)
🌟 KV数据库(如Redis)
🌟 文档型数据库(如MongoDB)
🌟 图数据库(如Neoj4)
🌟 搜索引擎(如ES)
🌟 时序数据库
```

## 2. 术语

**_结构型数据库和文档型数据库术语对比_**

|              | 相关记录的集合 | 单条记录 | 表元信息的定义 | 查询语言                       |
| ------------ | -------------- | -------- | -------------- | ------------------------------ |
| 结构型数据库 | table          | row      | schema         | SQL(Structure Query Language)  |
| 文档型数据库 | index          | document | mapping        | DSL(Domain Specified Language) |

**_如何让一个字段可以使用多种类型的索引方式?_**

```
使用multi-fields功能, 在mapping中定义字段的子字段

🌙 对于name字段, 一般类型为keyword, 想要支持分词匹配还需要添加类型为text的子字段, 查询时使用match("name.子字段名称")的方式
```

***常用的字段类型有哪些?***
```
text, keyword, long, integer, double, float, date, boolean

🌙 匹配查询时使用倒排索引
🌙 范围查询时使用BKD树索引
```

***什么是Nested字段?***
```
Nested字段是文档嵌套类型, 类型相当于List<Document>
```

***ES查询结果中有哪些重要字段?***
```
{
    // 耗时
    "took": 1,
    // 每个分片查询情况
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 总数,
        "max_score": 文档最大相关性得分,
        "hits": [
            {
                "_index": 索引名称,
                //  每个文档的分类, 高版本7.0后被废弃
                "_type": document类型,
                "_id": 主键id,
                "_score": 相关性得分,
                "_source": 文档内容
            }
        ]
    }
}
```

***ES 默认的刷新策略是什么?***
```
默认每秒刷新一次(写入文档后1s才能查询到变更, 按id写入后可立刻查到)
```

***ES并发控制方式?***
```
ES使用乐观锁(版本号)控制并发修改

🌟 推荐按id更新
🌟 更新方式有两种: update为增量更新(局部更新), index方式为全量更新
```

# 2. ES 查询

## 2.1 基本查询

**_ES中常用的查询条件有哪些?_**

```
🌟 bool: 用于其他查询条件的逻辑复合
🌟 match_all: 全字段分词查询
🌟 match_query: 单字段分词查询
🌟 multi_match: 多字段分词查询
🌟 term: 单值精准匹配
🌟 terms: 多值精准匹配
🌟 range: 单字段范围匹配
🌟 ids: id精准匹配
🌟 geo_distance: 地理位置半径查询
```

***bool.query中must和filter的区别?***
```
两者都表示AND的逻辑关系, 区别在于must的字段会计算相关性得分, filter不会(score=0)

🌙 filter因为不计算评分, 所以速度会更快
```

***bool.should查询如何指定最少满足条件的个数?***
```
设置minimum_should_match, 默认值为1

🌙 如果没有should子句, minimum_should_match参数无效
```

**_什么是 FunctionScoreQuery?_**

```
可以自定义相关性得分的再计算函数func: 最终相关性得分 = func(相关性得分)

🌙function_score_query如果使用随机权重可以实现每次分页查询结果都不同的效果
🌙如果不指定任何排序规则, 默认按相关性得分排序
```

***介绍一下ES分页查询时的Query Phase和Fetch Phase?***
```
ES的查询分为Query Phase和Fetch Phase:
🌟 Query Phase: 协调节点将查询路由到多个数据节点数, 据节点根据倒排索引(分词查询和匹配查询)/BKD树索引(范围查询)过滤出"_id", 并根据bool逻辑取并集或交集, 将过滤后的文档部分字段("_id"和"_score")返回协调节点

🌟 Fetch Phase: 协调节点进行归并排序, 确定结果页后根据"_id"从数据节点获取完整数据

🌙 Query Phase时数据节点会进行TopK排序, Fetch Phase时协调节点仅进行K路归并排序
```


**_ES 会有深分页问题吗?如何解决?_**

```
🌟 ES有深分页问题, 并且相较于单机ES集群的深分页问题更加严重!
例如: "from 1000 limit 10"的查询, 会从集群中的每个节点中都查询前1010条文档再聚合

🌟 解决方法: 业务上限制深分页场景(瀑布流);  使用游标查询法"search after"

🌙 ES在系统级别限制了深分页最大offset+limit <= 10000
```

***ES 使用Search After查询时需要注意什么?和scroll查询的区别?***
```
🌟search_after查询本身无状态, 不维护查询上下文, 所以sort字段应当具有唯一性
🌟scroll查询有状态, 维护查询上下文
```

**_如何高亮搜索关键字?_**

```
DSL中使用"highlignt"
```

## 2.2 聚合查询

**_ES 中常用的聚合方式?_**

```
🌟 桶聚合(bucket agg)
例如: range聚合 term聚合

🌟 指标聚合(metrix agg)
例如 avg, sum, min, max, ...
```

## 2.3 分词查询

**_text 和 keyword 的区别?_**

```
🌟 text在可以使用分词器进行match查询
🌟 keyword是一个完整的文本不会被分词, 仅用于terms匹配
```

**_ES 中常用的分词器(analyzer)IK 分词器有哪两种 tokenizer?_**

```
🌟 最细粒度分词tokenizer: ik_max_word
"中华人民共和国的位置" => "中华人民共和国"+"中华人民"+"中华"+"人民"+"共和国"+"位置"

🌟 粗粒度分词tokenizer: ik_smart
"中华人民共和国的位置" => "中华人民共和国"+"位置"

🌙最佳实践: 用户搜索内容分词器(search_analyzer)使用ik_smart, text字段分词(analyzer)使用ik_max_word
```

**_实现分词器的方法?_**

```
🌟 基于统计(字典法)
🌟 神经网络

🌙 使用字典法时(如IK分词器), 可以使用停用词过滤无意义的词或扩展词典适配流行语
```

**_分词查询时文档相关性得分是如何计算的?_**

```
🌟 低版本ES: TF-IDF算法
相关性得分 = 词频 * 逆文档频率

🌟 高版本: BM25算法
TF-IDF的改进, 相关性得分额外考虑了文本长度

🌙查询结果默认按照相关性得分排序
```

**_分词器由哪三部分组成?_**

```
1️⃣ character filter: 对文本进行预处理
2️⃣ tokenizer: 分词
3️⃣ tokenizer filter: 对词语进行后处理

🌙 可以通过组合三个部分来实现复杂的分词, 例如类型为"completion"的字段(支持自动补全)使用的分词器
为{tokenizer = "ik_max_word", tokenizer filter="pinyin"}
```

## 2.4 Suggest查询

**_ES 中常用的 Suggester 类型有哪些?_**

```
🌟 Term suggester
补全单词

🌟 Phrase suggester
补全短语

🌟 Completion suggester
基于数据结果FST（有限状态转换器）实现

🌙 term/phrase可以基于编辑距离修复拼写错误
```

**_如何实现常见的搜索补全功能?_**

```
1️⃣ 在mapping中定义type=completion的字段A, 并使用ik+pinyin分词器
2️⃣ 定义mapping中的部分字段为"copy_to": A
3️⃣ 使用suggest DSL查询
```

# 3. ES 架构

## 3.1 数据同步

**_如何将 mysql 的数据同步到 ES?_**

```
mysql --(binlog)--> 中间件canal --> MQ --> ES

🌙 canal将自己伪装成mysql从库实现监听binlog
```

## 3.2 集群架构

**_ES 集群体现了分布式存储系统的哪两个要点?_**

```
分片和备份
```

**_ES 集群中节点分为哪些类型?_**

```
🌟 主节点(master)
🌟 数据节点(data)
🌟 协调节点(coordinator): 分发查询请求, 聚合查询结果
```
