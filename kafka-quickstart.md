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

### 概念

Kafka 由 Producer 、Broker 、Consumer 组成，需要通过 ZooKeeper 完成注册发现。Producer 向 Broker 发送消息，每条消息都属于一个 Topic 。每个 Topic 包含一个或多个 Partition 。Partition 有多个副本，叫做 Replica 。Replica 分 Leader 和 Follower ，Producer 和 Consumer 与 Leader 交互，Follower 从 Leader 同步数据。Consumer 属于 ConsumerGroup ，Consumer 从 Partition 中消费消息，一条消息只能被一个 ConsumerGroup 中的一个 Consumer 消费。

Kafka 消息顺序写入磁盘，并进行分段存储，每个日志段文件写满后，接着写下一个。每个日志段文件不会太大，以便快速加载到内存，提高读效率。每个日志段文件对应都有索引文件，默认每 4KB 记一个索引，查找时使用二分查找，复杂度为 O(logN) 。