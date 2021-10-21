---
title: MQ
date: 2021-09-03 09:14:59
tags:
---



## Part. I MQ Fundamentals

#### 为什么要用 MQ？MQ 的优缺点？

优点：1. 解耦 2. 异步 3. 消峰。

缺点：1. 可用性降低 2. 复杂性变高 3. 存在一致性问题





#### 如何保证 MQ 高可用？

1. RabbitMQ 三种模式：单机模式、普通集群模式、镜像集群模式。单机模式和普通集群模式无法保证MQ高可用，镜像集群模式可用保证高可用。
2. Kafka 可以使用 Leader/Follower 模式，partition leader ，增加节点作为 follower，生产者往 leader 上写消息，消费者从 leader 消费消息，leader 向 follower 同步数据，如果 leader 宕机了，Kafka 会将 follower 提成 leader 保证 MQ 高可用。



#### Q: 如何保证消息可靠？消息丢失怎么办？

A: 

1. RabbitMQ 

   生产者使用 confirm 模式，可以保证消息在生产者一方不丢失。

   RabbitMQ server 开启同步落盘，保证消息持久化到磁盘才给生产者返回 confirm 保证在 MQ server 消息不丢。

   消费者使用手动 ack 模式，保证消息在消费者一方不丢失。

2. Kafka 

   生产者

   broker 设置

   消费者使用手动 ack 



#### Q: 如何保证消息顺序性？ 

A: 

1. RabbitMQ

   需要保证顺序的消息发送到同一个 queue 中，一个 queue 只有一个 consumer 消费。

2. Kafka

   需要保证顺序的消息发送到通一个 partition ，一个 consumer 消费一个 partition，此时需要单线程才能保证消息顺序，如果要提高吞吐量，使用多线程，可以使用内存队列，将需要保证顺序的消息发送到相同的内存队列，由一个线程消费。



#### Q: 如何解决消息延时和过期失效的问题？ 

A:  一般不会设置消息时效，如果真的失效了，只能自己写一个程序从源头开始一条一条手动补数据。



#### Q: 消息队列满了如何处理？如何解决消息积压？

A:  MQ 磁盘满了，一般情况都是消费者程序出了问题，可以修改消费者程序，将消费快速消费丢弃掉，保证 MQ 不会因为磁盘满了挂掉。

如果有大量的消息积压，一般情况也是消费者程序出了问题，可以建一个新的 topic ，这个 topic 多搞几个消费者，比如原来是 3 个，现在可以弄 30 个，然后修改消费者程序，将消息导到这个新的 topic 中，由 30 个消费者消费，这样消息的消费速度就是原来的 10 倍。



#### Q: 如何设计一个 MQ 架构。

A: 



## Part. II 消息中间件

**消息中间件比较**

|            | ActiveMQ                   | RabbitMQ                                                     | Kafka                                                        | RocketMQ                                                |
| ---------- | -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- |
| 吞吐量     | 万级                       | 万级                                                         | 百万级                                                       | 十万级                                                  |
| 时效性     | ms级                       | 微秒级                                                       | ms级                                                         | ms级                                                    |
| 可用性     | 主从架构                   | 主从架构                                                     | 分布式、多副本                                               | 分布式                                                  |
| 消息可靠性 | 低概率丢消息               |                                                              | 配置参数可以做到0丢失                                        | 配置参数可以做到0丢失                                   |
| 优缺点     | 成熟、功能完善、社区不活跃 | 基于elang语言开发，性能好，有比较好的UI，社区活跃，二次开发难度大 | 超高吞吐，高可用性，大数据实时计算和日志采集场景的事实上业内标准 | topic数量多性能也还好，社区比较活跃，在阿里有大规模应用 |

#### 

### RebbitMQ

#### 概念和术语

在学习 RabbitMQ 之前，先了解一下 RabbitMQ 的术语和基本概念。

1. AMQP
   高级消息队列协议 (Advanced Message Queuing Protocol, AMQP) 是一个应用层协议的开放标准。RabbitMQ 实现了 AMQP 协议。
