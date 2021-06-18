---
title: JVM Options
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-02-21 12:39:48
thumbnail:
---

JVM 的选项有 3 种：
1. 基本选项，如 -version。
2. 非基本选项，即 -X 选项，如 -Xmx，-Xms，-Xmn 等。
3. 高级选项，即 -XX 选项。高级选项有两种类型，一种是 boolean 类型的，用 -XX:+ 或 -XX:- 表示，如 -XX:+PrintGCDetails ，另一种是值类型，如 -XX:ParallelGCThreads=2 。

高级选项比较常用重要的有以下几个：
1. -XX:InitialHeapSize，初始堆内存大小，等价于 -Xms 。
2. -XX:MaxHeapSize，最大堆内存大小，等价于 -Xmx 。
3. -XX:NewSize，新生代初始内存，等价于 -Xmn 。
4. -XX:MaxNewSize，新生代最大内存。
5. -XX:ThreadStackSize，线程栈的内存大小，等价于 -Xss 。
6. -XX:SurvivorRatio=ratio，eden 和 survivor 的比例，默认为 8 ，即 eden:s0:s1 = 8:1:1 。
7. -XX:NewRatio=ratio，young 和 old 的比例，默认为 2 ，即 young 占 1/3 。
8. -XX:MaxTenuringThreshold=threshold，经过多少次 young GC 才能到 ParOldGen ，默认是 15 ，可以设置范围 0-15 。
9. -XX:+UseParallelGC，使用 ParallelGC 垃圾收集器。
10. -XX:+PrintCommandLineFlags，输出 JVM 选项。