# 1. ES 简介

## 1. 概述

_**Lucene 和 ES 的关系?**_

```
ES使用Lucene全文搜索引擎
```

_**什么是 Inverted Index?**_

```
将文档分词, 每插入一个文档时维护"词语到文档id列表"的索引结构:
词语a -> [文档id1, 文档id2, ...]
词语b -> [文档id2, 文档id3, ...]
```

**_常见数据库的类型有哪些?_**

```
* 结构型数据库(如MySQL)
* KV数据库(如Redis)
* 文档型数据库(如MongoDB ES)
* 图数据库(如Neoj4)
```

## 2. 术语

**_结构型数据库和文档型数据库术语对比_**

|              | 相关记录的集合 | 单条记录 | 表元信息的定义 | 查询语言                       |
| ------------ | -------------- | -------- | -------------- | ------------------------------ |
| 结构型数据库 | table          | row      | schema         | SQL(Structure Query Language)  |
| 文档型数据库 | index          | document | mapping        | DSL(Domain Specified Language) |

**_如何让一个字段可以使用多种类型的索引?_**

```
使用multi-fields功能, 在mapping中定义字段的子字段

tip: 对于name字段, 一般类型为keyword, 想要支持分词匹配还需要添加类型为text的子字段
```

# 2. ES 查询

## 2.1 基本查询

**_Es 中常用的查询条件有哪些?_**

```
* match_all: 全字段分词查询
* match_query: 单字段分词查询
* multi_match: 多字段分词查询
* term: 单值精准匹配
* terms: 多值精准匹配
* range: 单字段范围匹配
* ids: id精准匹配
* geo_distance: 地理位置半径查询

tip: 实际查询请求为RESTFUL API + DSL的形式, 但查询语法DSL复杂, 一般使用代码构造请求
```

**_什么是 FunctionScoreQuery?_**

```
可以定义相关性得分的再计算函数func:
   最终相关性得分 = func(相关性得分)

tip: function_score_query如果使用随机权重可以实现每次分页查询结果都不同的效果
```

**_ES 会有深分页问题吗?如何解决?_**

```
* ES有深分页问题, 并且相较于单机ES集群的深分页问题更加严重!
>>> 例如"from 1000 limit 10"的查询, 会从集群中的每个节点中都查询前1010条文档再聚合

* 解决方法: 1.业务上限制深分页场景 2.使用游标查询法"search after"

tip: es在系统级别限制了深分页最大offset+limit <= 10000
```

**_如何高亮搜索关键字?_**

```
DSL中使用"highlignt"
```

## 2.2 聚合查询

**_ES 中常用的聚合方式?_**

```
* 桶聚合(bucket agg)
>>> range聚合 term聚合

* 指标聚合(metrix agg)
>>> avg, sum, min, max, ...

* 管道聚合(pipeline agg)
>>> 对聚合结果再进行聚合
```

## 2.3 分词查询

**_text 和 keyword 的区别?_**

```
* text可以使用分词器进行模糊搜索

* keyword是一个完整的文本不会被分词, 仅用于精准匹配
```

**_ES 中常用的分词器(analyzer)IK 分词器有哪两种 tokenizer?_**

```
* 最细粒度分词tokenizer: ik_max_word

* 粗粒度分词tokenizer: ik_smart

tip: 最佳实践: 用户搜索内容分词器(search_analyzer)使用ik_smart, text字段分词(analyzer)使用ik_max_word
```

**_实现分词器的方法?_**

```
* 基于统计(字典法)
* 神经网络

tip: 使用字典法时(如IK分词器), 可以使用停用词过滤无意义的词或扩展词典适配流行语
```

**_分词查询时文档相关性得分是如何计算的?_**

```
* 低版本ES: TF-IDF算法
相关性得分 = 词频 * 逆文档频率

* 高版本: BM25算法
TF-IDF的改进, 相关性得分额外考虑了文本长度

tip: 查询结果默认按照相关性得分排序
```

**_分词器由哪三部分组成?_**

```
* character filter: 对文本进行预处理

* tokenizer: 分词

* tokenizer filter: 对词语进行后处理

tip: 可以通过组合三个部分来实现复杂的分词, 例如常见的type="completion"字段使用的分词器
为{tokenizer = "ik_max_word", tokenizer filter="pinyin"}
```

## 2.4 Suggester

**_ES 中常用的 Suggester 类型有哪些?_**

```
* Term suggester
补全单词

* Phrase suggester
补全短语

* Completion suggester
基于数据结果FST（有限状态转换器）实现

tip: term/phrase可以基于编辑距离修复拼写错误
```

**_如何实现常见的搜索补全功能?_**

```
1. 在mapping中定义type=completion的字段A, 并使用ik+pinyin分词器

2. 定义mapping中的部分字段为"copy_to": A

3. 使用suggest DSL查询
```

# 3. ES 架构

## 3.1 数据同步

**_如何将 mysql 的数据同步到 ES?_**

```
mysql --(binlog)--> 中间件canal --> MQ --> ES

tip: 中间件canal, canal将自己伪装成mysql从库实现监听binlog
```

## 3.2 集群架构

**_ES 集群体现了分布式存储系统的哪两个要点?_**

```
分片和备份
```

**_ES 集群中节点分为哪些类型?_**

```
* 主节点(master)
* 数据节点(data)
* 协调节点(coordinator): 分发查询请求, 聚合查询结果
```
