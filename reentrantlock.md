---
title: ReentrantLock
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-03 21:49:32
thumbnail:
---

ReentrantLock 可以创建公平锁和非公平锁，默认是非公平锁。公平锁由 FairSync 实现，非公平锁由 NonfairSync 实现，它们都是 Sync 的子类，Sync 又是 AbstractQueuedSynchronizer 的子类。ReentrantLock 的基本用法如下：
```
ReentrantLock reentrantLock = new ReentrantLock();
reentrantLock.lock();
try {
	// todo
} finally {
	reentrantLock.unlock();
}
```
下面我们就来分析一下每一步都做了什么。
##### **加锁**
<img src="https://s2.ax1x.com/2020/03/05/3Hhs4x.png" alt="3Hhs4x.png" border="0" />
从代码来看，公平锁和非公平锁的 lock 都是执行 acquire() 方法，区别在于非公平锁在 acquire() 前先进行一次 CAS 操作，成功了就不执行 acquire() ， 不成功才执行 acquire() 。这个 acquire() 是父类 AbstractQueuedSynchronizer 中的方法。
<img src="https://s2.ax1x.com/2020/03/05/3H4Uit.png" alt="3H4Uit.png" border="0" style="width:350px" />
公平锁和非公平锁的 tryAcquire() 分别有各自的实现。
<img src="https://s2.ax1x.com/2020/03/05/3HRjbR.png" alt="3HRjbR.png" border="0" />
比较两个 tryAcquire() 方法发现，公平锁多了一个 hasQueuedPredecessors() 的判断。在公平锁中，当线程执行 tryAcquire() 时，如果可以进行加锁时，先执行 !hasQueuedPredecessors() 的判断。
<img src="https://s2.ax1x.com/2020/03/05/3HW3rj.png" alt="3HW3rj.png" border="0" style="width:450px"/>
如果当前线程之前有排队的线程则返回 true ，如果队列为空或队列的 head 就是当前线程则返回 false 。这是一个取反判断，所以当队列中有排队的线程，那么就不再执行 compareAndSetState() 操作了。换句话说就是如果有线程在排队，那么就不执行加锁的操作，直接判定加锁失败。这样就回到 AbstractQueuedSynchronizer 的 acquire() 中，继续执行 acquireQueued() 操作，即加入到 AQS 队列中。
相对于公平锁来说，非公平锁没有执行 hasQueuedPredecessors() 判断，如果可以进行加锁，则直接进行加锁，加锁成功返回 true ，加锁失败返回 false ，再执行 acquireQueued() 入队操作。
总结一下，**非公平锁比公平锁多了两次获取锁的机会**。

##### **入队**
入队操作包含两步：
1. addWaiter()
首先将当前线程封装成 Node 节点 n ，如果队列没有初始化，则通过 enq() 方法来初始化队列，会进行两次循环，第一次循环初始化一个空节点 t ，将 head 和 tail 都指向该 t ，第二次循环将 tail 指向 n ，将 t.next 指向 n ，这样就将线程加入到 AQS 队列中。
<img src="https://s2.ax1x.com/2020/03/05/3HjMWQ.png" alt="3HjMWQ.png" border="0" style="width:600px"/>
2. acquireQueued()
这个方法会判断线程是否会阻塞。判断逻辑放在 acquireQueued() 方法的死循环中。当线程加入到 AQS 队列中后，并不会马上阻塞，会在第一次循环中 再次尝试获取锁，如果获取锁失败，就将线程的 waitStatus 条件标记为需要 signal 来唤醒。第二次循环，还是获取锁失败之后，才真正 park 该线程，然后跳出循环。
<img src="https://s2.ax1x.com/2020/03/05/3HX0qP.png" alt="3HX0qP.png" border="0" />

##### **解锁**
解锁就很简单了，从 AQS 对列中获取 head.next 然后 unpark 。