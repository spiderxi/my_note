# 0. SQL语句

```
Select ...
From ...
Where ...
Group by ...
Having ...
order by ...
limit ...
```

# 1. MySql数据库相关

## 1.1 数据类型和字符集

**字符集使用utf8mb4**

整数类型

```
1. 字节长度从小到大: tinyINT(1字节) smallINT(2字节) mediumINT(3字节) INT(4字节) bigINT(8字节)
   采用补码, 如tinyINT取值范围-128~127
2. 每种类型对应一种无符号的相同长度的类型 unsigned Xxx
```

浮点数和定点数

```
1. float(4字节浮点数), double(8字节浮点数)均为浮点数
2. 浮点数可以指定小数位数, 如float(5, 2)保留2位小数, 3位整数
3. decimal(10, 2)表示保留2位小数, 和8位整数, 相较于浮点数计算是精确的
```

时间类型

```
year: 四位数, 如2002, 2005
time: HH:MM:SS, 表示一天中的时刻或者时间间隔, 如20:16:18, 125:59:59
date: YYYY-MM-DD, 如2022-10-01
datetime: date和time的组合, YYYY-MM-DD HH:MM:SS
timestamp: 格式和datetime相同, 但在存储时会根据当前时区转换成UTC时间, 查询时会根据时区显示UTC时间对应的当前时区时间
```

字符串和文本

```
char: 长度为255的unicode字符串
varchar(x): 长度为x的unicode字符串
text, mediumtext, longtext(4GB): 长度更长的文本
```

其他类型

```
binary: 固定长度的二进制数据
varbinary(x): x字节长度的二进制数据
tinyblob blob(64kb) mediumblob(16MB) longbolb(4GB): 二进制大对象
json: json
```

## 1.2 约束和范式

### 1.2.1 范式

范式: 数据库设计的规范, 大NF包含小NF

* 1NF: 属性是原子的, 不可再分的

![1680052360180](image/mysql/1680052360180.png)

上面表中电话属性不是原子的

* 2NF: 非候选键对于主键只有完全函数依赖, 不存在部分函数依赖

![1680052834423](image/mysql/1680052834423.png)

主键为(学号, 课程名称), 姓名只依赖学号, 存在部分函数依赖

* 3NF: 不存在传递性依赖

![1680052997798](image/mysql/1680052997798.png)

班级依赖于学号, 班主任依赖班级, 产生了传递性依赖

### 1.2.2 约束

6大约束

```
* 非空约束  not null
* 默认约束 default "默认值"
* 检查约束 check age >= 18
* 主键约束 primary key
* 外键约束 foreign key
* 唯一约束 unique,  unique约束的字段中不能包含重复值(null除外)
```

约束的目的是为了实现数据完整性, 包括四个数据完整性

```
* 实体完整性:  一张表中可以
* 域完整性: 字段的取值范围要符合语义
* 引用完整性: 不同表之间的引用是有效的
* 用户自定义完整性: 用户自己对字段取值的限定
```

## 1.3 权限管理和访问控制

用户管理

```sql
select user from mysql.user;
create user'zhangsan'@'localhost'identified by'123';
drop user zhangsan@localhost;
```

访问控制

```sql
grant select(cust_id,cust_name)
    on mysql_test.customers
    to'zhangsan'@'localhost';--授予在数据库mysql_test的表customers上拥有对列cust_id和列cust_name的select权限

grant all
    on mysql_test.*
    to'zhangsan'@'localhost';--授予可以在数据库mysql_test中执行所有操作的权限

grant create user
    on *.*
    to'zhangsan'@'localhost';--授予系统中已存在用户zhangsan拥有创建用户的权限

revoke select 
    on mysql_test.customers
    from'zhangsan'@'localhost';--回收用户zhangsan在数据库mysql_test的表customers上的select权限
```

RBAC

```sql
CREATE ROLE 'app_developer';  //创建角色
GRANT ALL ON app_db.* TO 'app_developer';  //给角色赋予权限
GRANT 'app_developer' TO 'dev1'@'localhost'; //给用于赋予角色
```

## 1.4 mysql 架构

![1680113481060](image/mysql/1680113481060.png)

查询优化器优化方法

