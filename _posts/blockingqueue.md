---
title: BlockingQueue
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-07 21:36:24
thumbnail:
---


BlockingQueue 是 java.util.concurrent 包中的接口，实现了 Collection 和 Queue 接口，提供了 3 类操作。
BlockingQueue 操作
<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Throws exception</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Special value</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Blocks</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Times out</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Insert</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">add(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">offer(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">put(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">offer(e, time, unit)</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Remove</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">remove()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">poll()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">take()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">poll(time, unit)</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Examine</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">element()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">peek()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">not applicable</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">not applicable</td></tr></table>
和普通的 queue 相比，BlockingQueue 满时 put 元素和 BlockingQueue 为空时 take 元素都会阻塞。下面我们就以 LinkedBlockingQueue 为例来看看具体实现。
首先在 LinkedBlockingQueue 中定义了一些属性。
<img src="https://s2.ax1x.com/2020/03/07/3jTjvd.png" alt="3jTjvd.png" border="0" style="width:400px"/>
通过这些属性定义，我们大概可以猜到元素在是以 Node 的形式保存在队列中，并且入队和出队操作都会用锁，元素个数也是通过 AtomicInteger 来保存，所以 LinkedBlockingQueue 是一个线程安全的队列。
接下来我们主要看看两个 block 操作：
1. put(e)
<img src="https://s2.ax1x.com/2020/03/07/3jqyM4.png" alt="3jqyM4.png" border="0" style="width:500px" />
put() 中有 3 处判断:
1) 当元素个数小于 capacity 则继续执行，如果等于 capacity 则执行阻塞 notFull 线程。
2) 添加完元素之后，如果元素个数小于 capacity 则唤醒 notFull 线程。
3) 释放锁之前元素个数为 c+1 ，如果 c==0 那么说明队列中有一个元素，所以唤醒 notEmpty 线程。
2. take()
<img src="https://s2.ax1x.com/2020/03/07/3jjU6U.png" alt="3jjU6U.png" border="0" style="width:350px" />
和 put(e) 类似，take() 方法也是有 2 处判断：
1) 当元素个数等于 0 ，则阻塞 notEmpty 线程。
2) 当元素个数大于 0 ，则唤醒 notEmpty 线程。
3) take 元素个数会减少，在释放锁之前做了 take 操作，所以释放锁之后队列实际是小于 capacity 的，所以唤醒 notFull 线程。

总结一下：
1. **LinkedBlockingQueue 有两把锁 putLock 和 takeLock ，有两个 condition 条件 notFull 和 notEmpty 。**
2. **元素添加或删除操作先获取锁，再操作链表。**
3. **如果队列满时阻塞 notFull 线程，反之则唤醒 notFull 线程。**
4. **如果队列中没有元素时，阻塞 notEmpty 线程，反之则唤醒 notEmpty 线程。**