2. Publisher
   向 MQ 发送消息的程序。
3. Message
   Publisher 通过 MQ 传递给 Consumer 的数据。
4. Consumer
   从 MQ 接收消息的程序。
5. Connection
   应用程序和 RabbitMQ Broker 之间建立的 TCP 连接。
6. Queue
   存放消息的队列。
7. Confirm
   Publisher 向 RabbitMQ 发送 Message ，RabbitMQ 收到 Message 之后给 Publisher 的回执，用来标识 RabbitMQ 收到 Message 。
8. Message acknowledgment
   消息回执，简称 ack 。Message 被 Consumer 消费之后，Consumer 向 MQ Server 发送一个 ack ，MQ Server 收到 ack 后会将 Message 删除。如果没有收到 ack 并且 Consumer 和 MQ Server  断开连接，MQ Server 会将 Message 发送给其他的 Consumer 处理。
9. Message durability
   消息持久化，用来保证 MQ Server 在异常宕机时消息不丢失。
10. Prefetch count
11. Exchange
    在 RabbitMQ 中，Publisher 不会直接将消息发送到 Queue ，而是将消息发送到 Exchange ，再由 Exchange 将消息路由给 Queue 。
12. RoutingKey
    发送消息时携带的 key 。
13. Binding
    绑定就是将 Exchange 和 Queue 进行关联。 
14. BindingKey
    BindingKey 是在 Exchange 和 Queue 绑定时指定的。BindingKey 并不是实际存在，而是为了和 RoutingKey 进行区分，实际和 RoutingKey 是一个东西。Exchange 在路由消息时，会将 RoutingKey 和 BindingKey 进行比较，如果相等或匹配，Exchange 就会将消息路由到对应的 Queue 。
15. Exchange Types
    路由规则，有 4 种：fanuot 、direct 、topic 和 headers 。

#### 工作流程

了解了基本概念，再来看看 MQ 的执行过程。
<img src="https://s1.ax1x.com/2020/04/25/JyRABt.png" alt="JyRABt.png" border="0" style="width:700px"/>

1. RabbitMQ 已经运行，并且 Exchange 和 Queue 已经绑定。
2. Producer 连接到 RabbitMQ ，并发送一条消息，这个消息上带着一个 RoutingKey 。
3. RabbitMQ 接收到消息，并给 Producer 发送了 Confirm 消息。
4. Exchange 根据 RoutingKey 将消息路由到 Queue 。
5. Consumer 从 RabbitMQ 接收一条消息，处理完成后向 RabbitMQ 发送一条 Ack 。
6. RabbitMQ 收到 Ack 后删除消息，整个流程结束。

#### Exchange

RabbitMQ 中有 4 种不同的 Exchange ，分别是 Direct 、Topic 、Fanout 和 Headers 。

##### Direct

RabbitMQ 默认的 Exchange Type 。使用此方式 Exchange 会比较 RoutingKey 和 BindingKey ，两个相等才会将消息路由到 Queue ，如果没有和 RoutingKey 相等的 BindingKey ，消息就会被丢弃。

<img src="https://s1.ax1x.com/2020/04/25/JyHBWt.png" alt="JyHBWt.png" border="0" style="width:700px" />

有两个 Queue 绑定到 Exchange ，分别是 hello.queue 和 hello2.queue 。Producer 向 Exchange 发送消息，RoutingKey 作为参数存放在消息的 header 。Exchange 会将消息路由到 hello.queue 的队列中，因为 RoutingKey 和 BindingKey 完全相等。

##### Topic

使用此方式 Exchange 会通过模式匹配的方式进行路由，RoutingKey 可以使用 “.” 、“#” 、“*” ，例如 order.create.* ，order.# ，”*“ 匹配 0 或 1 个，”#“ 匹配 0 或 n 个。Consumer 可以订阅任何感兴趣的 Topic ，Exchange 会将消息路由到所有符合模式的队列中。

<img src="https://s1.ax1x.com/2020/04/25/Jyv2jI.png" alt="Jyv2jI.png" border="0" style="width:700px"/>

