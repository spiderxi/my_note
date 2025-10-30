# 1. Kafka 简介
## 1. 常识
_消息队列(MQ)的好处是什么??_
```
🌟 对流量削峰填谷
🌟 解耦不同系统(同步RPC改为异步MQ处理)
```

_什么技术和设计使得Kafka吞吐量大?_
```
🌟 PageCache
🌟 Zero Copy

🌙 kafka常用语日志收集+异步系统处理
```


## 1. 术语

_解释一下kafka的常用术语?_

```
🌟 Producer = 生产消息的客户端, Consumer = 消费消息的客户端, Broker = 保存消息的服务器
🌟 Topic = 生产消息的逻辑单位, Consumer Group = 消费消息的逻辑单位, 一个Topic对应多个Consumer Group
🌟 Zookeepr = 保存Kafka集群的元信息


🌙 一个Topic的一条消息, 可以被多个Consumer Group消费(每个Consumer Group各自消费一次)
```

*讲一下kafka是如何进行分片和备份的?*
```
🌟 分片(Partition): 一个主题可以分为多个Partition, 一条消息只会存储在一个分片里
🌟 备份(Replica): 一个主题一个分区中, 分为Leader和Follower, Follower仅同步数据并冗余存储消息, Leader负责处理生产/消费请求


🌙 一个Partition中的消息是按日志的方式顺序写的, 因此每个消息都有唯一Offset
🌙 实际集群中一个主题的多个Partition可能在一个Broker上, 但Follower和Leader往往不在一个Broker上
🌙 一个消息存储在哪个分区取决于路由算法, 一般使用hash或轮询路由
```

*什么Kafka死信队列?*
```
死信队列是一个特殊的延迟队列, 当多次消费失败导致消费次数 > 最大消费次数(配置项)时, 消息会被发送到死信队列

🌙 死信队列用来解决无限重试导致消息阻塞的问题
```

# 2. 设计细节

## 1.生产与消费
_出现QueueFullException原因?_
```
生产者生产消息的速度超过了Broker处理速度(超过处理队列长度)

🌙 可以减缓生产消息速度或增加Broker数量
```

_什么是Batch发送?_
```
生产者生产消息后按batch发送给Broker, 减少网络IO次数
```

_kafka的ACK模式有哪些?_
```
🌟 ACK=0: Broker在消息放到处理队列后立刻ACK, 客户端无需进一步等待
🌟 ACK=1: Leader保存完后再ACK, 客户端需要等待
🌟 ACK=ALL: 等待所有ISR都保存完后再ACK, 客户端等待时间可能偏长, 但能保证Leader故障后消息不丢失(类似与半同步复制)

🌙 严格意义上讲, kafka的Leader和Follwer之间只有异步复制, 注意ACK=ALL不等于同步复制/半同步复制
```

_kafka消费者使用的是poll还是push模式?_
```
采用poll模式, 消费者轮询Broker
```

_消费者可以指定分区和offset进行消费吗?_
```
可以
```

_AR ISR OSR 的关系?_
```
🌟 AR(all-replica): 一个分片的所有副本
🌟 ISR(in-sync-replica): 同步进度合格的副本
🌟 OSR(out-sync-replica): 同步进度过慢的副本

🌙 AR = ISR + OSR
🌙 ISR不能保证和Leader的Offset完全相同, Offset完全相同需要ACK模式=all保证, ISR仅仅保证上次同步时间间隔<配置阈值
🌙 Leader Replica宕机时, 优先选择ISR作为下一个Leader Replica, 如果ISR为空, 可以通过配置项决定是否阻塞等ISR出现或直接将OSR升级为Leader
```

## 2.高性能

## 3.高可用

_kafka 集群中 controller 的选举机制?_

```
broker启动后监听zookeeper中的值为controller的znode, 如果znode发生变化, 采用FCFS的方式抢占znode
```

_kafka 集群如何解决 controller 的脑裂问题?_

```
每次选举出的controller都有一个epoch number, 该值递增, controller在放松命令给broker时会带上epoch number, broker忽略更小的epoch number对应的命令
```
