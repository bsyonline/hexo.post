---
title: Eureka Server Self Preservation Mode
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:05:53
thumbnail:
---

Eureka Client 可以向 Eureka Server 注册信息，并通过心跳来维持注册信息有效，当 Eureka Client 进行显示的注销或一段时间内 Eureka Server 没有收到心跳时，Eureka Server 会删除掉该 Eureka Client 的注册信息。当发生网络分区时，Eureka Server 长时间无法收到 Eureka Client 的心跳信息，如果 3 次心跳异常数量超出阈值，会进入自我保护模式。目的是为了保护注册信息不会被 Eureka Server 删除而导致大面积注册信息失效，并且在 Eureka Server 之间同步时不会影响其他的 Server Node 。默认情况下，Eureka Server 会开启自我保护机制，当失效的服务数量大于 15% 时，会进入自我保护模式，直到恢复到阈值之上或自我保护被关闭。