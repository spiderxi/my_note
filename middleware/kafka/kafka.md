# 1. Kafka 简介
## 常识
_消息队列(MQ)的好处是什么??_
```
🌟 对流量削峰填谷
🌟 解耦不同系统(同步RPC改为异步MQ处理)

🌙 常见的MQ除了kafka, 还有rabbitMQ rocketMQ
🌙 kafka除了用于MQ, 还可以用作给流处理框架(如Spark)提供数据
```

_什么技术和设计使得Kafka吞吐量大?_
```
🌟 PageCache
🌟 Zero Copy

🌙 kafka常用于日志收集+异步系统处理
```


## 术语

_解释一下kafka的常用术语?_

```
🌟 Producer = 生产消息的客户端, Consumer = 消费消息的客户端, Broker = 保存消息的服务器
🌟 Topic = 生产消息的逻辑单位, Consumer Group = 消费消息的逻辑单位, 一个Topic下有多个Consumer Group
🌟 Zookeepr = 保存Kafka集群的元信息

🌙 Topic的一条消息可以被多个Consumer Group消费(每个Consumer Group各自消费一次)
```

*讲一下kafka是如何进行分片和备份的?*
```
🌟 分片(Partition): 单个主题下有多个Partition, 单条消息只会存储在一个分片里
🌟 备份(Replica): 单个分区中, 分为Leader Replica和Follower Replica, Leader负责处理生产/消费请求, Follower仅同步数据并冗余存储消息


🌙 单个Partition中的消息是按日志的方式顺序写磁盘的(追加日志), 因此消息在单分区上有序, 每条消息都有Offset标识, 但不同分区的消息不能保证顺序
🌙 一个消息存储在哪个分区取决于路由算法, 一般使用hash或轮询路由
```

_AR ISR OSR 的关系?_
```
🌟 AR(all-replica): 一个分片的所有副本
🌟 ISR(in-sync-replica): 同步进度合格的副本
🌟 OSR(out-sync-replica): 同步进度过慢的副本

🌙 AR = ISR + OSR
🌙 ISR不能保证和Leader的最大Offset完全相同(即不能保证主节点宕机时数据不丢失), ISR仅保证上次同步时间间隔<配置阈值, 数据不丢失需要ACK模式=all保证
🌙 Leader Replica宕机时, 优先选择ISR作为下一个Leader Replica
```
# 2. 设计细节

## 生产
_出现QueueFullException原因?_
```
生产者生产消息的速度超过了Broker处理速度(超过处理队列长度)

🌙 可以减缓生产消息速度或增加Broker数量
```

_什么是Batch发送?_
```
生产者生产消息后不立刻发送消息, 按batch积累后再发送给Broker, 减少网络IO次数
```

_生产者可以指定分区/offset发送消息吗?_
```
可以指定分区, 但不能指定offset(只能追加写入)
```

_kafka的ACK模式有哪些?_
```
🌟 ACK=0: 生产者在发送完消息后直接ACK, 不保证Broker能接收到消息
🌟 ACK=1: Leader保存完后再ACK, 客户端需要等待
🌟 ACK=ALL: 等待所有ISR都保存完后再ACK, 生产者等待时间可能偏长, 但能保证Leader故障后消息不丢失

🌙 实际上ISR可能只有Leader Replica, 导致ACK=ALL时也无法保证数据不丢失, 要保证数据不丢失需要配置最小ISR数量>1(默认值为1)
🌙 当ACK=1的时候看作主从异步复制, 当ACK=ALL时看作主从半同步复制
```

_讲一下生产者的retry-backoff机制?_
```
当生产者发送消息出现可重试异常RetriableException时, 客户端等待一定时间后自动重新发送消息

🌙 retry-backoff机制时为了自动解决某些重试后能成功的异常(网络抖动, 主节点故障转移)
```

_kafka的消息可以保证幂等吗?_
```
可以, 在配置项中开启enable.idempotence

🌙 幂等通过给生产者编号(pid)和消息编号(seq)实现, 对于每一个[pid, partition], broker都保存了一个自增的seq
🌙 broker在处理时会检查seq和last_seq, 如果seq <= last_seq会被幂等, 如果seq > last_seq+1会返回可重试异常
🌙 开启幂等后可以保证单分区单生产者消息有序
```

## 消费
_kafka消费者使用的是poll还是push模式?_
```
采用poll模式, 消费者轮询Broker
```

_消费者可以指定分区和offset进行消费吗?_
```
可以
```

## 高性能

## 高可用

_kafka 集群中 controller 的选举机制?_

```
broker启动后监听zookeeper中的值为controller的znode, 如果znode发生变化, 采用FCFS的方式抢占znode
```

_kafka 集群如何解决 controller 的脑裂问题?_

```
每次选举出的controller都有一个epoch number, 该值递增, controller在发送命令给broker时会带上epoch number, broker忽略更小的epoch number对应的命令(Raft机制)
```

# 3. 参考书籍
🌟 深入理解Kafka：核心设计与实践原理 -朱忠华