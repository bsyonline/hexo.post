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

