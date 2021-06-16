---
title: Kafka Quickstart
tags:
  - Kafka
category:
  - MQ
author: bsyonline
lede: 没有摘要
date: 2020-05-02 19:31:34
thumbnail:
---



### Kafka 简介

Kafka 是一个开源的、分布式的、基于发布订阅模式的消息中间件，广泛应用于日志收集和大数据领域。

### 安装

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

### 术语

#### Producer

消息的发送者，Producer 向 Broker 发送消息，每条消息都属于一个 Topic 。每个 Topic 包含一个或多个 Partition 。

#### Consumer

消息消费者。

#### ConsumerGroup

ConsumerGroup 是一种可扩展的消费者机制，一个 ConsumerGroup 可以有多个消费者，这些消费者具有相同的 groupId ，同组的消费者会平均消费 topic 的 partition 。一条消息只能被同一个 consumerGroup 中的一个 consumer 消费，但是可以被多个 consumerGroup 的不同 consumer 消费。如果 consumer 的数量大于 partition 的数量，那么会有 consumer 是空闲的。

#### Partition

Partition 对应一个或多个目录，Partition 有多个副本，叫做 Replica 。Replica 分 Leader 和 Follower ，Producer 和 Consumer 与 Leader 交互，Follower 从 Leader 同步数据。Consumer 属于 ConsumerGroup ，Consumer 从 Partition 中消费消息，一条消息只能被一个 ConsumerGroup 中的一个 Consumer 消费。

partition 是一个 FIFO 队列，partition 内的消息是有序的，但是多个 partition 之间消息是无序的。

#### Partition Leader

Partition 有且只有一个 partition 来负责读写请求，这个 partition 就是 Leader 。

#### Partition Follower

Partition Follower 是 Leader Partition 的备份，只会从 Leader Partition 同步消息。Leader Partition 宕机后，会从 Follower 中选一个作为 Leader Partition 。

#### ISR

In-Sync Replicas，副本同步列表，由 Leader 维护。如果 partition 和 Leader partition 连接不上，Leader 会将 broker id 从 ISR 移到 OSR（Outof-Sync Replicas）。ISR + OSR = AR（Assingd Replicas）。

#### Segment

Kafka 消息顺序写入磁盘，并进行分段存储，每个日志段文件写满后，接着写下一个。每个日志段文件不会太大，以便快速加载到内存，提高读效率。每个日志段文件对应都有索引文件，默认每 4KB 记一个索引，查找时使用二分查找，复杂度为 O(logN) 。

#### Offset

每个 partition 都有一个自己的 offset ，它是相对于当前 partition 的第一条消息的偏移量。rebalance 之后会按照 offset 开始消费。Offset 会保存到 __consumer_offset 主题的 partition 中。默认有 50 个 partition ，写入 offset 时会按照 consumer id hash 对 50 取模计算写到哪个 partition 中。如果 consumer 的数量大于 50 ，则应该将 \_\_consumer_offset 的 partition 的数量调大。

#### Broker Controller

多个 broker 中选举出一个 leader 叫做 broker controller ，负责整个集群的 partition 和 replicas 的状态。Zookeeper 负责 Broker Controller 的选举，Broker Controller 负责 Partition Leader 的选举。

#### HW

HighWatermark，高水位，表示 Consumer 可以消费的 partition 的最大偏移量。HW 用来保证 Kafka 中 Leader Partition 和 Follower Partition 之间消息的一致性。消息写入 Leader Partition ，不会立刻被消费，Consumer 只能消费到 HW 位置的消息。所有 Follower 都同步完成，HW 才会更新，Consumer 才能消费的到。  

#### LEO

Log End Offset，日志最后消息的偏移量。LEO 是每个 partition 自己维护的最后一个写入消息的偏移量。

#### Coordinator

Coordinator 是指运行在每个 broker 上的 group Coordinator 进程，用于管理 Consumer Group 中的成员的 offset 和 rebalance 。一个 Coordinator 可以管理多个 Consumer Group 。

#### Rebalance

当 Consumer Group 中的 Consumer 数量发生变化或 Topic 中的 partition 数量发生变化时，partition 和 consumer 的对应关系会（可能会）发生变化，这个过程叫做 rebalance 。rebalance 使 Consumer Group 及 broker 集群具有高可用性和伸缩性，但是在 rebalance 期间，整个 broker 集群不可用。

### 原理

#### 生产消费过程

Producer 发送消息过程如下：

1. Producer 向 broker 提交连接请求，broker 会返回 broker controller 的地址。
2. Producer 向 broker controller 发送请求，获取当前 topic 的所有 partition 的 leader 地址。broker 从 zk 中查找指定 topic 的所有 partition leader 信息返回给 producer 。
3. Producer 收到 partition leader 信息后，根据消息路由策略找到要发送的 partition leader ，并发送消息。
4. Partition leader 收到消息，并将消息写到 log 中，然后通知 ISR 中的 follower 。follower 从 leader 同步消息，并返回 ack 。
5. Partition leader 收到所有 follower 的 ack 后，修改 HW 。 然后 consumer 就可以消费到 HW 的消息。

消息的接收者，从 broker 读取消息。Consumer 的消费过程如下：

1. Consumer 向 broker 集群提交连接请求，broker 返回 broker controller 的地址。
2. consumer 发送 topic 信息给 broker controller ，broker controller 会为 consumer 分配一个或多个 partition leader ，并将 partition 的 offset 发送给 consumer 。
3. Consumer 消费 partition leader 中的消息，消费完成后发送 ack 。
4. Broker 收到 ack 后更新 offset ，并保存到 __consumer_offset 中。

#### 消息路由策略

如果指定了 partition 则直接写入到指定的 partition 。

如果没有指定 partition ，但是指定了 key ，则按照 key hash 对 partition 数量取模，确定 partition 。

如果 partition 和 key 均为指定，则轮询写入。

#### HW 截断机制

当原 Leader 宕机后再次恢复时，LEO 会退回到 HW 位置，然后从新的 Leader 重新同步数据。

#### 消息发送的可靠性

通过 request.required.acks 参数进行设置。

0：异步发送，不需要返回 ack 。效率最高，可靠性最低。

1：同步发送。默认。producer 发消息给 broker ，leader partition 接到消息之后就返回 ack 。producer 收到 ack 后再发送下一条消息。如果 producer 没有收到 ack ，则会重发消息。

-1：同步发送。producer 发消息给 broker ，leader partition 接到消息之后向 follower partition 同步消息，所有 follower partition 都同步完成，再返回 ack 。producer 收到 ack 后再发送下一条消息。如果 producer 没有收到 ack ，则会重发消息。很少会出现消息丢失的情况，但是会存在消息重复。