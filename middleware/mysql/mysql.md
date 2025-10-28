# 1. MySQL 简介

## 1. 关系型数据库理论

**_常见的约束类型有哪些?_**

```
🌟 主键约束Primary Key
🌟 非空约束Not Null
🌟 唯一约束Unique
🌟 外键约束Foreign Key
🌟 默认约束Default
🌟 检查约束Check
```

**_关系型数据库的三范式指的是什么?_**

```
1NF: 字段是原子的
2NF: 只有一个主键
3NF: 字段之间不存在传递性依赖, 都必须直接依赖主键

🌙 有时不一定严格遵循三范式, 为了加快查询速度冗余存储数据
```

**_什么是视图(View)和存储过程(Stroed Procudure)?_**

```
🌟视图是使用DQL语句定义的一张逻辑上的表(CREATE VIEW)
🌟存储过程=数据库函数(CREATE PROCEDURE)
```


## 2. SQL 查询

**_DDL/DML/DQL/DCL 的区别?_**

```
🌟 DDL(Database define language): 如create/drop/alter table
🌟 DML(Database manipulate language): 如insert/delete/update table
🌟 DQL(Datebase query language): 如select table
🌟 DCL(Datebase control language): 如commit/rollback grant/revoke
```

**_SELECT 语句的语法?_**

```sql
/* 普通查询 */
SELECT a,b,c
FROM table
WHERE a>1 AND b>2
ORDER BY a ASC,b DESC
LIMIT 0,10

/* 聚合查询 */
SELECT a,count(*)
FROM table
WHERE b > 0
GROUP BY a
HAVING a != 1
```

**_内连接/外连接/全外连接的区别?_**

```
🌟 内连接: 行A和行B符合连接条件 => 行A-行B
🌟 左外连接: 行A与任何行都不符合连接条件 => 行A-NULL
🌟 右外连接: 任何行与行B都不符合连接条件 => NULL-行B
🌟 全外连接: 左外连接+右外连接
```

**_count(🌟)/count(1)/count(字段)的区别?_**

```
🌟 count(*)和count(1)都可以统计总行数, 但数据库一般只对count(*)有性能优化
🌟 count(字段A)只统计字段A NOT NULL的行的数量
```

**_drop/truncate/delete 的区别?_**

```
🌟 drop: 删除表的数据和元信息(如表字段定义,当前自增值)
🌟 truncate: 删除表的数据(无事务)
🌟 delete: 按事务的方式删除满足条件的行
```

**_union 和 union all 的区别?_**

```
union合并两个集合时会去重, union all不会;

🌙 union去重时每一列数据都相同才会判定为相同
```

**_字符集 utfmb3 和 utfmb4 的区别?_**

```
🌟 utfmb3是uft8的子集, 单个字符最大编码长度为3byte
🌟 utfmb4是完整的uft8, 单个字符最大编码长度为4byte, 支持表情字符

🌙 如无特殊要求, 使用utfmb4字符集; 字符比较规则(Collation)使用utf8mb4_unicode_ci
```

***哪些常用的数据类型的存储空间是固定的?***
```
🌟 bigint = 8字节（64bit整数，与显示宽度M无关）
🌟 int = 4字节（32bit整数，与显示宽度M无关）
🌟 tinyint/boolean = 1字节（8bit整数）
🌟 timestamp = 4字节（范围至2038年）

🌙 对于上面的类型, 显示宽度M（如bigint(M)）仅影响ZEROFILL显示，不改变存储空间和数值范围
```

***有常用的不定长的数据类型?***
```
🌟 char(M) = 最多M个字符(M <= 255)
🌟 varchar(M): 最大存储M个字符(M <= 65535)
🌟 datetime(M): 默认M=0精确到秒, M=3时精确到毫秒, 但都占用8字节空间
🌟 text：存储大文本, 最大约64KB

🌙 上面的类型在DDL时强烈建议显示指定M的值
🌙 注意datetime(0)在保存时会强制截断毫秒部分, 可能导致精度丢失
```

## 3. MySQL Server

**_MySQL服务器采用分层架构是怎样的?_**
```
从上到下分为:
1️⃣ 连接池管理层: 负责连接池管理+鉴权
2️⃣ SQL解析层: 负责解析并优化SQL
3️⃣ 存储引擎层: 负责与文件系统交互, 可以使用不同的存储引擎(MyISAM, InnoDB, Memory)
4️⃣ 文件系统
```

