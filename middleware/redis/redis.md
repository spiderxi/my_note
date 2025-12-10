# 1. Redis简介

## 1. 术语

***常见的缓存实现方式有哪些?***

```
🌟 本地缓存 (如Guava Cache)

🌟 分布式缓存 (如Redis, Tair)
```

***Redis为什么快?***

```
🌟 使用内存存储数据, 内存io速度快
🌟 单线程避免了多线程的锁竞争和上下文切换
🌟 IO模型为IO多路复用
```

***Redis并发性能相较与DB如何?***
```
🌟 单机Redis并发QPS可达10w+, 可以通过集群扩展QPS
🌟 单机DB并发QPS可达1W, 读请求QPS可以通过增加从服务数量提升 
🌟 单次Redis请求耗时一般微秒级别, 单次DB请求耗时1ms到1s

🌙 注意单个hot key只会存在一个Redis实例中, 设计hot key问题可考虑多级缓存
🌙 实际性能和机器配置有关
```

***Redis和Tair的区别?***

```
🌟 Redis使用纯内存, 速度更快(<1ms)
🌟 Tair使用内存和外存, 速度稍慢(1ms-20ms), 但会进行持久化
🌟 两个并发QPS差距不大

🌙Tair适用于冷热数据显著的场景, 内部有热点探测机制
```

***Redis事务能保证原子性和持久性吗?***

```
不能保证严格意义上的原子性和持久性:
🌟原子性: redis可以保证原子地执行一批指令, 但不支持回滚
🌟持久性: Redis采用RDB和AOF持久化机制保证持久性, 但默认不开启

🌙使用multi exec开启和提交事务
```

***Redis的Pipeline可以保证原子性吗?***

```
不能, pipeline只能用于减少网络吞吐量(批量执行一批指令), 不保证执行这些指令期间不会执行其他指令
```

## 2.数据类型

***Redis五大常用数据类型是什么, 三大扩展类型是什么?***

```
基本类型: string, list, hash, set, zset(sorted set)

扩展类型: geospitial, hyperloglog(统计UV), bitmap
```

***Redis中哈希表扩容/缩容 是如何进行的?如何解决哈希冲突的?***

```
🌟 采用链表法解决哈希冲突
🌟 扩容缩容采用渐进式rehash, 维护了两个的散列表, 在对单个元素进行CRUD操作时会检查Load Factor是否过大/过小以决定是否将元素迁移到另一个散列表(Copy On Write)
```

***什么是Skip List?***

```
🌟跳表本质上是一个链表, 但链表的一个节点有多个不同level的指针, 每个level指针指向链表中下一个同level指针, 更高级的level的指针实质上构成了低级level指针的索引

🌟索引原理: 新插入的节点需要分配随机level(level从低到高概率呈指数级下降), 对跳表的基本操作的时间复杂度为log(N)
```

***GeoSpitial的邻近算法的原理?***

```
1️⃣ 使用GeoHash算法对经纬度进行编码
GeoHash算法: 将经纬度按照二分法进行分块(左半=0,右半=1,上半=0,下半=1), 然后按经纬度的01交替拼接组成一个数字, 数字越长确定的区域越小, 并且相领数字对应的区域也相邻
2️⃣ 将(key=GeoHash(经纬度), value=List<经纬度>)放入skip list中
3️⃣ 进行地理位置A邻近匹配时, 会先读取GeoHash(A)下的所有地理位置, 再读取改区域周围8个区域的节点位置(以此类推)
```

***Hyperloglog统计原理是什么?***

```
🌟 核心思想: 利用局部数据特征估计数据总体值
🌟 概率学理论: 伯努利实验中, 进行实验次数n与每次实验抛掷硬币次数k在最大似然估计下有如下关系: n = 2^{max(k1, k2, ..., kn)}

🌟 具体实现: 为了避免单轮伯努利实验带来的误差, 采用m轮实验并使用m个桶储存实验结果(数据bucket共存储m个k_max)
1️⃣ 对于每个uid, index(hash(uid))确定桶索引, sample(hash(uid))为单次实验结果
2️⃣ 获取sample(hash(uid))第一次比特位为1的索引位置k, 更新bucket[index(hash(uid))] = max(k, bucket[index(hash(uid))])
3️⃣ 根据公式计算: uid去重个数 = 实验次数n = 矫正常量c * 实验轮数m * 2^{m个k_max的调和平均数}

🌙除了Hyperloglog, 还可以使用HashMap和BitMap统计UV, 但内存开销更大
```


