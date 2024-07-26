---
title: Implementing RPC
tags:
  - RPC
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-06-23 23:05:04
thumbnail:
---



在上一篇 [Understanding Remote Procedure Call](../../../../2018/05/18/understanding-rpc/) 中，我们了解 rpc 的基本原理，本节我们通过实现一个 rpc 的 demo 来进一步理解 rpc 。

实现思路如下：

1. 首先我们有两个服务， OrderService 和 UserService 分别部署，OrderService 需要调用 UserService 。
2. rpc over tcp 。rpc 可以基于 http 也可以基于 tcp ，由于 http 是短链接，性能低于 tcp ，所以客户端和服务器通讯使用 tcp ，NIO 性能要优于 BIO ，但是直接使用 Java NIO 难度较大，所以通信框架可以使用 netty 。
3. 客户端和服务器通信效率除了协议的差异，还有一个重要因素是序列化和反序列化的效率。Java 提供了序列化库，但是效率不高。除此以外还有很多性能优秀的序列化/反序列化库，比如 protobuf，json，t'hrift，kryo 等等。pb 的性能和空间占用都很优秀的，并且是跨平台的，如果是 Java 平台，可以使用 protostuff 。
4. 注册中心。注册中心的作用是维护 service 和 地址映射，是一个 key/value，可以使用 redis 实现。单节点，不考虑高可用，探活，负载均衡等功能。
5. 使用代理来完成方法调用。

![](https://raw.githubusercontent.com/bsyonline/pic/master/20190624/rpc.png)

以上就是 rpc demo 的实现思路，代码详见 [https://github.com/bsyonline/microlabs/tree/develop/rpc](https://github.com/bsyonline/microlabs/tree/develop/rpc) 。