Consumer1 订阅了 order.# ，Consumer2 订阅了 order.del.# ，Consumer3 订阅了 order.create.* 。Producer 向 Exchange 发送了一条消息，RoutingKey 为 order.create.test ，order.create.* 、order.# 都可以匹配 order.create.test ，因此 Consumer1 和 Consumer3 都会收到消息， order.del.# 无法匹配，则 Consumer2 不会收到消息。

##### Fonout

使用此方式 Exchange 会消息复制多份，并路由到所有与 Exchange 绑定的队列，此时将不再关心 RoutingKey 。这种模式通常在需要对相同的消息采取不同的处理时使用。

<img src="https://s1.ax1x.com/2020/04/25/J6SzUf.png" alt="J6SzUf.png" border="0" style="width:700px" />

##### Headers

使用此方式 Exchange 在绑定 Queue 时会增加一个特殊参数 x-match ，x-match 有两个值，分别是 any 和 all ，默认是 all 。Exchange 在路由匹配时，通过消息的 Header 进行匹配，而不是 RoutingKey 。
<img src="https://s1.ax1x.com/2020/04/25/J6kuPx.png" alt="J6kuPx.png" border="0" style="width:700px"/>
Consumer1 订阅了 type:order format:xml ，Consumer2 订阅了 type:pay format:binary ，Consumer3 订阅了 type:pay format:json 。Producer1 发送的消息 headers 为 type:order format:json ，Exchange 没有与其匹配的 Queue ，所以消息会被丢弃。Producer2 发送的消息 headers 为 type:pay format:text x-match:any ，Exchange 会将消息路由给 Queue2 和 Queue3 。Producer3 发送的消息 headers 为 type:pay format:json ，Exchange 会将消息路由到 Queue3 。

#### RabbitMQ Cluster

RabbitMQ 集群有两种：普通集群模式和镜像集群模式。

##### 普通集群模式

queue 的数据只保存在一个节点上，queue 的元数据信息每个节点都会保存。消费者可以连到任意一个节点上进行消费，如果这个节点没有 queue 的数据，这个节点会先从保存 queue 数据的节点把数据同步过来，再给消费者消费。

这种模式无法保证高可用，如果保存 queue 数据的节点宕机，数据就无法消费。另外，由于消费时会同步数据，所以，在节点之间会有大量 IO 。

##### 镜像集群模式

和普通集群模式不同，镜像集群模式会在每个节点保存 queue 的数据。当生产者往 MQ 的一个节点写一条数据，MQ 会同步到所有节点，这样消费者可以连到任意一个节点进行消费，这种模式可以保证高可用，但是单机存储会有瓶颈。

#### GUI