# 2. Redis单机

## 1. 过期键删除

***Redis中是如何删除过期键值对的?***
```
redis采用惰性删除和周期删除相结合的方式删除过期键

🌟 惰性删除: 当对键值对操作时, 如果检测到已经过期, 则删除
🌟 周期删除: 每隔一段时间(每个处理周期内)删除过期字典中部分的过期键
```

## 2. AOF和RDB

***AOF和RDB的原理?***
```
🌟 AOF: append only file, 将写指令以日志的方式写入文件
🌟 RDB: 保存数据库快照文件(fork子进程进行持久化, 由于fork()的Copy On Write特性子进程复制了父进程的数据库快照)

🌙 Redis默认不开启持久化
🌙 Redis启动时会选择AOF文件或RDB文件进行数据恢复
🌙 RDB持久化由于使用的快照, 可能导致数据丢失
```

***如何启动Redis的持久化?***

```
修改Redis配置文件

🌙 也可以手动执行指令SAVE(主进程同步持久化) BGSAVE(子进程异步持久化) 进行RDB持久化
```

***AOF文件过大怎么办?***
```
当检测到AOF文件过大时Redis会fork子进程对AOF文件进行重写, 重写原理为对数据库快照生成写入指令

🌙 如果重新期间有写请求会将写入指令会被放到重写缓冲区, 待子进程重写完成后主进程会将缓冲区的命令刷到AOF文件中
```

## 3. Reactor模型
***讲一下Redis的Reactor模型?***
```
Redis采用单Reactor单Worker模型, 主线程中循环使用IO多路复用(系统调用select()/epoll())监听并处理多个socket IO事件并处理周期任务
```

# 3. Redis集群

## 1. Redis Sentinal
***Redis Sentinal模式架构是怎样的?***
```
一主多从多哨兵, 其中每个哨兵监听所有主从节点的状态, 主节点宕机时哨兵决定fail over后的主节点
```

***Redis中主从复制方式是什么?***
```
🌟 异步复制
🌟 新节点启动时请求主节点进行全量复制(RDB文件), 后续增量复制

🌙 Redis默认读写都从主节点
🌙 实际使用中, redis一般都用作缓存, 可以接受缓存丢失
```

***Redis中主从异步复制导致的问题?***
```
🌟 主节点写请求完成后立刻宕机, 在Fail Over时, 从节点还没有复制, 导致数据丢失
🌟 脑裂也会导致数据丢失

🌙 脑裂指网络分区导致出现两个主节点的现象(主节点在北京机房, 从节点和哨兵都在上海机房, 但上海和北京之间网络断开时上海节点中会选举出一个主节点)
```

***Redis如何配置同步复制和半同步复制?***
```
🌟 修改配置文件的配置项: min-slaves-to-write(可解决脑裂问题)
```

***讲一下Fail Over的过程?***
```
Sentinel节点1秒1次检测主节点是否存活, 当多个Sentinel节点检测到主节点宕机时, Sentinel节点间通过Raft算法选出Leader Sentinel, 并由Leader Sentinel进行Fail Over
```

## 2. Redis Cluster

***Redis Cluster的作用和原理?***
```
原理: 使用一致性哈希算法, 将key映射为slot, 请求时直接请求slot对应的主从节点(一主多从)
作用: 便于横向扩展数据容量

🌙 客户端请求错误节点时, 返回MOVED重定向到正确节点
🌙 主节点间通过Gossip协议交换集群状态信息, 维护最终一致性
🌙 当一个主节点宕机时, 其他主节点通过协议从从节点中选举出下一个主节点
🌙 默认读主节点, 可通过READONLY命令指定从节点读
```

# 4. 缓存实践

## 1. 缓存使用要点

