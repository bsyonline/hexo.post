---
title: Netty IO Model
tags:
  - Interview
category:
  - Netty
author: bsyonline
lede: 没有摘要
date: 2020-01-04 19:21:18
thumbnail:
---

在传统的 JavaIO 网络编程中，我们使用的是阻塞 IO 模型，即一个线程只对应一个客户端，单个线程负责客户端的连接及读写事件。

<img src="https://s2.ax1x.com/2020/02/27/3axaVA.png" alt="3axaVA.png" border="0"  style="width:700px"/>

显而易见，阻塞 IO 模型具有诸多缺点：
1. 每个客户端都需要有一个线程，并且没有 IO 操作也不能释放，导致消耗大量的服务器资源；
2. 服务器资源有限，无法承载大量的客户端请求；
3. 线程间等待、唤醒，切换上下文会严重的消耗 cpu 资源。 

所以，针对阻塞问题，Java NIO 引入了事件驱动模型。

<img src="https://s2.ax1x.com/2020/02/27/3axdUI.png" alt="3axdUI.png" border="0" style="width:750px" />

在 NIO 模型中，使用 selector 来维护客户端和服务器之间的通道 channel ，当有事件触发，selector 根据 selectKey 找到对应的通道进行处理，这样解决了线程阻塞的问题，但是这个模型相对复杂，编程难度比较高。

Reactor 模式是事件驱动的一种实现，它包含两个部分：reactor 和 handler 。reactor 使用单线程循环监听连接事件，并将请求分发给相应的 handler 进行处理，处理完成通过回调程序返回结果。下图就是**基本 reactor 模型**。

<img src="https://s2.ax1x.com/2020/02/27/3axWan.png" alt="3axWan.png" border="0" style="width:750px" />

和 NIO 模型类似，Reactor 模型让多个客户端复用一个 acceptor ，减少了阻塞造成的线程资源浪费。在没有连接事件时，线程可以进行 handler 处理。 处理完成后，线程不用销毁，可以处理新的分发任务，提高系统的处理能力。虽然减少了线程连接时的阻塞消耗，但是一个线程无法同时处理 accept 和 handler ，所以这种基本模型不适合 handler 业务比较耗时的场景。
为了能够充分的利用系统资源，又对 reactor 模型进行了改进，这就是**单 reactor 多线程模型**，如下图所示。

<img src="https://s2.ax1x.com/2020/02/27/3ax6Kg.png" alt="3ax6Kg.png" border="0" style="width:800px" />

在多线程模式中，handler 只负责响应事件，收到事件后将任务分配给线程池中的工作线程，这样任务处理是异步的，处理完成后由回调函数返回结果，不会影响线程处理新的 accept 事件，这样就进一步减少了线程阻塞。但是在单 reactor 多线程模型中，由于 reactor 是既要处理连接，又要将请求分发给 handler ，所以请求量大时，会有性能瓶颈，因此对进一步改进出现了**主从 reactor 模型**。

<img src="https://s2.ax1x.com/2020/02/27/3axD8f.png" alt="3axD8f.png" border="0" style="width:800px" />

主从 reactor 模型是在单 reactor 多线程模型基础上，对 reactor 进行了功能区分，主 reactor 只负责处理 accept 事件，接收到请求后将连接分配给子 reactor 。 子 reactor 负责将事件分发给 handler，handler 再将事件分发给 worker 线程池中的线程来处理，处理结果由回调方法返回，这样就进一步提高了服务器的处理能力。

Netty 模型是在主从 reactor 模型的基础上演化而来的，如下图所示。

<img src="https://s2.ax1x.com/2020/02/27/3axgbj.png" alt="3axgbj.png" border="0"  style="width:550px"/>

BossGroup 相当于主 reactor， WorkerGroup 相当于子 reactor 。每个 group 中都有多个不断循环处理事件的线程 NioEventLoop 。BossGroup 只处理 accept 事件，将 WorkerGroup 中的 NioEventLoop 注册到 selector 。WorkerGroup 中的 NioEventLoop 负责处理读写事件。每个 NioEventLoop 中都维护了一个 pipeline ，用来维护 handler 。 

