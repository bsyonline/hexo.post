---
title: spark-hardware-provisioning
tags:
  - Spark
category:
  - Spark
author: bsyonline
lede: 没有摘要
date: 2017-12-26 10:59:07
thumbnail:
---

### 存储系统

Spark 任务会从外部存储系统中读取数据，所以将它放在尽可能近的地方是非常重要的。一般来说 Spark 程序和数据放在相同节点是效率最高的，如果无法保证相同节点，退一步讲，应将程序和数据放在同一网络环境下。

#### 本地磁盘

虽然 Spark 是基于内存的运算，但是在数据无法放置到内存或保存 stage 间的中间数据还是需要使用磁盘。推荐每个节点 4-8 块磁盘，单独挂载不需要配置 RAID ，使用 noatime 选项减少不必要的写操作。 磁盘信息通过设置 spark.local.dir 参数配置。如果运行在 HDFS 上，可以使用和 HDFS 相同的配置。

#### 内存

Spark 可以使用 8 到上百 GB 的内存，但是推荐分配 75% 的内存给 Spark，其他的留给操作系统和缓存使用。JVM 在 200 GB 以上的机器上表现并不理想，如果机器内存超过 200 GB ，可以在节点上运行多个 worker JVMs 。

#### 网络

Spark 在运行期间是网络绑定的，使用万兆或更大的带宽程序会执行更快。对 group-bys，reduce-bys，joins 等分布式减少的程序尤其适用。

#### CPU

Spark 可以在每个节点上扩展到十多个 CPU 内核，因为它是线程之间最小共享的。