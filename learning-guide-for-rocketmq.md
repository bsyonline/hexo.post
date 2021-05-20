---
title: learning guide for rocketmq
date: 2021-05-07 08:39:20
tags:
---



### 简介

RocketMQ 是由阿里巴巴开源的消息中间件。

### 特性

RocketMQ 支持同步发送、异步发送、顺序发送、单向发送。

RocketMQ 支持推和拉两种消费方式。

RocketMQ 有组的概念，Producer Group 和 Consumer Group 。Producer Group 中的 Producer 发送的消息类型和逻辑是相同的，消息发送失败可由同组的其他 Producer 重试。Consumer Group 中的 Consumer 订阅相同的 Topic ，消费相同类型的消息。

RocketMQ 支持广播消息和集群消息。集群消息模式下，相同 Consumer Group 的 Consumer 实例平均分摊消息。广播消息模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

RocketMQ 支持标签 Tag ，用于区分相同 Topic 下的不同类型消息。Consumer 可以根据 Tag 进行过滤，消息过滤在 Broker 端完成。

RocketMQ 支持严格保证消息有序。消息有序分全局有序和分区有序。全局有序指 Topic 中的所有消息是有序的。分区有序指一组消息的顺序是有序的。

### 架构

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

### 存储

![https://gitee.com/bsyonline/pic/raw/master/20210519095716.png](https://gitee.com/bsyonline/pic/raw/master/20210519095716.png)

RocketMQ 的消息存储在 CommitLog 中，顺序写，写满一个在写下一个。在读的时候为随机读。

为了提高消费的效率，会将元数据（消息在 CommitLog 中的 offset 、消息的大小、tag 的 hashcode）保存到逻辑消费队列 ConsumerQueue ，ConsumerQueue 相当于 CommitLog 的索引。ConsumerQueue 是顺序读，利用了 PageCache 和 Mmap ，所以读的效率很高。

IndexFile 可以通过 key 或时间来查询消息，低层通过 HashMap 实现。

<img src="https://gitee.com/bsyonline/pic/raw/master/rocketmq_design_2.png"/>

RocketMQ 支持同步刷盘和异步刷盘。同步刷盘在消息存储到磁盘上才会返回 ack ，可靠性高，性能不高。异步刷盘消息写入 PageCache 中就返回 ack ，后台异步线程将数据写到磁盘，性能和吞吐都很高。

### 安装

#### 1. 解压

```
unzip rocketmq-all-4.4.0-bin-release.zip
mv rocketmq-all-4.4.0-bin-release rocketmq
```

#### 2. 修改配置文件

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

#### 3. 启动

```
nohup /opt/rocketmq/bin/mqnamesrv > namesrv.log &
nohup /opt/rocketmq/bin/mqbroker -n localhost:9876 -c /opt/rocketmq/conf/broker.conf > broker.log &
```

#### 4. 测试

发送消息

```
export NAMESRV_ADDR=127.0.0.1:9876
sh /opt/rocketmq/bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

消费消息

```
export NAMESRV_ADDR=127.0.0.1:9876
sh /opt/rocketmq/bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