***为什么Innodb的页大小为16KB但MySQL单行长度限制为64KB?***
```
单行长度限制=64KB是在服务层限制的, 页大小=16KB是在存储引擎层的实现

🌙 Innodb在Dynamic行格式下会采用指针和溢出页的方式存储长度大于4KB的varchar/Blob/Tex字段, 但前缀768字节数据会保留在原记录中
🌙 varchar字段的溢出不影响索引
```

## 4. 存储引擎

**_常用的存储引擎有哪些, 它们之间有哪些区别?_**

```
🌟Innodb: 支持事务, 行锁 
🌟MyISAM: 不支持事务, 表锁
🌟Memory: 基于内存而不是文件系统

🌙 存储引擎层向MySQL服务层提报统一的接口
```

# 2. Innodb

## 1. 表的存储

**_Innodb中是如何存储表的?_**
```
🌟 表本身就是一个B+树索引(聚簇索引), B+树的一个节点对应一个16KB大小的页
🌟 聚簇索引使用主键id进行排序
🌟 一个页内部维护一个单链表, 单链表的节点为数据节点或页指针
🌟 聚簇索引的root页常驻内存, 其他页在使用时才加载到内存的buffer pool中
```
![1691476779977](image/mysql/1691476779977.png)

**_为什么在单页中支持对行记录的二分查找?_**
```
页存储了一个Page Diretory指针数组, 每个指针指向行记录开始位置, 便于进行二分查找 
```


## 2. 行格式

**_Innodb中行格式(Raw Format)有哪些?_**
```
有4种行格式: Compact, Redundant, Compressed, Dynamic

🌙 默认行格式为Dynamic, 一般业务场景也推荐使用Dynamic
```


**_Dynamic行格式中每行的元数据有哪些?_**
```
🌟 roll_ptr: 行指针, 用于 mvcc
🌟 row_id: 自动生成的主键id
🌟 trx_id: 事务id, 用于 mvcc
🌟 null列表
🌟 变长字段长度列表
```

## 3. 索引

**_Innodb为什么使用 B+树作为索引, 而不使用B树?_**
```
B+树索引可以支持范围查询

🌙 B+数可以多分叉, 最终保证树的高度不会太高(一般小于等于3层, 一个页大小为16KB可以存储约1000条记录, 3层的B+树可以存储10^6条记录), 树每多一层就需要多一次磁盘IO
```

**_什么是聚簇索引/二级索引/联合索引?_**
```
🌟 聚簇索引(clustered index): 以id建立的索引, 叶节点存储全部字段
🌟 二级索引(secondary index): 以字段A排序的索引, 叶节点存储(A, id)
🌟 联合索引: 以多个字段(A, B)排序的索引, 叶节点存储(A, B, id)

🌙 聚簇索引就是表本身
```

**_什么是最左匹配原则?_**
```
要使用联合索引时, 作为查询条件的字段必须覆盖联合索引的左前缀
```

**_什么是回表?_**
```
"select * from table where ctime > 10"执行时会:
1️⃣ 查二级索引"ctime"获取id
2️⃣ 按id查询聚簇索引获取全部字段(回表)

🌙 如果二级索引就包含所有查询需要的字段(如上面使用Select id,ctime 而不是 Select *), 会直接返回不用回表(又称覆盖索引)
🌙 除了覆盖索引外, 二级索引还可以支持排序和过滤, 这种将操作提前到二级索引而不是聚簇索引的方式被称为"索引下沉", 可以减少回表次数
```

**_索引什么情况下会失效?_**
```
🌟联合索引没有遵守最左匹配原则
🌟查询条件中对索引字段进行计算/函数/类型转换
🌟查询条件为!= / NOT NULL / like '%abc'
```

**_什么字段适合建立索引?_**
```
🌟 前缀区分度高的字段
🌟 频繁作为查询条件或排序条件或分组条件的字段

🌙 索引本质上是空间换时间, 过多索引会影响DML语句性能
🌙 对姓名等字段reverse后再建立索引
```

**_为什么建议使用自增主键?_**
```
在插入数据时使用自增主键可以减少聚簇索引的页分裂

🌙页分裂时会阻塞其他事务对页的访问, 并且分裂后的两个页利用率只有50%
🌙如果删除一行记录, 这个记录所在页会有"空洞"
```

**_深分页为什么慢?_**

