---
title: Spark 存储管理
date: 2016-05-20 11:50:54
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

将数据集持久化或缓存到内存里是 Spark 的重要能力之一。在持久化数据集时，在内存中参与计算的数据集被存储在每一个节点上，其他的 action 可以快速的重用这些数据集。缓存是迭代算法和快速交互的关键。

### 数据块



### 持久化数据集

使用 ```persist()``` 或 ```cache()``` 方法可以持久化数据集。Spark 会在第一次计算时将数据集保存在节点的内存中。如果任何一个数据集分区丢失，Spark 会自动使用创建改数据集的转换重新计算。

不同的数据集可以使用不同存储级别，可以自行选择。```persist()``` 可以使用 StorgaeLevel 对象设置存储级别，```cache()``` 就是使用的默认的存储级别，```StorageLevel.MEMORY_ONLY```（将序列化对象保存在内存中）。
<table class="table table-bordered table-striped table-condensed"><tr><th>Storage Level</th><th>Meaning</th></tr><tr><td>MEMORY_ONLY</td><td>默认的存储级别，数据集以非序列化 Java 对象保存在 JVM 中。如果数据集不能放到内存里，则一写分区不会被缓存，而是在每次使用时重新计算。</td></tr><tr><td>MEMORY_AND_DISK</td><td>数据集以非序列化 Java 对象保存在 JVM 中。如果分区不能放到内存中则放到磁盘上，每次使用时从磁盘读取。</td></tr><tr><td>MEMORY_ONLY_SER  (Java and Scala)</td><td>数据集以序列化 Java 对象（字节数组）方式保存。这种方式相对于非序列化对象来说更节省空间，但同时反序列化也更耗 CPU 。</td></tr><tr><td>MEMORY_AND_DISK_SER  (Java and Scala)</td><td>和 MEMORY_ONLY_SER 类似，但是会将无法放到内存中的分区保存到磁盘而不是重新计算。</td></tr><tr><td>DISK_ONLY</td><td>保存数据集到磁盘。</td></tr><tr><td>MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc. </td><td>和上边类似，但是会在两个集群节点之间复制分区，保证有 2 份拷贝。</td></tr><tr><td>OFF_HEAP (experimental)</td><td>和 MEMORY_ONLY_SER 类似，但是会将数据保存在 [off-heap memory](https://spark.apache.org/docs/2.1.1/configuration.html#memory-management)。 </td></tr></table>

那如何选择存储级别呢？

Spark 的存储级别是为了在内存使用和 CPU 效率之间达到平衡。可以通过以下步骤来选择：

* 数据集可以全部缓存到内存中，使用默认存储级别会获得最佳性能
* 否则，使用 ```MEMORY_ONLY_SER``` 并选择一个高效的序列化库
* 通常不要将数据集持久化到磁盘上，除非重新计算数据集开销大于从磁盘读取数据
* 使用复制的存储级别可以在出错时快速恢复而不用等待计算丢失的分区

对于持久化的数据，Spark 会自动监控每个节点的缓存，按照最近被使用算法删除旧的分区。

### 持久化 shuffle

Spark 会在执行 shuffle 操作时自动持久化一些中间数据。这样可以避免 shuffle 失败时重新计算整个输入。如果确定结果会被重用，建议对数据集进行持久化。

### 共享变量

通常，函数传递到 Spark 集群节点执行，函数中使用所有变量的独立副本，这些变量被拷贝到每一个节点，集群节点上的变量变更不会影响驱动程序，因为读写共享的变量访问时低效的。但 Spark 提供了两种有限类型的共享变量：广播变量和累加器。

#### 广播变量

广播变量允许程序保持一份只读的变量缓存在机器上，而不是把变量拷贝到任务中。Spark 的 action 是分阶段执行的，每个阶段任务需要使用的数据被自动广播。广播数据以序列化的形式缓存并在任务执行前反序列化。但多个阶段的任务需要相同的数据时，广播变量更加有效。当广播变量创建后，会在方法中代替变量值，以保证不会被多次发送。广播变量在发送到节点后不能再被修改。

