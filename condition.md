---
title: Condition
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-05 11:00:33
thumbnail:
---

Condition 是 java.util.concurrent 包中提供的接口，Condition 可以用来阻塞和唤醒线程，如果有多个线程，还可以指定唤醒某个线程。具体用法我们可以看一个例子：
```
public class ConditionTest {

    public static void main(String[] args) throws InterruptedException {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();
        Thread t1 = new Thread(() -> {
            reentrantLock.lock();
            try {
                condition.await();
                System.out.println(Thread.currentThread().getName() + " wake up");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }, "t1");
        t1.start();
        Thread.sleep(1000);
        Thread t2 = new Thread(() -> {
            reentrantLock.lock();
            try {
                condition.await();
                System.out.println(Thread.currentThread().getName() + " wake up");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }, "t2");
        t2.start();
        Thread.sleep(1000);
        new Thread(() -> {
            reentrantLock.lock();
            condition.signalAll();
            reentrantLock.unlock();
        }, "t5").start();
    }
}
```
这个例子展示了 Condition 的用法，我们来看一下每一步都做了什么。
1. 首先 lock.newCondition() 返回的是 sync.newCondition() sync 继承自 AQS ，最终创建了一个 AQS 的 ConditionObject() 对象。ConditionObject 中包含了两个引用 firstWaiter 和 lastWaiter ，其类型是 AbstractQueuedSynchronizer.Node ，Node 中又包含了一个指针 nextWaiter ，这样就可以来构建一个链表。
2. 当调用 await() 方法时，先创建一个包含当前线程的 node 节点 n1 ，然后如果 condition queue 的 lastWaiter 节点 t 为空，说明condition queue 中没有节点，就将 firstWaiter 节点指向 n1 ，否则就将 lastWaiter.nextWaiter 指向 n1 。
<img src="https://s2.ax1x.com/2020/03/06/3bd2sf.png" alt="20200306102035" border="0" style="width:500px">
3. 当调用 signal() 方法时，从 condition queue 中取出 firstWaiter 节点 n1 执行 doSignal() 。在 doSignal() 中，先将 firstWaiter 指向 n1.next 节点，如果 n1.next 为空，就将 lastWaiter 置空。然后将 condition queue 的节点 n1 取出来，加入到 AQS 队列中。
<img src="https://s2.ax1x.com/2020/03/06/3bdRL8.png" alt="20200306101634" border="0" style="width:500px">
4. 当执行 unlock() 方法，就可以从 AQS 队列中执行出队操作，从而 unpark 线程。

>signal() 和 signalAll() 的区别：
signal() 方法是从 condition queue 中取出 firstWaiter 节点执行 transferForSignal() 操作。signalAll() 方法是遍历 condition queue 执行 transferForSignal() 直到 firstWaiter 为空。