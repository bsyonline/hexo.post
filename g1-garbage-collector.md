---
title: G1 Garbage Collector
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-02-23 08:35:19
thumbnail:
---



### 什么是 G1

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)

Garbage-First (G1) 垃圾收集器是服务器端的垃圾收集器，适用于大内存多处理器的服务器。G1 的目标是替代 CMS ，和 CMS 相比，G1 具有 2 个优势：
1） G1 是压缩的垃圾收集器，可以避免内存碎片。
2） G1 支持垃圾收集暂停时间预测，时间可以由用户设置。

### 内存结构
和之前的垃圾收集器架构不同，G1 使用了另一种架构。

<img src="https://s2.ax1x.com/2020/02/23/3l6Bad.png" alt="3l6Bad.png" border="0" style="width:500px"/>

堆内存被分成若干大小相等的 region ，大小 1-32Mb 不等，数量大约 2000 个，每个 region 中都有 eden，survivor ，old ，humogous 和 unused 。G1 的内存分配是一种逻辑划分，不一定是连续的。

### GC 过程
1. Initial Mark(Stop the World Event)
  应用暂停，标记 survivor region （root region），这些 region 可能含有 old gen 的引用。
2. Root Region Scanning
  应用不暂停，扫描 root region 获取 old 的引用。
3. Concurrent Marking
  查找存活的对象。
4. Remark(Stop the World Event)
  完成存活对象的标记，使用的是 snapshot-at-the-beginning(SATB) 算法。
5. Cleanup(Stop the World Event and Concurrent)
  首先，统计存活对象和完全空闲的 region (Stop the World Event) 。第二步，清除记录的区域 (Stop the World Event) 。最后，将空白的 region 重置，然后加入到空闲列表 (Concurrent) 。
6. Copying(Stop the World Event)
  将存活的对象拷贝到 unused region 。