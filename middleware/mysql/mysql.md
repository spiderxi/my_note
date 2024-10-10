# 1. MySQL 简介

## 1. 关系型数据库理论

**_(constraint)类型有哪些?_**

```
* 主键约束Primary Key
* 非空约束Not Null
* 唯一约束Unique
* 外键约束Foreign Key
* 默认约束Default
* 检查约束Check
```

**_关系型数据库的三范式指的是什么?_**

```
1NF: 字段是原子的
2NF: 只有一个主键
3NF: 字段之间不存在传递性依赖, 都必须直接依赖主键

tip: 有时不一定严格遵循三范式, 为了加快查询速度存储冗余数据
```

**_什么是视图(View)?_**

视图是使用 SQL 查询语句定义的一张逻辑上的表

```sql
-- 创建视图
CREATE VIEW view_name AS
SELECT * FROM table WHERE ...
```

**_什么是存储过程(Stroed Procudure)?_**

一个存储过程相当于存储在数据库的一个函数

```sql
-- 创建存储过程
CREATE PROCEDURE get_customer_orders(IN customer_id INT)
BEGIN
    SELECT order_id, order_date, total_amount
    FROM orders
    WHERE customer_id = customer_id;
END;
```

## 2. SQL 查询

**_DDL/DML/DQL/DCL 的区别?_**

```
* DDL(Database define language): 如create/drop/alter table

* DML(Database manipulate language): 如insert/delete/update table

* DQL(Datebase query language): 如select table

* DCL(Datebase control language): 如commit/rollback grant/revoke
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
* 内连接: 行A和行B符合连接条件 => 行A-行B

* 左外连接: 行A与任何行都不符合连接条件 => 行A-NULL

* 右外连接: 任何行与行B都不符合连接条件 => NULL-行B

* 全外连接: 左外连接+右外连接
```

**_count(🌟)/count(1)/count(字段)的区别?_**

```
>> count(*)和count(1)都可以统计总行数, 但数据库一般只对count(*)有性能优化

>> count(字段A)只统计字段A NOT NULL的行的数量
```

**_深分页为什么慢?_**

```sql
SELECT * FROM user ORDER BY ctime LIMIT 1000000,10
上面的这行SQL会先查二级索引"ctime", 会先遍历前1000000行记录!
```

**_深分页问题如何解决?_**

```sql
游标法, 上面的SQL可以改为:
SELECT * FROM user WHERE ctime > 上次查询结果最大的ctime ORDER BY ctime LIMIT 10

tip: 需要和产品沟通, 让用户使用瀑布流的方式浏览记录
```

**_drop/truncate/delete 的区别?_**

```
* drop: 删除表的数据和元信息, 不可恢复
* truncate: 删除表的数据, 不可恢复
* delete: 满足条件的行, 可恢复
```

**_union 和 union all 的区别?_**

```
union合并两个集合时会去重, union all不会;
```

**_字符集 utfmb3 和 utfmb4 的区别?_**

```
* uftmb3是uft8的子集, 单个字符最大编码长度为3byte, 不支持表情字符

* uftmb4是完整的uft8, 单个字符最大编码长度为4byte, 支持表情字符
```

## 3. MySQL Server

**_mysql 服务器采用分层架构, 具体怎么分的?_**

```
从上到下分为:
连接管理层: 进行连接的安全验证和连接池管理
解析优化层: 查询缓存, 进行词法分析, 语法解析, 通过SQL处理器, 查询优化器生成查询计划
存储引擎层: 执行查询
文件系统: 操作系统的文件系统, 保存了表的物理结果
```

![1691131088320](image/mysql/1691131088320.png)

**_如何对 mysql 服务器进行个性化设置?_**

通过设置系统变量来对服务器进行设置, 系统变量可以通过配置文件设置, 也可以通过指令 `SET GLOBAL 变量名 = 值`设置

可以通过 `SHOW VARIABLES LIKE 'xxx'` 查看系统变量的值

**_如何查看 mysql 服务器运行状态?_**

通过指令查看状态变量 `SHOW STATUS LIKE 'xxx'`

## 4. 存储引擎

**_常用的存储引擎有哪些, 它们之间有哪些区别?_**

常用的存储引擎: Innodb, MyISAM, Memory

Innodb 和 MyISAM 的区别:

```
* Innodb支持事务, MyISAM不支持
* Innodb支持行锁级别的锁粒度, MyISAM的锁粒度级别为表锁
* Innodb支持外键, MyISAM不支持
* Innodb使用聚簇索引, MyISAM使用非聚簇索引
```

Memory 是使用内存的存储引擎

**_如何使用 SQL 语句设置/切换表的存储引擎?_**

```sql
ALTER TABLE table_name ENGINE = MYISAM

CREATE TABLE table_name(
 ...
) ENGINE = InnoDB
```

# 2. Innodb

## 1. 表的存储

**_Innodb 中表是如何存储的?_**

表存储在文件系统中, 读写表时先将表文件读取到内存缓冲池中(以 16KB 大小的页为基本单位), 读写完成后再刷新到文件中

**_一个页中行记录是如何存储的?, 如何进行查找的?_**

一个页中的行记录使用**单链表**连接, 并且链表至少有一个最大记录和最小记录

查找记录时**对单链表的目录进行二分查找,** 快速定位到要查找的记录所在的 slot

![1691476779977](image/mysql/1691476779977.png)

## 2. 记录行的存储

**_Innodb 中行格式有哪些?_**

常见的行格式: Compact, Dynamic, Compressed

**_简单说一下你知道的行格式具体是怎样的?_**

一行记录由: `变长字段长度列表+null值列表+记录头+记录数据` 组成

其中 null 值列表使用一个 bit 位记录一个字段是否位 null

![1691475887978](image/mysql/1691475887978.png)

**_记录头中记录了什么信息?_**

-   **trx_id:** 事务 id, 用于 mvcc
-   **roll_ptr**: 用于 mvcc
-   **row_id**: 行 id, 自动生成

**_如果一行记录太长了, 导致一个页放不下, 如何存储?_**

对于过长的字段(如 varchar 最大字节长度为 65535), 在原本存储字段值的地方存储一个**溢出指针**, 指向存放溢出数据的页

## 3. 索引

**_Innodb 为什么使用 B+树作为索引, 而不使用 B 树/AVL?_**

```
B树/AVL在范围查询时不支持索引
```

**_什么是聚簇索引/二级索引/联合索引?_**

```
* 聚簇索引(clustered index): 以id建立的索引, 叶节点存储全部字段

* 二级索引(secondary index): 以字段A排序的索引, 叶节点存储(A, id)

* 联合索引: 以多个字段(A, B)排序的索引, 叶节点存储(A, B, id)
```

**_什么是回表?_**

```
"select * from table where ctime > 10"执行时会:
1. 查二级索引"ctime"获取id
2. 按id查询聚簇索引获取全部字段(回表)

tip: 如果在第1步就能获取到select中的全部字段(覆盖索引)则不用回表
```

**_什么是最左匹配原则?_**

```
要使用联合索引, 作为查询条件的字段必须覆盖联合索引的左前缀
```

**_索引 B+树为什么高度一般不会超过 3 层?_**

```
一个页大小为16KB可以存储约1000条记录, 3层的B+树可以存储10^6条记录
```

**_索引什么情况下会失效?_**

-   索引字段进行计算时发生了类型转化 / 使用了函数转换
-   联合索引没有按照最左匹配原则
-   特殊的查询条件, 如 NOT NULL, !=, like+"%"通配符作为前缀

**_什么字段适合建立索引, 建立索引有哪些技巧, 使用索引有哪些注意事项?_**

-   重复度小的字段适合作为索引
-   频繁作为查询条件和排序条件和分组条件的字段适合作为索引
-   不要建立过多的索引
-   表太小也不要建立索引

**_为什么建议使用自增主键?_**

```
在插入数据时使用自增主键可以减少聚簇索引的页分裂
```

# 3. 事务和锁

## 1. 事务简介

**_什么是事务, 事务的 ACID 特性的是什么?_**

事务通俗的讲就是一段**要么全部执行, 要么全部不执行**的 SQL

```
A: 原子性: 要么全部执行, 要么全部不执行
C: 一致性: 事务执行前后数据库保持一致性
I: 隔离性: 不同事务之间的执行互不影响
D: 持久性: 事务对数据库的影响是永久的
```

**_事务隔离级别有哪些, 每个隔离级别会导致的问题?_**

