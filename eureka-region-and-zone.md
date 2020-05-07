---
title: Eureka Region and Zone
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:05:39
thumbnail:
---

Region 和 Zone 是 AWS 的概念，Region 可以理解为地区，Zone 可以理解为机房。参考 Eureka 的架构图：
<img src="https://s1.ax1x.com/2020/03/22/84w60K.png" alt="84w60K.png" border="0"  style="width:600px;"/>

us-east-1 为 Region ，us-east-1 下有 3 个 Zone : us-east-1c ，us-east-1d ，us-east-1e 。每个 Zone 中有 Eureka Server 和 Eureka Client ，Eureka Client 优先向同一个 Zone 中的 Eureka Server 注册，除非当前 Zone 中的 Eureka Server 不可用，才会向远端的 Zone 中的 Eureka Server 注册。Eureka Client 虽然只向各自 Zone 中的 Eureka Server 注册，但是 Eureka Server 之间会同步注册信息，所以 Eureka Client 之间依旧可以相互调用。
Practice 参考 [Building Eureka Server with Region and Zone](../../../../2020/03/22/building-eureka-server-with-region-and-zone/)