---
title: Spark ExecutorLostFailure 错误简单分析
date: 2017-02-03 10:03:50
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

用 Spark 读 HBase 的 4400 万数据的表时，出现了 ExecutorLostFailure 错误，如果读一个 100 万的数据表则正常执行。[网上资料](https://www.zybuluo.com/xtccc/note/254078)说它常常出现在数据量很大，特别是shuffle的数据量很大，或者 executor 内存比较小的时候。
我的环境 spark 1.5 ，由 4 个节点组成的 spark standalone 集群，每个 100G 内存，24 个 cores ，在运行的时候显示可用内存不到 50G （看资料说 spark 会将可用的内存的 50% 用于执行，50% 用于存储）。整个 4400 万数据全部加载到内存的话，大概会占用 40-50G 的存储。所以猜想是由于 executor 的内存不够，在查看 spark 日志也发现 java.lang.OutOfMemoryError: Java heap space 信息，所以基本可以确定是内存过小的缘故。

在寻找解决办法时，发现[网络资料](https://my.oschina.net/tearsky/blog/629201)有如下说明：
>由于我们在执行 Spark 任务是，读取所需要的原数据，数据量太大，导致在 Worker 上面分配的任务执行数据时所需要的内存不够，直接导致内存溢出了，所以我们有必要增加 Worker 上面的内存来满足程序运行需要。
>在 Spark Streaming 或者其他 Spark 任务中，会遇到在 Spark 中常见的问题，典型如 Executor Lost 相关的问题 ( shuffle fetch 失败，Task 失败重试等 )。这就意味着发生了内存不足或者数据倾斜的问题。
>这个目前需要考虑如下几个点以获得解决方案：

>A、相同资源下，增加 partition 数可以减少内存问题。 原因如下：通过增加 partition 数，每个 task 要处理的数据少了，同一时间内，所有正在运行的 task 要处理的数量少了很多，所有 Executor 占用的内存也变小了。这可以缓解数据倾斜以及内存不足的压力。

>B、关注 shuffle read 阶段的并行数。例如 reduce, group 之类的函数，其实他们都有第二个参数，并行度 ( partition 数 )，只是大家一般都不设置。不过出了问题再设置一下，也不错。

>C、给一个 Executor 核数设置的太多，也就意味着同一时刻，在该 Executor 的内存压力会更大，GC 也会更频繁。我一般会控制在 3 个左右。然后通过提高 Executor 数量来保持资源的总量不变。

所以按照 A 方案对程序进行了修改：
1. 将 HBase 表按 rowkey 进行划分，大概分成了 30 份；
2. 按 startRow 和 stopRow 分别读取转换成 RDD；
3. 将 30 个 RDD 进行 union 操作合并成一个 RDD，在进行运算。

另外，针对 spark-submit 的参数也进行了相应的调整:
```
--executor-memory 90G
--driver-memory 95G
--executor-cores 3
```
每台机器有 24 个 CPU ，如果不指定 executor-cores , 将会使用所有的 cores ，这样会因 GC 造成比较大的内存压力，在尝试将 executor-cores 降低后，内存溢出明显减少。

这样修改以后，程序正常执行。
