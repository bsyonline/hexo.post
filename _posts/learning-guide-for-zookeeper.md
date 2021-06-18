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



### 简介

ZooKeeper 是一个开源的高性能分布式协调服务。ZooKeeper 基于 Paxos 的 ZAB 协议提供一致性服务。

### 安装

[installation guide for zookeeper](../../../../2015/07/14/installation-guide-for-zookeeper)

### 基本概念

#### The ZooKeeper Data Model

ZooKeeker 具有命名空间的概念，并且命名空间是有等级的。命名空间下的数据以一个树形结构存储，每个节点的路径都是用斜线分割的绝对路径，不能使用相对路径。每个节点都被称作 znode ，每个 znode 都可以存储数据，并维护一个 stat 结构，用来存储 version、acl 、timestamp 的信息。

##### znode 分类

1. 持久节点和顺序节点

   znode 节点分持久节点 PERSISTENT 和临时节点 EPHEMERAL 。持久节点创建后一直存在，除非主动删除。

2. 临时节点

   临时节点和 session 绑定。session 结束，临时节点会被删除。临时节点只能是叶子节点，不能有子节点。

3. 顺序节点

   znode 也可以是有序的，顺序以自动生成，以 10 位数字标识，最大到 2 的 31 次幂。

4. Container Node

   

5. TTL Node

##### znode stat 的构成

- **czxid** The zxid of the change that caused this znode to be created.
- **mzxid** The zxid of the change that last modified this znode.
- **pzxid** The zxid of the change that last modified children of this znode.
- **ctime** The time in milliseconds from epoch when this znode was created.
- **mtime** The time in milliseconds from epoch when this znode was last modified.
- **version** The number of changes to the data of this znode.
- **cversion** The number of changes to the children of this znode.
- **aversion** The number of changes to the ACL of this znode.
- **ephemeralOwner** The session id of the owner of this znode if the znode is an ephemeral node. If it is not an ephemeral node, it will be zero.
- **dataLength** The length of the data field of this znode.
- **numChildren** The number of children of this znode.

#### ZooKeeper Sessions



#### ZooKeeper Watches

一次性，触发之后就失效了，如果需要再次监听，需要重新注册。不适合监听频繁变化的内容。

变更事件由服务端发送给客户端，相同内容的监听是串行的，在一个监听过程完成之前，客户端不能创建新的相同的监听。

在一个监听事件被触发到下一个 watch 创建之间，节点的内容有可能会变化。比如想持续监听一个 znode 的值，如果 znode 的值变了，会触发事件，客户端收到事件后，再次创建新的 watch ，但是在新的 watch 创建之前 znode 的值的变化不会被监听。

对于通一个监听对象，监听只会触发一次。比如对一个节点 exist 和 getData 的监听，当节点被删除，只会触发 exist 的监听事件。

watcher 在客户端，回调逻辑在客户端保存。

#### Consistency Guarantees



### 基本操作

#### 常见问题和排错

### 选举机制

#### Paxos 算法

Paxos 是一个基于消息传递的一致性算法。Paxos 的前提条件是要求计算环境是可信的，即不存在拜占庭将军问题（计算机硬件错误和网络攻击可能会影响投票结果）。

三种角色：Proposer 提案者、Acceptor 表决者、Learner 学习者。

Paxos 算法分为两个阶段：prepare 阶段和accpet 阶段。

#### ZAB

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

#### 选举规则

先比较 zxid ，zxid 大的投票为 leader 。如果 zxid 相同，则 myid 大的投票为 leader 。

#### 选举过程

##### 集群启动时选举



1. node1 启动，首先投票给自己，然后将 proposal (epoch,xid) 广播给其他节点。
2. node2 启动，首先投票给自己，然后将 proposal (epoch,xid) 广播给其他节点。
3. node1 收到 node2 的 proposal ，集群刚启动 epoch+xid ，则比较 myid ，node2 的 myid 大，所以 node1 接受 node2 的 proposal ，并反馈给 node2 。
4. node 2 收到 node1 的 proposal ，集群刚启动 epoch+xid ，则比较 myid ，node2 的 myid 大，所以 node2 忽略 node1 的 proposal ，并反馈给 node1 。
5. 一轮投票后，汇总投票结果，node2 投票超过半数，则 node2 当选 leader 。
6. node3 启动，node2 通知 node3 ，leader 节点为 node2 。

##### leader 宕机时选举

<img src="https://gitee.com/bsyonline/pic/raw/master/20210521092055.png"/>

1. leader node2 宕机，所有 follower 节点状态改为 LOOKING 。
2. node1 先投自己，然后将 proposal 广播给其他节点。
3. node3 先投自己，然后将 proposal 广播给其他节点。
4. node1 收到 node3 的 proposal ，如果 node3 的 xid 大，node1 接受 node3 的 proposal ，然后将结果返回给 node3 。
5. node3 收到 node1 的 proposal ，如果 node3 的 xid 大，node3 忽略 node1 的 proposal ，然后将结果返回给 node1 。
6. 一轮投票后，汇总投票结果 node3 投票超过半数，则 node3 当选 leader 。



### 客户端

#### 连接 zkServer 流程

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

### 应用场景

1. 配置管理

   利用watch机制

   比如 HBase 用 zk 来管理集群配置信息，Kafka 用 zk 来管理 broker 信息。

2. 命名服务

   利用顺序节点自动生成唯一编号的特性。

3. 分布式同步

   1. 分布式锁

   2. leader 选举

      （1）多个客户端同时发起对同一个临时节点的创建请求，最终只有一个客户端创建成功，这个创建成功的客户端就是 Master ，其他客户端就是 Slave 。

      （2）所有 Slave 都想这个临时节点的父节点注册一个子节点列表的 watch 监听。一旦 Master 节点宕机，临时节点就会消失，zk 就会向所有 Slave 发送事件通知。Slave 在接收到事件后会执行回调，重新想这个父节点创建响应的临时节点。哪个客户端创建成功，则该客户端就是新的 Master 。

4. 集群管理

   比如 DNS 服务

   比如 Dubbo 用 zk 做服务注册发现。Kafka 用 zk 做 consumer 端的重平衡。

   