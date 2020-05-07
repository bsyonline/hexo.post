---
title: RabbitMQ-Concepts-and-Glossary
tags:
  - RabbitMQ
category:
  - MQ
author: bsyonline
lede: 没有摘要
date: 2020-04-25 11:58:08
thumbnail:
---

#### 概念和术语

在学习 RabbitMQ 之前，先了解一下 RabbitMQ 的术语和基本概念。
1. AMQP
高级消息队列协议 (Advanced Message Queuing Protocol, AMQP) 是一个应用层协议的开放标准。RabbitMQ 实现了 AMQP 协议。
2. Publisher
向 MQ 发送消息的程序。
3. Message
Publisher 通过 MQ 传递给 Consumer 的数据。
4. Consumer
从 MQ 接收消息的程序。
5. Connection
应用程序和 RabbitMQ Broker 之间建立的 TCP 连接。
6. Queue
存放消息的队列。
7. Confirm
Publisher 向 RabbitMQ 发送 Message ，RabbitMQ 收到 Message 之后给 Publisher 的回执，用来标识 RabbitMQ 收到 Message 。
8. Message acknowledgment
消息回执，简称 ack 。Message 被 Consumer 消费之后，Consumer 向 MQ Server 发送一个 ack ，MQ Server 收到 ack 后会将 Message 删除。如果没有收到 ack 并且 Consumer 和 MQ Server  断开连接，MQ Server 会将 Message 发送给其他的 Consumer 处理。
9. Message durability
消息持久化，用来保证 MQ Server 在异常宕机时消息不丢失。
10. Prefetch count
11. Exchange
在 RabbitMQ 中，Publisher 不会直接将消息发送到 Queue ，而是将消息发送到 Exchange ，再由 Exchange 将消息路由给 Queue 。
12. RoutingKey
发送消息时携带的 key 。
13. Binding
绑定就是将 Exchange 和 Queue 进行关联。 
14. BindingKey
BindingKey 是在 Exchange 和 Queue 绑定时指定的。BindingKey 并不是实际存在，而是为了和 RoutingKey 进行区分，实际和 RoutingKey 是一个东西。Exchange 在路由消息时，会将 RoutingKey 和 BindingKey 进行比较，如果相等或匹配，Exchange 就会将消息路由到对应的 Queue 。
15. Exchange Types
路由规则，有 4 种：fanuot 、direct 、topic 和 headers 。

#### 工作流程
了解了基本概念，再来看看 MQ 的执行过程。
<img src="https://s1.ax1x.com/2020/04/25/JyRABt.png" alt="JyRABt.png" border="0" style="width:700px"/>
1. RabbitMQ 已经运行，并且 Exchange 和 Queue 已经绑定。
2. Producer 连接到 RabbitMQ ，并发送一条消息，这个消息上带着一个 RoutingKey 。
3. RabbitMQ 接收到消息，并给 Producer 发送了 Confirm 消息。
4. Exchange 根据 RoutingKey 将消息路由到 Queue 。
5. Consumer 从 RabbitMQ 接收一条消息，处理完成后向 RabbitMQ 发送一条 Ack 。
6. RabbitMQ 收到 Ack 后删除消息，整个流程结束。