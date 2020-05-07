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

Kafka 快速开始：
1. 下载
```
wget https://www.apache.org/dyn/closer.cgi?path=/kafka/2.4.1/kafka_2.11-2.4.1.tgz
tar -xzf kafka_2.11-2.4.1.tgz
```
2. 配置
修改 config/server.properties 。
```
broker.id=0
port=9092
host.name=192.168.0.206
advertised.host.name=192.168.0.206
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/opt/kafka-logs				# Kafka 日志路径，重要
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=192.168.0.206:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
```  
Kafka 启动需要用到 Zookeeper ，需要提前搭好。
3. 启动
```
./bin/kafka-server-start.sh ./config/server.properties
```
4. 创建 topic
```
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test --replication-factor 1 --partitions 1 
```
创建成功之后可以查看。
```
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list 
```
5. 发送消息
```
./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
```
6. 消费消息
```
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```
7. 查看消费进度
```
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe
```
8. 删除 topic
```
./kafka-topics.sh --zookeeper localhost:2181 --delete --topic test
```

单个节点就搭建完成了，现在把它扩展成集群。