```sql
SELECT * FROM user ORDER BY ctime LIMIT 1000000,10
上面的这行SQL会先查二级索引"ctime", 会先遍历前1000000行记录, 因为每个B+树索引的非叶节点不存储行数, 仅存储最大最小值和子节点指针!

🌙 RR级别下, 遍历ctime索引时会加Gap锁
🌙 注意由于MySQL的流式设计, 这里即使没有Where条件, 在遍历ctime索引时每遍历一个(ctime, id)都会立刻回表, 总计回表1000010次, 不会发生索引下沉
```

**_深分页问题如何解决?_**

```sql
游标法, 上面的SQL可以改为:
SELECT * FROM user WHERE ctime > 上次查询结果最大的ctime ORDER BY ctime LIMIT 10

🌙 遇到深分页问题, 尽量使用游标查询(或说服产品使用瀑布流)
```

# 3. 事务和锁

## 1. 事务简介
**_什么是事务?_**
```
要么全部成功, 要么全部失败的一组动作

例如: A向B转账时, A账户扣款和B账户加款两个动作必须为事务
```
**_数据库事务的ACID特性?_**

```
A=原子性: 要么全部执行, 要么全部不执行
C=一致性: 事务执行前后数据库保持一致性(不破坏数据的约束)
I=隔离性: 不同事务之间的执行互不影响
D=持久性: 事务对数据库的影响是永久的
```

**_事务隔离级别有哪些? 每个隔离级别会导致的问题?_**

```
RU:  一个事务内可以读取到还未提交的事务进行的更改, 会有脏读问题
RC: 一个事务内只能读取到已经提交的事务的更改, 会有不可重复读问题
RR:  一个事务期间数据的值在事务开始和结束时一致, 会有幻读问题
SERIAL: 事务之间串行执行

🌙 一般/默认使用RR级别, Mysql使用MVCC+Gap锁在RR级别下可以解决幻读问题
```

## 2. MVCC

**_RC/RR隔离级别下的MVCC原理?_**
```
🌟 MVCC依赖于行头中的roll_ptr指针和trx_id实现
🌟 rollback_ptr指向undo log中的历史版本, 组成历史版本链表
🌟 每次执行事务时, 根据当前事务id, 判断事务可见的最新历史版本

🌙 undo log在内存中有buffer page, 可以快速获取历史版本记录(由于undo log是逻辑日志, 需要推导出历史版本)
🌙 MVCC是快照读, 读的历史版本数据, 不一定是最新的数据, 但从事务的角度看是数据一致的
```

## 3. 锁

***锁的作用?***
```
在多个事务并发执行时, 保证事务的隔离性（防止脏读、不可重复读、幻读）

🌙 加锁的对象是索引上的记录(或记录间的间隔), 聚簇索引和二级索引上都会加锁(多个二级索引加锁顺序和定义索引顺序一致)
```

**_锁的分类有哪些?_**
```
🌟 互斥锁(X锁): SELECT ... FOR UPDATE / UPDATE / DELETE / INSERT
🌟 共享锁(S锁): SELECT ... IN SHARE MODE

🌟 行锁
🌟 表锁: ALTER TABLE
🌟 表意向锁(IX/IS锁): 申请行锁前需要先申请表意向锁, 不阻塞仅标记

🌟 自增锁: 自增主键递增时需要申请自增锁

🌟 记录锁(Record Lock): 修改/删除记录前先申请记录锁
🌟 间隙锁(Gap Lock): 插入数据前需要申请间隙锁, 解决了当前读的幻读问题
🌟 临键锁(Next key Lock): 记录锁 + 记录旁边的间隙锁, 仅RR隔离级别才加临键锁

🌙 如果SELECT不指定加S锁还是X锁, 那么使用的是MVCC进行快照读
```

## 4. 日志

_讲一下redo log是如何生效的?_

```
Write Ahead Log(WAL)思想:

1️⃣ 修改buffer pool中的页
2️⃣ 将修改过程记录到内存中的redo log buffer页中
3️⃣ 事务提交时, 将redo log buffer刷新到redo log文件(同步刷盘，即fsync), 每个日志都有一个自增唯一LSN
4️⃣ 定时(Checkpoint时)将buffer pool中的脏数据页刷新到磁盘中(同步刷盘，即fsync), 刷新后移动“checkpoint LSN”(更新对应文件)

🌙 写redo log时是顺序写, 速度很快
🌙 在数据库crash后的重启阶段, 会先执行“checkpoint LSN”后的redo log记录, 恢复内存中丢失的脏页
🌙 如果第4️⃣步刷盘后宕机, 即使“checkpoint LSN”没有移动, 由于redo log的幂等性(只记录物理变动)重复执行也没有问题
```

