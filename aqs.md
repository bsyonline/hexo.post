---
title: AQS
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-03 19:54:20
thumbnail:
---

AQS(AbstractQueuedSynchronizer) 是 java.util.concurrent.locks 包中提供的同步框架。AQS 是一个抽象类，java.util.concurrent 中很多并发工具类都是基于 AQS 实现的。AQS 比较复杂，但是我们可以先比较概括的来描述一下 AQS 。简单来说，AQS 维护了一个 state 属性和一个由双向链表构成的同步队列。线程通过 CAS 操作 state 获取锁，获取锁失败的线程被构造成 Node 放到同步队列中阻塞。当锁被释放，再将队列中的线程唤醒来重新获取锁。AQS 中提供了很多获取和释放资源的操作，子类继承和扩展这些方法来完成具体的功能，AQS 比较抽象和难以理解，我们可以通过学习具体场景的类来学习 AQS 。

[ReentrantLock](../../../../2020/03/03/reentrantlock/)
[Condition](../../../../2020/03/05/condition/)
[CountDownLatch](../../../../2020/03/06/countdownlatch/)