```
1. 利用关系代数进行查询逻辑优化 6大基本关系代数: 投影 (π) 选择 (σ) 笛卡尔积 并集 差集  
   重命名 (ρ)
2. 利用索引进行优化
```

缓存

```
MySQL8.0+后默认关闭, 对于修改频率很低的表可以显式开启缓存
缓存命中判断方式是对QL使用Hash算法, 当表修改后缓存失效
```

## 1.5 缓冲池

内存中的**缓冲池** (buffer pool)用来加速数据的访问

```
对于外存中的数据页, Mysql会预读到主存页中加速访问速度, 采用局部访问性原理, 已经访问的页附近的页会进行预读(Read-Ahead)
```

页的替换策略采用优化后的LRU算法

```
 - LRU链表分为new和old区, 预读的页放入old区, 只有真正访问时才进入new区头部
 - 页被访问时, 该页在old区停留时间超过阈值时才可以进入new区, 避免顺序扫描时热点页丢失
```

![1680117103152](image/mysql/1680117103152.png)

## 1.6 存储引擎

常用存储引擎Innodb Myisam

![1680118434418](image/mysql/1680118434418.png)

## 1.7 ER图

一个实体对应一张表, 属性对应一个字段

![1680172749645](image/mysql/1680172749645.png)

多个实体间的关系对应一张表, ER图中标注关系的数量关系

![1680172831767](image/mysql/1680172831767.png)

# 2. 索引

### 2.1 Innodb索引

索引采用B+树, b+树叶节点是数据项组成的页, 分支节点是目录项组成的页

![1680150198857](image/mysql/1680150198857.png)

假设一个数据项页100项, 一个目录项页1000项, 3层时可以检索100×1000×1000 = 1亿项数据, 实际b+树层数 <= 4

聚簇索引和二级索引

```
聚簇索引: 一条完整的数据存储在索引b+树的叶子节点中, 聚簇索引适用于自增主键, 自增主键在插入时
          页分裂次数少
二级索引: b+树叶子节点中存储的是主键的值, 非聚簇索引适合非主键的键, 查询时需要两次索引, 
          一次找到主键, 一次通过主键查询聚簇索引
```

联合索引

```
联合索引也是一个B+树, 但数据项和目录项比较大小是使用多个字段(f1, f2, f3), f1相等时比较f2, f2相等时比较f3
```

目录项的唯一性

```
如果一个目录页里的两个目录项相同均为x, 那么一条值为x的数据插入时久不知道走哪条目录项, 所以对于二级索引和联合索引, 还需要添加主键形成"联合索引", 确保目录项的唯一性 
```

### 2.2 MyIsam索引

MyIsam的数据顺序存储, 索引单独一个文件存储, 所以所有索引都属于非聚簇索引

* MyIsam表中可以没有主键

### 2.3 索引的注意事项

* 主键类型不宜过长, 这样一个非聚簇索引一个页可以存放更多的项
* 索引数量不宜过多, 过多时增删改会在索引上有过多开销
* 实际B+树根节点常驻内存, 避免多次磁盘IO

### 2.4 创建和删除索引

```sql
CREATE INDEX idx_name ON table_name(field1, field2);//添加
ALTER TABLE table_name DROP INDEX index_name; //删除
```

### 2.5 索引的最佳实践

适合做索引的字段

* 唯一性的字段 (如学生id)
* Where条件的字段
* group by/order by的字段
* 连接条件的字段

不适合做索引的字段

* 表较小时(<=1000)不使用索引
* 重复数据多的字段(性别)
* 频繁更新的字段

索引字段注意事项

* 索引对应的类型存储长度越小越好
* varchar类型使用固定长度的前缀做索引
* **被索引最频繁的列放在联合前缀的最左侧**

## 2.6 常见的树-数据结构

常见的树

```
BST: 二叉搜索树
AVL: 二叉平衡搜索树
B-tree: 多路平衡搜索树
B+ tree: 也是多路搜索树, 但数据全部存储在叶子节点, 叶子节点靠指针相连
R tree: B树的空间版

B+树更适合数据库检索, 顺序检索时速度更快
```

## 2.7 Sql调优

### 2.7.1 最佳左前缀法则

如果要使用联合索引, 索引字段必须包含联合索引的左前缀

