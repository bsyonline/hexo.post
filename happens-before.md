---
title: Understanding happens-before
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-10 10:20:51
thumbnail:
---

在学习 java.util.concurrent 的过程中，经常看到一个词就是 happens-before 。happens-before 表示了两个操作之间的顺序关系， A happens-before B 就是说 A 在 B 之前执行，则 A 的执行结果对 B 是可见的。为了方便 A happens-before B 可以用 hb(A,B) 表示。
##### **为什么需要 happens-before**
了解过 JMM 模型都知道，线程在操作主内存中的变量时并不会直接在主内存中修改，而是将主内存中的数据拷贝到自己的工作内存，在多线程环境下就会面临可见性的问题。还有就是我们的代码在编译的时候编译器会对代码进行指令优化，有可能改变指令的顺序，从而导致结果和预期不一致。这些问题是非常复杂的，并且没有办法枚举出每一种情况，面对这样的情况，于是提出了一些通用的规则，这就是 happens-before 。

##### ** happens-before 规则**

1. 如果线程 t1 对 A 解锁之后，线程 t2 又对 A 加锁，那么 t1 对 A 的操作对 t2 都是可见的。
2. 对于一个 volatile 变量 A ，如果线程 t1 修改了 A 之后，线程 t2 读取了 A ，那么 t1 的修改对 t2 是可见的。
3. 线程的 start() 在其他方法之前执行。
4. 如果线程 t1 成功执行 t1.join() 之后执行线程 t2 那么 t1 中的操作对 t2 都是可见的。
5. 如果线程 t1 执行了两个操作 A 和 B ，那么 A 中的操作对 B 都是可见的。 
6. happens-before 具有传递性，如果 hb(A,B) , hb(B,C) , 那么可以得到 hb(A,C) 。 
