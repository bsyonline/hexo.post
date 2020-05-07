---
title: CountDownLatch
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-06 19:52:48
thumbnail:
---

CountDownLatch 是 java.util.concurrent 包中提供的一个倒计时器工具类。基本用法如下：
```
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " task done");
                latch.countDown();
            }, "t" + i).start();
        }
        latch.await();
        System.out.println("all thread wake up");
    }
}
```
接下来我们来看看每一步都做了什么。
1. 初始化 CountDownLatch 。
```
public CountDownLatch(int count) {
	if (count < 0) throw new IllegalArgumentException("count < 0");
	this.sync = new Sync(count);
}
//
Sync(int count) {
	setState(count);
}
```
可以看到我们创建的计数 count 通过 setState 保存到 AQS 的 state 中。
2. await() 
首先判断 state ，因为是倒计数器，所以是判断 state 是不是等于 0 。如果不为 0 ，将包含主线程信息的 node 节点加入到 AQS 队列，然后再进行循环。第一次循环会再判断一下计数器是否到 0 了，如果还没有到 0 ，说明任务还没有就绪，那就将 node 的 waitStatus 标记为需要 signal 来唤醒。然后进行第二次循环，如果还没有到 0 ，则 park 主线程。
3. countDown()
每执行一次 countDown() 方法，会通过 CAS 将 state-1 ，直到 state=0 才执行 doReleaseShared() 。当最后一次 countDown() 执行，AQS 队列中有两个节点，一个是虚拟的 node 节点 head ，另一个是包含主线程信息的 node 节点。取出 head 节点，并且由于 waitStatus 是 signal ，会执行 unparkSuccessor() 。在 unparkSuccessor 中会 unpark head.next 即包含主线程信息的节点。

