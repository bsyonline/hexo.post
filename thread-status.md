---
title: 线程的状态
date: 2016-07-23 21:50:03
tags:
  - Interview
category:
  - Concurrent
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

参考 jdk 源码 java.lant.Thread.State 。Java 的线程有以下几种状态：
* NEW
  线程未启动。
* RUNNABLE
  线程在 JVM 中运行，也可能在等待其他资源。
* BLOCKED
  线程阻塞并等待锁，在调用 wait() 方法后进入或重新进入同步方法。
* WAITTING
  线程在执行特定的方法如： Object.wait(), Thread.join(), LockSupport.park() 后等待另一个线程执行特定的方法，如：Object.notify()/Object.notifyAll() 。
* TIMED_WAITTING
  线程在执行特定方法后进入执行时间的等待状态，如： Thread.sleep(), Object.wait(), Thread.join(), LockSupport.parkNanos, LockSupport.parkUntil
* TERMINATED
  线程执行完成。
