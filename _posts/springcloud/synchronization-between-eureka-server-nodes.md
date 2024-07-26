---
title: Synchronization between Eureka Server Nodes
tags:
  - Spring Cloud
  - Microservices
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:06:05
thumbnail:
---


多个 Eureka Server 可以构成集群，集群搭建方式参考 [Building Eureka Server with Peer Awareness](../../../../2020/03/22/building-eureka-server-with-peer-awareness/) 。

Eureka Server 使用 peer to peer 的方式来同步数据，即每个 Server Node 都作为 Client 向其他 Server 注册信息。Eureka Server 收到 Eureka Client 的请求（注册，续约，注销）之后，会将自己当作 Client 向其他 Eureka Server 发送请求同步数据。多个 Eureka Server 之间进行数据同步，就可能会存在同一个 Eureka Client 的信息存在多份，Eureka Server 会通过 lastDirtyTimestamp 来获取最新的数据。如果这个过程出现了网络异常，数据同步失败，那么会在下一次心跳的时候再进行复制。当发生网络分区时， Eureka Server 依旧可以提供服务注册，如果大面积心跳接收失败，会进入自我保护模式。当网络恢复后，会自动恢复。

再来看下源码，就以 heartbeat 为例：
<img src="https://s1.ax1x.com/2020/03/22/85XrVI.png" alt="85XrVI.png" border="0" />
会创建一个 ReplicationTask ，通过 TaskProcessor 异步执行任务进行数据同步。如果响应 404 ，说明对方没有该注册信息，则向对方注册，同样也是任务放到线程队列由 TaskProcessor 来执行。如果信息的 timestamp 不一致，则根据 timestamp 判断是否进行 override 。
<img src="https://s1.ax1x.com/2020/03/22/8ISec8.png" alt="8ISec8.png" border="0" />