---
title: CMS Garbage Collector
tags:
  - GC
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-02-23 12:00:21
thumbnail:
---

CMS 收集器执行过程分为 5 个阶段：
1. Initial Mark(Stop the World Event)
暂停较短时间，标记可达对象。
2. Concurrent Marking
应用不暂停，从 GC root 开始标记可达对象。
3. Remark(Stop the World Event)
查找在并发标记过程中遗漏的对象。
4. Concurrent Sweep
应用不暂停，清除不可达对象。
5. Resetting
为下一次收集做准备。

最初堆中 eden，survivor，old 都是空的。
<img src="https://s2.ax1x.com/2020/02/23/3l4vPe.png" alt="20200223122552" border="0" style="width:300px">
当 young GC 触发，存活的对象将从 eden 和 survivor 拷贝到另一个 survivor ，到达年龄阈值的对象晋升到 old 。

<img src="https://s2.ax1x.com/2020/02/23/3l4x8H.png" alt="20200223122351" border="0" style="width:500px">

young GC 完成后，eden 和一个 survivor 被清空。

<img src="https://s2.ax1x.com/2020/02/23/3l4z2d.png" alt="20200223125905" border="0" style="width:500px">
当 old 到达一定容量触发 CMS 。
1） 首先进行 **Initial Mark** 阶段，标记出可达对象。
2） 短暂暂停之后，进入 **Concurrent Marking** 阶段，应用程序继续执行，CMS 同时找到可达的对象。
3） 在 **Remark** 阶段，找到并发标记期间丢失的对象。
4） 进入 **Concurrent Sweep** 阶段，释放掉未被标记的对象。
5） 释放之后，进入 **Resetting** 阶段，一次 CMS GC 完成。


<img src="https://s2.ax1x.com/2020/02/23/3l4X5D.png" alt="20200223125952" border="0" style="width:500px">