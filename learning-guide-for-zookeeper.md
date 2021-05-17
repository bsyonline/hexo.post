---
title: learning guide for zookeeper
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-09 12:01:55
thumbnail:
---



#### 简介

ZooKeeper 是一个开源的分布式协调服务。ZooKeeper 基于 Paxos 的 ZAB 协议提供一致性服务。

#### 安装



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

zkServer 状态有 3 中模式：

恢复模式：集群启动或 Leader 宕机时，集群进入恢复模式。恢复模式包括 Leader 的选举和初始化同步两个阶段。

广播模式：分为初始化广播和更新广播。

同步模式：分为初始化同步和更新同步。

状态

LOOKING：选举状态。

FOLLOWING：Follower 正常工作或从 Leader 同步数据。

OBSERVING：Observer 正常工作或从 Leader 同步数据。

LEADING：Leader 正常工作或广播数据更新。

#### 应用场景

1. 配置维护
2. 命名服务
3. DNS 服务
4. Master 选举
5. 分布式同步
6. 集群管理
7. 分布式锁
8. 分布式队列