```
RU:  一个事务内可以读取到还未提交的事务进行的更改, 会有脏读问题
RC: 一个事务内只能读取到已经提交的事务的更改, 会有不可重复读问题
RR(innodb默认隔离级别):  一个事务期间数据的值在事务开始和结束时一致, 会有幻读问题
SERIAL: 事务之间串行执行
```

## 2. MVCC

**_RC, RR 隔离级别下是如何进行快照读的/MVCC 原理?_**

每条行记录中的记录头中包含**rollback_pointer 指针**和写入该记录的事务 id: **transaction_id**, 由 roll_ptr 指针构成了一个版本链(旧记录存储在**undo log**中), 每次查询的时候会生成一个 ReadView, ReadView 定义了当前事务可见的记录的 transaction_id 的范围, 然后顺着版本链查找记录, 如果记录不可见则跳过

## 3. 锁

**_锁的分类有哪些, CRUD 各自会加什么锁?_**

锁分类:

```
* 互斥锁 SELECT ... FOR UPDATE
* 共享锁 SELECT ... IN SHARE MODE

* 行锁
* 表锁
* 表意向锁: 申请行锁前需要先申请表意向锁
* 自增锁: 自增主键递增时需要申请自增锁

* 记录锁: 修改记录前先申请记录锁
* 间隙锁: 插入数据前需要申请间隙锁
* 临键锁: 记录锁 + 记录旁边的间隙锁
* 隐式锁
```

-   SELECT: 互斥锁/共享锁
-   DELETE: 互斥锁
-   UPDATE: 先加互斥锁(读), 在按照情况加 INSERT 对应锁(修改了主键的情况)
-   INSERT: 加插入意向锁, 隐式锁(使用事务 id 实现)

## 4. 日志

_如何解决事务的持久性问题?_

```
Write Ahead Log(WAL)思想, 在写入磁盘前先写入日志, 例如Innodb执行事务时:

1. 修改buffer pool中的数据页
2. 将修改过程记录到内存中的redo log buffer中
3. 事务要提交时, 将redo log从内存刷新到磁盘, 状态为Commit Record, 此时事务提交成功
4. 定时(Checkpoint时)将buffer pool中的脏数据页刷新到磁盘中, 刷新后将磁盘中的redo log状态标记为End Record

tip: 在数据库crash后的重启阶段, 会先执行redo log中状态为Commit Record的记录, 恢复内存中丢失的数据页
tip: 第2步中, redo log中一条记录的结构为(页号, 偏移号, 修改后的数据)
tip: 第4步中, 实际写入到磁盘时还会经过os级别的文件系统缓存, 但innodb默认会立刻fsync
```

_为什么有了 redo log 还需要 undo log?_

```
仅仅有redo log不能保证原子性, 例如一个事务修改了所有页面, buffer pool中的部分脏页面必须在事务提交前刷新到磁盘上(steal策略)
```

# 4. 调优相关

## 1. Explain

**_EXPLAIN 可以做什么?_**

Explain+DQL 语句可以查看执行计划, 执行计划中重要的字段有:

```
* type: 查询方式
* key: 使用到的索引
* rows: 预估需要扫描行数
* filtered: WHERE子句过滤了行数占比(%)

tip: 使用explain只能查看执行计划, 并没有实际执行SQL!
```

**_查询方式有哪些?_**

```
* 全表遍历: all
* 索引上全量遍历: index
* 索引范围扫描: range
* 索引的等值查询: ref
* 直接查系统表: system/const
```

## 2. 分库分表

**_什么是雪花算法, 为什么使用雪花算法而不是表的自增 ID?_**

雪花算法生成的 ID 结构: `1个标志位+41位时间戳+10位机器码+12位序列号`

```
使用雪花算法的原因:
* 分库分表导致表的自增id可能相同, 使用雪花算法可以生成全局唯一id且这些id大致递增, 避免B+树索引频繁的节点分裂

* 即使不是分库分表, 单一表中使用自增id会导致表锁, 高并发下性能差
```

**_分库分表有哪些方式?_**

分为垂直分表和水平分表, 垂直分表将一个表中不常使用的字段和常用字段拆分到不同表中, 水平分表将一张表中的记录拆分到不同表当中

## 3. 主从数据库

**_什么是主从架构, mysql 中如何实现主从同步的?_**

主服务器负责写入, 通过同步机制同步到从服务器, 从服务器负责响应读请求

mysql 中通过 binlog 实现主从同步