RabbitMQ 提供了一个管理图形界面，通过 [http://localhost:15672/](http://localhost:15672/) 可以访问，默认用户名密码是 guest/guest 。



### RocketMQ

#### 简介

RocketMQ 是由阿里巴巴开源的消息中间件。

#### 特性

RocketMQ 支持同步发送、异步发送、顺序发送、单向发送。

RocketMQ 支持推和拉两种消费方式。

RocketMQ 有组的概念，Producer Group 和 Consumer Group 。Producer Group 中的 Producer 发送的消息类型和逻辑是相同的，消息发送失败可由同组的其他 Producer 重试。Consumer Group 中的 Consumer 订阅相同的 Topic ，消费相同类型的消息。

RocketMQ 支持广播消息和集群消息。集群消息模式下，相同 Consumer Group 的 Consumer 实例平均分摊消息。广播消息模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

RocketMQ 支持标签 Tag ，用于区分相同 Topic 下的不同类型消息。Consumer 可以根据 Tag 进行过滤，消息过滤在 Broker 端完成。

RocketMQ 支持严格保证消息有序。消息有序分全局有序和分区有序。全局有序指 Topic 中的所有消息是有序的。分区有序指一组消息的顺序是有序的。

#### 架构

RocketMQ 主要由 Producer 、Consumer 、Broker 和 NameServer 组成。

NameServer 是 Topic 的路由和注册中心，支持 Broker 的注册与发现。Broker 会向 NameServer 注册，NameServer 通过心跳检查 Broker 是否存活。Producer 和 Consumer 从 NameServer 查询 Broker 的信息以完成消息发送和消费。

NameServer 是无状态的，可以部署多台，彼此之间相互独立。Broker 会向每一台 NameServer 进行注册，Producer 和 Consumer 可以连接任意一台 NameServer 。

Broker 主要用于存储消息，查询消息，投递消息等。主要包含以下模块：

1. Remoting Module：处理 clinet 的请求。
2. Client Manager：管理 Producer 和 Consumer 。
3. Store Service：提供消息存储和查询 API 。
4. HA Service：提供 Master 和 Slave 之间的数据同步。
5. Index Service：根据 Message Key 对消息进行索引，以便快速查询。

Broker 为主从架构，一个 Master 可对应多个 Slave 。Producer 只能写 Master ，Consumer 可以从 Master 或 Slave 上读。

#### 存储

![https://gitee.com/bsyonline/pic/raw/master/20210519095716.png](https://gitee.com/bsyonline/pic/raw/master/20210519095716.png)

RocketMQ 的消息存储在 CommitLog 中，顺序写，写满一个在写下一个。在读的时候为随机读。

为了提高消费的效率，会将元数据（消息在 CommitLog 中的 offset 、消息的大小、tag 的 hashcode）保存到逻辑消费队列 ConsumerQueue ，ConsumerQueue 相当于 CommitLog 的索引。ConsumerQueue 是顺序读，利用了 PageCache 和 Mmap ，所以读的效率很高。

IndexFile 可以通过 key 或时间来查询消息，低层通过 HashMap 实现。

<img src="https://gitee.com/bsyonline/pic/raw/master/rocketmq_design_2.png"/>

RocketMQ 支持同步刷盘和异步刷盘。同步刷盘在消息存储到磁盘上才会返回 ack ，可靠性高，性能不高。异步刷盘消息写入 PageCache 中就返回 ack ，后台异步线程将数据写到磁盘，性能和吞吐都很高。

#### 安装

1. 解压

```
unzip rocketmq-all-4.4.0-bin-release.zip
mv rocketmq-all-4.4.0-bin-release rocketmq
```

2. 修改配置文件

RocketMQ 提供了多种配置：

1. 2m-2s-async 2主2从异步刷盘
2. 2m-2s-sync 2主2从同步刷盘
3. 2m-noslave 2主
4. broker.conf 单机

这里使用单机部署。

```
brokerClusterName = rocketmq-cluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
defaultTopicQueueNums=4
messageDelayLevel=1s 2s 3s 4s 5s 6s 7s 8s 9s 10s 11s 12s 13s 14s 15s 16s 17s 18s
storePathRootDir=/home/rocketMQ/store
storePathCommitLog=/home/rocketMQ/store/commitlog
```

创建存储路径

```
mkdir -p /home/rocketMQ/store
mkdir -p /home/rocketMQ/store/commitlog
```

vim runbroker.sh  vim runserver.sh

3. 启动

```
nohup /opt/rocketmq/bin/mqnamesrv > namesrv.log &
nohup /opt/rocketmq/bin/mqbroker -n localhost:9876 -c /opt/rocketmq/conf/broker.conf > broker.log &
```

4. 测试

发送消息

```
export NAMESRV_ADDR=127.0.0.1:9876
sh /opt/rocketmq/bin/tools.sh rg.apache.rocketmq.example.quickstart.Producer
```

消费消息

```
export NAMESRV_ADDR=127.0.0.1:9876
sh /opt/rocketmq/bin/tools.sh rg.apache.rocketmq.example.quickstart.Consumer
```

#### 命令

创建 Topic

```
bin/mqadmin updateTopic -n localhost:9876 -b localhost:10911 -t world
```

发消息

```
bin/mqadmin sendMessage -n localhost:9876 -b localhost:10911 -t world -p test
```

消费消息

```
bin/mqadmin consumeMessage -n localhost:9876 -b localhost:10911 -g group1 -t world
```



#### 自动创建 Topic 流程

**step 1：broker 启动时注册路由信息**

brokerA 启动时，broker 会创建默认topic TBW102 注册到 nameserv ，并且每隔 30 秒通过心跳拉取 nameserv 上的路由信息，更新本地缓存的路由信息。

brokerB 启动时，broker 也会创建默认topic TBW102 注册到 nameserv ，并且每隔 30 秒通过心跳拉取 nameserv 上的路由信息，更新本地缓存的路由信息。

此时 nameserv 上的路由信息为 TBW102=[brokerA, brokerB] 。

**step 2：producer 发消息**

producer 向 topic=Topic_01 发消息时，会先从本地缓存取 Topic_01 的路由信息，结果是取不到，然后就会去 nameserv 取，还是取不到，最后就去 nameserv 取默认 topic 也就是 TBW102 的路由信息作为 Topic_01 的路由信息，并缓存到本地。

此时，producer 本地缓存中的路由信息为 Topic_01=[brokerA, brokerB] ，然后通过路由策略选择一个 broker 发送消息。假定选择 brokerA 。

**step 3：broker 接收消息**

消息发送到 brokerA 后，brokerA 会 checkMsg ，如果本地没有 Topic_01 的信息，并且 rocketmq 开启了 autoCreateTopicEnable=true ，那么 brokerA 会按照默认 topic 的配置创建新的 topic Topic_01 。

假定没有其他操作，Topic_01 已经同步到 nameserv ，此时 nameserv 上的路由信息为 Topic_01=[brokerA] 。

**step 4：producer 再次发消息**

producer 也有心跳定时从 nameserv 获取路由信息，此时 producer 的 topicList=[TWB102, Topic_01] ，一个心跳周期后，producer 本地缓存就变成了 Topic_01=[brokerA] 。

**step 5：producer 再次发消息**

producer 再次向 topic=Topic_01 发消息时，先从本地缓存取 Topic_01 ，只会取到 brokerA ，所以就只会向 brokerA 发消息。

### Kafka

#### 简介

Kafka 是一个开源的、分布式的、基于发布订阅模式的消息中间件，广泛应用于日志收集和大数据领域。

#### 安装

下载

```
tar -xzf kafka_2.12-2.8.0.tgz
mv kafka_2.12-2.8.0 kafka
```

配置修改 config/server.properties 。

```properties
broker.id=0

num.network.threads=3

num.io.threads=8

socket.send.buffer.bytes=102400

socket.receive.buffer.bytes=102400

socket.request.max.bytes=104857600

log.dirs=/tmp/kafka-logs

num.partitions=1

num.recovery.threads.per.data.dir=1

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

log.retention.hours=168
message.max.byte=5242880
default.replication.factor=1
replica.fetch.max.bytes=5242880

log.segment.bytes=1073741824

log.retention.check.interval.ms=300000

zookeeper.connect=hadoop1:2181,hadoop2:2181,hadoop3:2181

zookeeper.connection.timeout.ms=18000

group.initial.rebalance.delay.ms=0
```

讲 kafka 复制到其他节点。

```
scp -r ./kafka/ hadoop2:/opt/
scp -r ./kafka/ hadoop3:/opt/
```

修改 server.properties 文件的 broker.id 。

启动

Kafka 启动需要用到 Zookeeper ，需要提前搭好。

```
./bin/kafka-server-start.sh -daemon ./config/server.properties
```

测试

创建 topic

```
./bin/kafka-topics.sh --create --zookeeper hadoop1:2181 --replication-factor 1 --partitions 1 --topic test_01
```

查看 topic

```
./bin/kafka-topics.sh --list --zookeeper hadoop1:2181
```

发送消息

```
./bin/kafka-console-producer.sh --broker-list hadoop1:9092 --topic test_01
```

消费消息

```
./bin/kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic test_01 --from-beginning
```

7. 查看消费进度

```
./bin/kafka-consumer-groups.sh --bootstrap-server hadoop1:9092 --describe --all-groups
```

8. 删除 topic

```
./bin/kafka-topics.sh --zookeeper hadoop1:2181 --delete --topic test_01
```

#### 术语

**Producer**

消息的发送者，Producer 向 Broker 发送消息，每条消息都属于一个 Topic 。每个 Topic 包含一个或多个 Partition 。

**Consumer**

消息消费者。

**ConsumerGroup**

ConsumerGroup 是一种可扩展的消费者机制，一个 ConsumerGroup 可以有多个消费者，这些消费者具有相同的 groupId ，同组的消费者会平均消费 topic 的 partition 。一条消息只能被同一个 consumerGroup 中的一个 consumer 消费，但是可以被多个 consumerGroup 的不同 consumer 消费。如果 consumer 的数量大于 partition 的数量，那么会有 consumer 是空闲的。

**Partition**

Partition 对应一个或多个目录，Partition 有多个副本，叫做 Replica 。Replica 分 Leader 和 Follower ，Producer 和 Consumer 与 Leader 交互，Follower 从 Leader 同步数据。Consumer 属于 ConsumerGroup ，Consumer 从 Partition 中消费消息，一条消息只能被一个 ConsumerGroup 中的一个 Consumer 消费。

partition 是一个 FIFO 队列，partition 内的消息是有序的，但是多个 partition 之间消息是无序的。

**Partition Leader**

Partition 有且只有一个 partition 来负责读写请求，这个 partition 就是 Leader 。

**Partition Follower**

Partition Follower 是 Leader Partition 的备份，只会从 Leader Partition 同步消息。Leader Partition 宕机后，会从 Follower 中选一个作为 Leader Partition 。

**ISR**

In-Sync Replicas，副本同步列表，由 Leader Partition 维护。如果 Follower Partition 和 Leader Partition 连接不上，Leader 会将 broker id 从 ISR 移到 OSR（Outof-Sync Replicas）。ISR + OSR = AR（Assingd Replicas）。

**Segment**

Kafka 消息顺序写入磁盘，并进行分段存储，每个日志段文件写满后，接着写下一个。每个日志段文件不会太大，以便快速加载到内存，提高读效率。每个日志段文件对应都有索引文件，默认每 4KB 记一个索引，查找时使用二分查找，复杂度为 O(logN) 。

**Offset**

每个 partition 都有一个自己的 offset ，它是相对于当前 partition 的第一条消息的偏移量。rebalance 之后会按照 offset 开始消费。Offset 会保存到 __consumer_offset 主题的 partition 中。默认有 50 个 partition ，写入 offset 时会按照 consumer id hash 对 50 取模计算写到哪个 partition 中。如果 consumer 的数量大于 50 ，则应该将 \_\_consumer_offset 的 partition 的数量调大。

**Broker Controller**

多个 broker 中选举出一个 leader 叫做 broker controller ，负责整个集群的 partition 和 replicas 的状态。ZooKeeper 负责 Broker Controller 的选举，Broker Controller 负责 Partition Leader 的选举。

**HW**

HighWatermark，高水位，表示 Consumer 可以消费的 partition 的最大偏移量。HW 用来保证 Kafka 中 Leader Partition 和 Follower Partition 之间消息的一致性。消息写入 Leader Partition ，不会立刻被消费，Consumer 只能消费到 HW 位置的消息。所有 Follower 都同步完成，HW 才会更新，Consumer 才能消费的到。  

**LEO**

Log End Offset，日志最后消息的偏移量。LEO 是每个 partition 自己维护的最后一个写入消息的偏移量。

**Coordinator**

Coordinator 是指运行在每个 broker 上的 group Coordinator 进程，用于管理 Consumer Group 中的成员的 offset 和 rebalance 。一个 Coordinator 可以管理多个 ConsumerGroup 。

**Rebalance**

当 Consumer Group 中的 Consumer 数量发生变化或 Topic 中的 partition 数量发生变化时，partition 和 consumer 的对应关系会（可能会）发生变化，这个过程叫做 rebalance 。rebalance 使 ConsumerGroup 及 broker 集群具有高可用性和伸缩性，但是在 rebalance 期间，整个 broker 集群不可用。

#### 原理

##### **生产消费过程**

Producer 发送消息过程如下：

1. Producer 向 broker 提交连接请求，broker 会返回 broker controller 的地址。
2. Producer 向 broker controller 发送请求，获取当前 topic 的所有 partition 的 leader 地址。broker 从 zk 中查找指定 topic 的所有 partition leader 信息返回给 producer 。
3. Producer 收到 leader partition 信息后，根据消息路由策略找到要发送的 leader partition 并发送消息。
4. Leader partition  收到消息，并将消息写到 log 中，然后通知 ISR 中的 follower partition 。Follower partition 从 leader partition 同步消息，并返回 ack 。
5. Leader partition  收到所有 follower partition  的 ack 后，修改 HW 。 然后 consumer 就可以消费到 HW 之前的消息。

消息的接收者，从 broker 读取消息。Consumer 的消费过程如下：

1. Consumer 向 broker 集群提交连接请求，broker 返回 broker controller 的地址。
2. Consumer 发送 topic 信息给 broker controller 。
3. Broker controller 会为 consumer 分配一个或多个 leader partition  ，并将 partition 的 offset 发送给 consumer 。
4. Consumer 从 offset 位置开始消费 leader partition 中的消息，消费完成后发送 ack 。
5. Leader partition 收到 ack 后更新 offset ，并保存到 __consumer_offset 中。

##### **消息路由策略**

如果指定了 partition 则直接写入到指定的 partition 。

如果没有指定 partition ，但是指定了 key ，则按照 key hash 对 partition 数量取模，确定 partition 。

如果 partition 和 key 均未指定，则轮询写入。

##### **HW 截断机制**

当原 Leader 宕机后再次恢复时，LEO 会退回到 HW 位置，然后从新的 Leader 重新同步数据。

##### **消息发送的可靠性**

通过 request.required.acks 参数进行设置。

0：异步发送，不需要返回 ack 。效率最高，可靠性最低。

1：同步发送。默认。producer 发消息给 broker ，leader partition 接到消息之后就返回 ack 。producer 收到 ack 后再发送下一条消息。如果 producer 没有收到 ack ，则会重发消息。

-1：同步发送。producer 发消息给 broker ，leader partition 接到消息之后向 follower partition 同步消息，所有 follower partition 都同步完成，再返回 ack 。producer 收到 ack 后再发送下一条消息。如果 producer 没有收到 ack ，则会重发消息。很少会出现消息丢失的情况，但是会存在消息重复接收。

##### **Leader Partition 选举**

Leader Partition 宕机后，broker controller 会从 ISR 中选一个 partition 作为 leader 。如果 ISR 没有 partition ，则必须等待 ISR 中有 partition 后才进行选举，此种策略可用性较低。通过 unclean.leader.election.enable=true ，来设置当 ISR 中没有 partition 时，选择一个未宕机的 partition 作为 leader 。

##### **什么情况下会出现重复消费？**

1. 在 consumer 消费完一条消息，准备还没有提交 offset 时，自动提交时间到了。
2. 在 consumer 消费完一条消息，准备还没有提交 offset 时，发生 rebalance ，其他 consumer 可能会重复消费该条消息。

##### **什么情况下会丢数据？**

1. 如果 request.required.acks 设置为 0 ，消息发送了，但是 broker 宕机，消息就丢了。
2. 如果 request.required.acks 设置为 1，消息发送给 leader partition ，leader partition 返回 ack 后，还没有同步到 follower partition ，leader partition 宕机，消息丢失。
3. 如果 request.required.acks 设置为 -1，消息发送给 leader partition，所有 follower partition 同步完成后返回 ack 给 leader partition ，leader partition 再返回 ack 给 producer ，大部分情况下不会丢消息。但是在极端情况下，broker 接收到消息，将消息写到 log 中，此时数据还在 oscache 中，尚未刷写到磁盘上，此时 broker 宕机，消息丢失。