```sql
SELECT * FROM student
WHERE classid=1 AND name = 'abcd';
/*没有使用索引：因为没有classid索引、也没有（classid、name）索引
  没有使用(age,classId,NAME)索引的原因：不符合最佳左前缀法则

SELECT * FROM student
WHERE classid=1 AND student.age>0 AND student.name = 'abcd'; 
/*使用了索引三(age,classId,NAME)：索引字段为age、classId  、NAME
```

### 2.7.2 索引失效情景

函数/计算会导致索引失效

```sql
#left函数的使用导致索引失效
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE LEFT(student.name,3) = 'abc'; 
# 使用“stuno+1 = 900001”算术运算导致索引失效
EXPLAIN SELECT SQL_NO_CACHE id, stuno, NAME FROM student WHERE stuno+1 = 900001;
# subString函数导致索引失效
EXPLAIN SELECT id, stuno, NAME FROM student WHERE SUBSTRING(NAME, 1,3)='abc';
```

联合索引中 < <= > >=等条件后面的字段, !=, is not NULL字段

```sql
#创建联合索引 
CREATE INDEX idx_age_classId_name ON student(age,classId,NAME);

SELECT * FROM student 
WHERE student.age=30 AND student.classId>20 AND student.name = 'abc' ; 
# name字段没有使用索引

SELECT * FROM student 
WHERE student.age=30 AND student.classId = 20 AND student.name != 'abc' ;
# name字段没有使用索引

SELECT * FROM student 
WHERE student.age=30 AND student.classId = 20 AND student.name is not null ;
# name字段没有使用索引
```

like "%xxx"

```sql
SELECT * FROM student 
WHERE student.age=30 AND student.classId = 20 AND student.name like '%jerry%' ;
# name字段没有使用索引
```

## 2.8

# 3. 事务

## 3.1 事务特性

ACID

```
A: 原子性
C: 一致性(数据完整性没有被破坏)
I: 独立性(并发事务之间要相互隔离, 避免相互影响)
D: 持久性(对数据库的修改是永久的)
```

## 3.4 事务隔离级别

### Read Uncommitted（读取未提交内容）

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。 可能造成脏读

### Read Committed（读取提交内容）

这是大多数数据库系统的默认隔离级别。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。 可能造成不可重复读

### Repeatable Read（可重读）

这是MySQL的默认事务隔离级别, 在可重复读中，该sql第一次读取到数据后，就将这些数据加锁（悲观锁），其它事务无法修改这些数据，就可以实现可重复读了。但这种方法却无法锁住insert的数据。可能造成幻读

### Serializable（可串行化）

这是最高的隔离级别，它强制事务完全互斥.

## 3.5 锁

### 3.5.1 全局锁

在数据库备份时会加全局锁, 此时只能执行DQL

### 3.5.1 表锁和行锁

* S(share)锁  X(exclusive)锁
* IS, IX锁(意向锁)

```
-- 事务要获取表的 X/S 锁，必须先获得表的 IX/IS 锁。
```

![1680179793947](image/mysql/1680179793947.png)

当A想申请表的S锁时, 需要确保每一行X行锁没有被其他人持有, 遍历查找效率低, 直接检查IX锁是否被持有

* 间隙锁

![1680180159868](image/mysql/1680180159868.png)

当要往3-8间插入数据时需要获取3-8间的间隙锁, 可以防止幻读

## 3.6 MVVC

全称Multi-Version Concurrency Control，即 `多版本并发控制`，主要是为了提高数据库的 `并发性能`

> 通俗的讲，数据库中某一条记录的多个版本同时存在

表中的每一行都有三个隐藏字段

![1680182278542](image/mysql/1680182278542.png)

多个事务并发操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（roll_pointer），连成一个链表，这个链表就称为 **版本链** 。

![1680182340559](image/mysql/1680182340559.png)

**快照读：** 读取的是记录数据旧版本。不加锁

**当前读** ：读取的是记录数据的最新版本，加锁

MVVC查询流程

```
1. 根据隔离级别和当前事务id和数据行最后修改的事务id判断数据行是否可见
2. 如果不可见顺着版本链查找, 找到最近的可见版本(版本号就是事务id号)位置
3. 返回可见的版本的数据
```

# 4. Mysql日志

## 4.1 redo undo

## 4.2 binlog

## 4.3 数据库备份和恢复