***什么是缓存穿透问题?***
```
大量请求访问了DB中不存在的数据(此时缓存层没有起作用, 每次都会请求DB导致DB被压垮)

🌙 解决方法: 对DB中不存在的数据缓存空值/BloomFilter
```

***什么是缓存击穿问题?***
```
缓存过期瞬间, 多个请求直接访问了DB

🌙 解决方法: 请求访问DB更新缓存前先获取Redis分布式互斥锁(未获取到锁的线程自旋)
```

***什么是缓存雪崩问题?***
```
多个热点缓存同时失效导致请求大量打到DB

🌙 解决方法: 过期时间设置为随机数/预热时间错开
```

***什么是BloomFilter?***
```
🌟BloomFilter可以判断一个数据一定不存在
🌟原理: 一个BloomFilter由一个bitmap和多个hash函数组成, 所有索引位置=hash(data)的bit位为1则表示存在该数据, 否则不存在
```

## 2. 缓存更新策略
***当请求写入了DB时如何修改缓存?***
```
先修改DB, 再删除缓存(对一致性较高的话还需要使用延迟MQ双删, 定时任务diff)

🌙 必须删除缓存而不是更新缓存, 并发情况下更新的次序不能保证!
```


## 3. 分布式锁

***如何使用Redis指令实现分布式锁?***

```
🌟加锁:
SETNX(LockName, IP_ThreadId, ExpireTime)

🌟解锁:
CompareAndDelete(LockName, IP_ThreadId)

🌙将value设置为IP_ThreadId目的是为了防止线程处理超时后, 解锁时key已经删除, 如果不校验value可能会解锁其他线程加的锁
🌙可以使用开源库Redisson作为分布式锁实现
```

***为什么可以通过LUA脚本保证操作的"原子性"?***

```
Redis采用单线程模型, LUA脚本执行时不会有其他Redis命令执行

🌙集群中如果一个Lua脚本中同时操作了多个key会导致cross-slot问题不能保证原子性
```

***怎么解决线程执行时间超时导致锁超时问题?***

```
使用守护线程在超时前更新超时时间实现自动续约(watch dog机制)
```

***Redis主从异步复制为什么会导致分布式锁出现问题?***

```
如果在加锁成功后主节点立刻宕机, 来不及将数据同步给从节点. 此时可能导致锁被另一个线程获取

🌙可以使用RedLock算法解决: 使用多个Redis主节点, 只有当半数以上主节点SETNX成功才算加锁成功
```

***Redis如何实现可重入锁和公平锁?*** 

```
🌟可重入锁: 将加锁操作和解锁操作改为: 
加锁: IncrementIfNX(LockName, IP_ThreadId, ExpireTime) 
解锁: CompareAndDecrement(LockName, IP_ThreadId, ExpireTime)

🌟公平锁: 加锁前维护一个Redis List, 只有IP_ThreadId在列表头时才加锁成功
```

***使用Zookeeper实现分布式锁和使用Redis实现分布式锁有什么差异?***

```
🌟 Redis并发性能更好
🌟 Zookeeper安全性更好(Redis主从是异步复制, 可能导致锁失效; Zookeeper采用分布式一致性协议)
```

## 4. 其他实践

***如何使用Redis实现排行榜?***
```
使用ZSet, 其中key=score, value=uid
```

***如何使用Redis实现集群限流?***
```
使用ZSet实现滑动窗口(score = 时间戳)

🌙 单机限流一般使用令牌桶算法
Guava中的限流器原理为 TokenNum = Min(MaxTokenNum, lastNum + rate * (nowTime - lastTime))
```

***如何使用Redis实现BloomFilter?***
```
使用bitmap和多个hash函数
```

***什么是Large Key?***
```
Large Key:
🌟String类型的key, value过大(一般认为>100KB就过大, Redis系统级别限制了最大大小512MB, 超过会报错)
🌟hash/set/list/zset元素过多

🌙 Large Key操作耗时长会导致后续指令阻塞时间过长
🌙 删除Large集合时要对元素时要分批删除(Redis中集合删除元素的复杂度为O(N)), 或使用UNLINK指令
```
