---
title: Learning Guide for ZooKeeper
tags:
  - zookeeper
category:
  - bigdata
author: bsyonline
lede: 没有摘要
date: 2021-05-09 12:01:55
thumbnail:
---



#### 简介

ZooKeeper 是一个开源的高性能分布式协调服务。ZooKeeper 基于 Paxos 的 ZAB 协议提供一致性服务。

#### 安装

[installation guide for zookeeper](../../../../2015/07/14/installation-guide-for-zookeeper)

#### 选举机制

##### Paxos 算法

Paxos 是一个基于消息传递的一致性算法。Paxos 的前提条件是要求计算环境是可信的，即不存在拜占庭将军问题（计算机硬件错误和网络攻击可能会影响投票结果）。

三种角色：Proposer 提案者、Acceptor 表决者、Learner 学习者。

Paxos 算法分为两个阶段：prepare 阶段和accpet 阶段。

##### ZAB

ZAB（ZooKeeper Atomic Broadcast）协议是 Paxos 算法的一种实现，用来实现分布式数据一致性，以支持 ZooKeeper 的崩溃恢复。ZooKeeper 中也有三种角色：Leader 、Follower 和 Observer 。

Leader 能处理读写请求和选举。

Follower 只能处理读请求，写请求会转给 Leader 。Leader 节点挂了，Follower 能够参与选举和投票。

Observer 没有选举权，其他功能和 Follower 一致。

zxid 32 位的 epoch + 32 位 xid

epoch 每个 Leader 的标号，用于区分不同的时期

xid 事务id ，流水号

#### 模式

zkServer 状态有 3 种模式：

恢复模式：集群启动或 Leader 宕机时，集群进入恢复模式。恢复模式包括 Leader 的选举和初始化同步两个阶段。

广播模式：分为初始化广播和更新广播。

同步模式：分为初始化同步和更新同步。

#### 状态

LOOKING：选举状态。

FOLLOWING：Follower 正常工作或从 Leader 同步数据。

OBSERVING：Observer 正常工作或从 Leader 同步数据。

LEADING：Leader 正常工作或广播数据更新。

#### 存储结构

ZooKeeker 的存储结构是一个树形结构，每个节点是一个 znode ，每个 znode 都可以存储数据。

#### znode 节点类型

有 4 种节点类型：

1. PERSISTENT 持久节点：创建后一直存在，除非主动删除。
2. PERSISTENT_SEQUENTIAL 持久顺序节点
3. EPHEMERAL 临时节点：生命周期和客户端连接相同。临时节点只能是叶子节点，不能有子节点。
4. EPHEMERAL_SEQUENTIAL 临时顺序节点

#### 客户端连接 zkServer 流程

不管使用 zkClient 还是 curator ，连接大致相同。

1. 首先创建 ZooKeeper 对象。
2. 解析连接字符串，在 StaticHostProvider 中用 Collections.shuffle() 将 server 顺序打乱，目的是做负载均衡。
3. 创建 ClientCnxn 对象，调用 start()  方法。
4. 在 start() 中启动 2 个线程 SendThread 和 EventThread 。SendThread 从 server 地址列表里轮询取出地址尝试连接。

#### 客户端连接状态

```java
CONNECTING, 		// 连接中
ASSOCIATING, 		// 关联
CONNECTED,  		// 已连接
CONNECTEDREADONLY, 	// 只读
CLOSED, 			// 连接关闭
AUTH_FAILED,  		// 鉴权失败
NOT_CONNECTED;  	// 未连接
```

#### 应用场景

1. 配置管理

   比如 HBase 用 zk 来管理集群配置信息，Kafka 用 zk 来管理 broker 信息。

2. 命名服务

   比如 DNS 服务

3. 分布式同步

   1. 分布式锁
   2. leader 选举

4. 集群管理

   比如 Dubbo 用 zk 做服务注册发现。Kafka 用 zk 做 consumer 端的重平衡。

   