_为什么有了redo log还需要undo log?_
```
undo log提供了对事务回滚和MVCC的支持, 保证了事务的原子性(要么全部都不执行)

例如:
一个事务修改了所有页面导致buffer pool中的部分脏页面必须在事务提交前刷新到磁盘上(steal策略), 此时如果没有undo log, 就会导致未提交事务的却修改了DB, 如果事务在之后提交不成功就会破坏事务的原子性

🌙 undo log是逻辑日志, 只记录SQL操作的反操作, MVCC快照读时需要通过历史版本链上的所有反操作推导历史版本
```

_一次完整的SQL修改过程是怎样的?_
```
假设表有二级索引且修改时用到了二级索引
1️⃣ 二级索引加锁
2️⃣ 聚簇索引加锁

3️⃣ 写入undo log buffer页
4️⃣ 修改聚簇索引buffer页
5️⃣ 修改二级索引buffer页

6️⃣ 写入redo log buffer

8️⃣ 事务提交（Commit）时, redo log刷盘, 释放所有锁

🌙 undo log本身通过redo log机制确保事务提交后旧版本数据的持久性(redo log记录了所有数据页和undo log页的变更)
🌙 后台线程异步将脏buffer页进行刷盘(check point), 并推进check point LSN
🌙 如果内存不足, 即使事务未提交redo log buffer也可以刷盘, 但日志会标记事务未提交, 重启时先使用redo log还原undo log, 然后使用undo log回滚事务
```

_什么是Check Point?_
```
Check Point的目的是将脏页刷到磁盘中并移动check point LSN, 避免Redo Log文件溢出, 每次使用Redo log还原时都会从check point LSN开始

🌙 Redo log文件为环形, 如果Checkpoint LSN推进速度 < Redo Log生成速度会导致文件溢出
🌙 Check point期间会对要刷盘的脏页加锁
🌙 默认每秒进行一次Check Point
```

# 4. 调优

## 1. Explain

**_EXPLAIN 可以做什么?_**

Explain+DQL 语句可以查看执行计划, 执行计划中重要的字段有:

```
🌟 type: 查询方式
🌟 key: 使用到的索引
🌟 rows: 预估需要扫描行数
🌟 filtered: WHERE子句过滤了行数占比(%)

🌙 使用explain只能查看执行计划, 并没有实际执行SQL!
```

**_查询方式有哪些?_**

```
🌟 全表遍历: all
🌟 索引上全量遍历: index
🌟 索引范围扫描: range
🌟 索引的等值查询: ref
```

## 2. 分库分表
***分库分表有哪些方式?***
```
🌟 水平分表: 数据库A存储记录record1; 数据库B存储记录record2, record3;
🌟 垂直分表: 数据库A存储字段record1.a, record1.b; 数据库B存储字段record1.c, record1.d;

🌙 分库分表用于提升性能, 当单表预估行数>2000W时可以考虑分库分表
🌙 多数情况下, 优先考虑对无用数据进行归档, 而不是分表
```

***分库分表需要处理的难点?***
```
🌟 id不能使用数据库自增id: 需要使用分布式自增id
🌟 特殊SQL(self-join)无法支持, 聚合查询需要广播查询再由分片中间件聚合(数据量过大时有性能风险)
🌟 多库之间的分布式事务: 一般使用柔性事务(如TCC, 最大努力通知)
```

***如何生成分布式自增ID?***
```
🌟 Snowflake ID: 雪花ID = 时间戳+机器节点ID+机器生成序列号, 其中机器ID使用Zookeeper维护
优点: 数据更安全, 无法通过ID判断记录的个数
缺点: 系统时间回拨时可能会导致ID重复, 可以本地缓存上一次发号的时间戳判断时间戳是否回拨, 如果回拨则不可用

🌟 Leaf ID: 一次性获取一段ID并缓存到本地, 使用数据库存储段长和MaxId
缺点: 可以通过ID大致判断记录个数

🌙 Snowflake和Leaf都不能保证严格递增, 只能保证趋势递增
```

# 5. MySQL集群

***MySQL的主从架构是什么样的?***
```
采用主从架构, 主服务器负责响应读/写请求, 从服务器只能响应读请求; 主从服务器之间通过binlog异步复制

🌙 读请求具体读主还是读从取决于中间层/客户端的路由策略, 但一般而言是读从
```

