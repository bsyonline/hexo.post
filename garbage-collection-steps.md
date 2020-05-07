---
title: Garbage Collection Steps
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-02-23 08:34:38
thumbnail:
---

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)

堆被分成若干部分：YoungGen ，OldGen ，parmGen 。
<img src="https://s2.ax1x.com/2020/02/23/3l3SPA.png" alt="20200223093804" border="0"  style="width:400px">
上图是 hotspot 的示意图，oracle 为了整合多种 jvm 产品，在 java8 中使用 metaspace 替代了 PerGen 。

#### GC的过程
1. 最初 eden 、form 、to 都是空的，新创建的对象首先放到 eden 。
<img src="https://s2.ax1x.com/2020/02/23/3l1vUH.png" alt="20200223093404" border="0" style="width:400px">
2. 对象不断放入 eden ，当 eden 满时触发 minor GC 。
<img src="https://s2.ax1x.com/2020/02/23/3l1jVe.png" alt="20200223093458" border="0" style="width:400px">
将 eden 中的对象移动到 s0 ，然后清空 eden 。
<img src="https://s2.ax1x.com/2020/02/23/3l1ObD.png" alt="20200223093528" border="0" style="width:400px">
3. 新创建的对象继续放到 eden ，当 eden 满时触发第二次 miner GC 。将 eden 中的对象移动到 s1 ，同时将 s0 中的对象也移动到 s1 ，同时对象年龄 +1 ，最后将 eden 和 s0 清空。
<img src="https://s2.ax1x.com/2020/02/23/3l1LDO.png" alt="20200223093551" border="0" style="width:400px">
4. 当第三次 minor GC 时，eden 和 s1 中的对象会被移动到 s0 并且对象年龄 +1 ，同时将清空 eden 和 s1 ，如此往复。
<img src="https://s2.ax1x.com/2020/02/23/3l1qKK.png" alt="20200223093611" border="0" style="width:400px">
5. 当若干次 minor GC 之后，当对象的年龄到达阈值时，会移动到 OldGen 。
<img src="https://s2.ax1x.com/2020/02/23/3l1x5d.png" alt="20200223093639" border="0" style="width:400px">
6. 对象不断晋升到 OldGen ，当 OldGen 满时会触发 major GC 来清理 OldGen。