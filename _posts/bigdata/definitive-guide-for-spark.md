---
title: Definitive Guide for Spark
tags:
  - Spark
category:
  - Bigdata
author: bsyonline
lede: 没有摘要
date: 2021-05-30 18:31:35
thumbnail:
---



## Part I. Spark Core

### 1. Spark 简介



### 2. Spark 安装



### 3. Spark 运行机制

Spark 程序是一组运行在集群上的独立进程，进程之间通过 SparkContext 协调。Spark 程序包含两部分：Driver 程序（SparkContext）和 Executor 。SparkContext 能连接多种类型的 Cluster Manager （spark standalone cluster、Mesos、YARN），虽然 cluster manager 的类型多样，但是 Spark 运行机制基本相同。Driver 程序由用户启动，通过资源调度模块和 Executor 通信。在连接到 Cluster Manager 后，Spark 获得集群 Worker 节点上的 Executor （负责计算和存储数据的进程），将需要运行的代码发送到 Executor ，然后 SparkContext 发送 Task 到 Executor 去执行。示意如图：

<img src="https://z3.ax1x.com/2021/06/01/2KYw9J.png" alt="234941814.png" border="0" />

每个 Spark 程序有自己的 Executor 进程，并且在程序生命周期内都是相互独立的，这也意味着不同的 Spark 程序之间无法共享数据，除非借助其他存储系统，如 HDFS 。

Spark 对于 Cluster Manager 是不可知的，只要能获取到 Executor 进程并能相互通信即可，所以更换Cluster Manager 并不需要修改程序。

在整个生命周期内，Worker 节点和 Driver 程序之间需要保持联通。

在距离 Worker 的机器上运行 Driver 程序会更有效率。

#### Local 模式

本地运行，可快速验证代码。

#### Spark Standalone 模式

Spark 提供的一种简单部署模式，通常用来进行测试。

#### YARN 模式

通过 YARN 来调度 Spark 程序所需要的资源。YARN 模式有两种模式来启动 Spark 程序：YARN Cluster 和 YARN Client 。在 Cluster 模式中，Driver 程序运行在 master 进程中，程序初始化完成后 Client 可以结束。在 Client 模式中，Driver 程序运行在 client 进程中，master 只负责向 YARN 请求资源。

<img src="https://z3.ax1x.com/2021/06/08/2sUHWq.png" alt="spark-on-yarn.png" border="0" />

YARN cluster：

1. Client 通过 YARN Client 向 Resource Manager 提交 Spark 程序，Resource Manager 在集群中启动一个 Spark Application Master ，注册到 Resource Manager 并初始化 SparkContext 。
2. Spark Application Master 通过 Resource Manager 在分配的 YARN 节点中启动 Container 。
3. 在 Container 中运行 Task 。

YARN client：

1.  Client 在本地初始化 SparkContext 。
2.  通过 YARN Client 在随机一个 YARN 节点上启动 Spark Application Master 。
3.  Spark Application Master 通过 Resource Manager 在分配的 YARN 节点中启动 Container 。
4.  在 Container 中运行 Task 。

#### 几种模式比较

##### 环境变量传递

SparkContext 在初始化过程中需要将环境变量传递给 Executor ，如何将环境变量设置到 Executor 的环境中，取决于 Executor 的启动方式。

* Local 模式在单机运行，不存在环境变量传递的问题。
* Spark Standalone 模式中，环境变量被封装在 org.apache.spark.deploy.Command 类中，由 Spark Master 通过 Actor 转发给合适的 Worker ，Worker 通过 ExecutorRunner 构建 java.lang.Process ，再传递给 java.lang.ProcessBuilder.environment 。
* YARN 模式中，环境变量首先通过 YARN client 设置到 Spark Application Manager 的运行环境中，在通过 ExecutorRunnable 设置到运行 Executor 的 ContainerLaunchContext 中。

##### 包分发

* Local 和 Standalone 模式是单机或整个环境部署多个节点，所以不需要分发。


* YARN 模式中，运行库和程序运行依赖的文件首先通过 HDFS 客户端 API 上传到作业 .sparkStaging 目录下，然后将对应的文件和 URL 映射关系通知 YARN ，YARN 的 Node Manager 在启动 Container 的时候会从指定的 URL 下载文件。Spark Application Manager 在创建 Executor 的 Container 的时候通过 setLocalResources 设置 Executor 的环境变量。

#### 术语表

| Term            | Meaning                                                      |
| --------------- | ------------------------------------------------------------ |
| Application     | 由 Driver 程序和 Executors 组成。                            |
| Application jar | 包含用户程序代码和依赖的 jar 包，Hadoop 和 Spark 的 libraries 应该排除在外，因为它们会在运行时被加入。 |
| Driver program  | 运行主函数的进程。                                           |
| Cluster manager | 在集群中获取资源的扩展服务 (e.g. standalone manager, Mesos, YARN)。 |
| Deploy mode     | Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster. |
| Worker node     | 集群中任何可以运行 application code 的节点。                 |
| Executor        | 在 Worker 节点上由程序启动的进程，负责执行 Task 和存储数据。 |
| Task            | 发送给 Executor 的一个 work 单元。                           |
| Job             | 有多个 Task 组成由 Spark action 触发的并行计算单元。         |
| Stage           | 每个 job 中，相同的 task 组成的集合。Stage 之间相互依赖。    |

### 4. Spark RDD

RDD (resilient distributed dataset) 是分布式数据集，是 Spark 中最基本的数据单元。

RDD 具有 5 个特点：

1. 一组分片的集合。分片是数据集的基本组成单位，每个分片都会被计算任务处理，用户可以指定分片数量，也可以使用默认分片数量（按 CPU core）。
2. 每个分片都有用来 computing 的函数。RDD 的计算以分片为单位，每个 RDD 都会实现 compute 函数。
3. RDD 之间有依赖关系。RDD 每次转换都会生成新的 RDD ，RDD 之间存在前后依赖关系。
4. key-value 类型的 RDD 具有 partitioner 。Spark 中实现了 2 种类型的分片函数，基于哈希的 HashPartitioner 和基于范围的 RangePartitioner 。key-value 类型的 RDD 才有 partitioner ，非 key-value 类型的 RDD 的 partitioner 为 none 。
5. RDD 存储了每个分片的优先位置。类似于 hadoop 的数据和计算尽可能在同一个 node 的概念。

#### 类型

Spark 中有 2 种类型的 RDD ：transformation 和 action 。

transformation 是延迟加载，action 会触发计算。

常用 transformation 有：

| transformation                                 | 说明                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| map(f)                                         | 返回一个新的RDD，新RDD的元素由f函数转换而来。                |
| filter(f)                                      | 返回一个新的RDD，新RDD的元素由f函数计算为true的元素组成。    |
| flatMap(f)                                     | 每个输入元素被转换成一个序列。                               |
| mapPartitions(f)                               | 和map类似，但f函数的类型必须是Iterator[T]=>Iterator[U] 。    |
| mapPartitionsWithIndex(f)                      | 和mapPartitions类似，但f函数的类型必须是(Int,Iterator[T])=>Iterator[U] 。 |
| sample(withReplacement, fraction, seed)        | 根据fraction指定的比例对数据进行采样，可以选择是否使用随机数进行替换，seed用来执行随机数生成器的种子。 |
| union(dataset)                                 | 对源RDD和参数RDD求并集返回新的RDD，元素不会去重。            |
| intersection(dataset)                          | 对源RDD和参数RDD求交集返回新的RDD。                          |
| distinct([numTasks])                           | 对RDD去重。                                                  |
| groupByKey([numTasks])                         | 按key聚合，返回(key, Iterator[V])的RDD。                     |
| reduceByKey(f, [numTasks])                     | 使用指定的redusce函数，对相同key元素进行聚合操作，reduce任务的个数可以由参数指定。 |
| aggregateByKey                                 |                                                              |
| sortByKey([ascending], [numTasks])             | 返回一个按照key进行排序的RDD，key必须实现Ordered接口。       |
| sortBy(f, [ascending], [numTasks])             |                                                              |
| join(dataset, [numTasks])                      | 返回一个(K, (V,W))的RDD。                                    |
| cogroup(dataset, [numTasks])                   | 返回一个(K, (Iterator[V],Iterator[W]))的RDD。                |
| cartesian(dataset)                             | 笛卡尔积                                                     |
| pipe(command, [numTasks])                      |                                                              |
| coalesce(numPartitions)                        |                                                              |
| repartition(numPartitions)                     |                                                              |
| repartitionAndSorWithinPartitions(partitioner) |                                                              |

```scala
scala> sc.parallelize(Array(6,5,4,9,8,7,1,2,3)).filter(_>5).collect()
res0: Array[Int] = Array(6, 9, 8, 7)
scala> sc.parallelize(Array("a b c", "x y z", "o p q")).flatMap(_.split(" ")).collect()
res1: Array[String] = Array(a, b, c, x, y, z, o, p, q)
scala> sc.parallelize(Array(6,5,4,9,8,7,1,2,3)).map(_*2).sortBy(x=>x, true)
res2: Array[Int] = Array(2, 4, 6, 8, 10, 12, 14, 16, 18) 
scala> val rdd1 = sc.parallelize(List(("Tom", 20),("John", 21),("Alice", 19)))
scala> val rdd2 = sc.parallelize(List(("Bob", 20),("John", 21),("Pony", 22)))
scala> rdd3.collect()
res5: Array[(String, Int)] = Array((Tom,20), (John,21), (Alice,19), (Bob,20), (John,21), (Pony,22))
scala> rdd3.groupByKey.collect()
res6: Array[(String, Iterable[Int])] = Array((Alice,CompactBuffer(19)), (Bob,CompactBuffer(20)), (Tom,CompactBuffer(20)), (Pony,CompactBuffer(22)), (John,CompactBuffer(21, 21)))
scala> rdd3.reduceByKey(_+_).collect()
res7: Array[(String, Int)] = Array((Alice,19), (Bob,20), (Tom,20), (Pony,22), (John,42))
```

常用的 Action 有：

| action                                   | 说明                                          |
| ---------------------------------------- | --------------------------------------------- |
| reduce(f)                                | 通过f函数聚合RDD中的所有元素。                |
| collect()                                | 以数组形式返回数据集的所有元素。              |
| count()                                  | 返回RDD的个数。                               |
| first()                                  | 返回RDD的第一个元素。                         |
| take(n)                                  | 返回数据集的前n个元素的数组。                 |
| takeSample(withReplacement, num, [seed]) | 返回从数据集中随机采样的n个元素组成的数组。   |
| takeOrdered(n, [ordering])               |                                               |
| saveAsTextFile(path)                     | 将数据集元素以textFile形式保存到文件系统。    |
| saveAsSequenceFile(path)                 | 将数据集元素以hadoop sequencefile的格式保存。 |
| saveAsObjectFile(path)                   |                                               |
| countByKey()                             | 返回(K, Int)的map。                           |
| foreach(f)                               | 对数据集的每一个元素执行f函数。               |



#### 缓存

cache() 或 persist() ，cache() 最终也是调用的 persist() 。默认是缓存在内存中，缓存会将 RDD 的结果缓存起来，缓存是延迟触发，在下一个 action 操作才会执行。

#### 容错机制

Spark RDD 的容错是通过 checkpoints 来做的。

有两种设置检查点的方式：本地目录和HDFS 目录。

```scala
scala> sc.setCheckpointDir("hdfs://hadoop1:8020/checkpoint/")
scala> val rdd1 = sc.textFile("hdfs://hadoop1:8020/user/wordcount.txt")
scala> rdd1.checkpoint
```

checkpoints 优点是可以容错，但是缺点是有磁盘 IO 操作，会影响效率。

#### 依赖关系

Spark 作业提交后，会将 RDD 划分成多个 stage ，划分的依据就是依赖关系。RDD 之间的依赖关系有两种：一种是窄依赖，一种是宽依赖。窄依赖是说父 RDD 的分区最多只会被子 RDD 的一个分区使用。宽依赖则是一个父 RDD 的分区会被子 RDD 的多个分区使用。

<img src="https://z3.ax1x.com/2021/06/07/2BKxnf.png" alt="855959-20161009115627506-271998705.png" border="0" />

按照 RDD 的执行顺序，遇到窄依赖会被加入到同一个 stage ，遇到宽依赖则新划分一个 stage 。每个 stage 中 task 的数量由该 stage 中最后一个 RDD 的分区数量决定。

随着数据量的增加，窄依赖的子 RDD 依赖的父 RDD 的 partition 数量数确定的，而宽依赖是数据量越大，子 RDD 依赖的父 RDD 的数量越多。

### 5. Spark 调度

#### Spark Application 之间的调度

每个 Spark 程序会获得一个独立的 JVM 来执行任务和存储数据。当多个 Spark 程序共享集群资源，不同的集群管理器会有不同的调度策略。

##### 静态资源分配

最简单的方式是静态划分所有可用的集群资源。这种方式，每个 Spark 程序被分配它可用的最大资源，并在整个声明周期都占有这些资源。这种方式在 Spark Standalone 、YARN 和 [coarse-grained Mesos](https://spark.apache.org/docs/1.6.3/running-on-mesos.html#mesos-run-modes) 模式中。

* Standalone mode 模式默认提交的任务是按照 FIFO 的顺序执行，执行期间占有所有可用的节点。通过修改 `spark.cores.max` 和 `spark.executor.memory` 参数可以修改程序可使用的节点数和内存。
* YARN 模式通过设置 `--num-executors` 选项控制 Executor 的数量，使用 `--executor-memory` 和 `--executor-cores` 选项控制 Executor 可用的资源。

> 不管使用何种方式，都不能在程序之间共享内存。

##### 动态资源分配

Spark 提供了一种按照负载动态分配资源的方式，当程序不在使用资源的时候将资源还给集群，这种特性在多个程序共享集群资源的时候是非常有用的。这个特性是默认关闭的，可以在所有 coarse-grained cluster managers 上设置启用。

Spark 程序可在等待被调度的任务被挂起的时候动态请求额外的 Executor 。Spark 以轮询的方式请求 Executor ，在任务被挂起到达一定时间时，请求才真正触发。每次触发请求的 Executor 数量以指数递增。这样做的原因是考虑到应用在开始阶段只需要较少的 Executor ，当确实需要大量的 Executor 时，指数递增能够快速提升处理能力。

相对来时 Spark 的释放策略要简单的多，当闲置超过一定时间 Executor 会被释放。在大多是情况下，任务被挂起说明资源不足，闲置说明资源过剩，所以 Executor 的申请和释放是互斥的。

静态分配资源时，相关的程序正常退出或出错，Executor 都会退出，两种情况下和 Executor 相关的状态不会再被使用，可以安全的移除。但是动态分配资源时，Executor 退出时程序还在运行，如果程序访问该 Executor 保存的状态，就会重新计算。比如，在 shuffle 期间，Spark 先将 map 写到本地磁盘，然后其他 Executor 可以访问计算结果。在 shuffle 完成之前，Executor 会被移除，完成后，shuffle 的结果将不可访问，如果其他的 Executor 要访问，就需要重新计算。因此，Spark 提供了额外的 shuffle 服务来保存 shuffle 文件，即使 Executor 退出，也可以通过该服务访问 shuffle 文件，因为 shuffle 服务是独立于程序和 Executor 的。

#### Application 内的调度

##### 调度流程

Spark 是按照 DAG 图来进行的任务调度的。调度可分为两部分： DAGScheduler 和 TaskScheduler 。DAGScheduler 负责 stage 级别的调度， TaskScheduler 负责 task 级别的调度。

① Spark 程序每一个 action 都会提交给 DAGScheduler 一个 job，DAGScheduler 首先需要逆向遍历 RDD 依赖链，划分出 stage 并确定 stage 之间的依赖关系。Spark 有两种类型的 stage ： ShuffleMapStage 和 ResultStage 。Wide Dependency 会产生 shuffle 操作，shuffle 操作的结果是生成 ShuffleRDD ，其依赖关系是 ShuffleDependency 。Spark 通过 ShuffleDependency 来划分 stage，stage 的边界是从 ShuffleRDD 的父 RDD 开始计算的。

> Narrow Dependency 指子 RDD 只依赖父 RDD 的一个或几个 partition 。
>
> Wide Dependency 指子 RDD 只依赖父 RDD 的所有的 partition 。

② 划分好 stage ，Spark 会生成 FinalStage 并提交。根据依赖关系，判断 FinalStage 的父 stage 结果是否可用，如果父 stage 的结果不可用，则尝试迭代提交父 stage 。如果所有的父 stage 结果都可用，则提交 FinalStage 。

③ stage 按 partition 数量拆分出 task 放入 TaskSet 提交给 TaskScheduler ，至此 DAGScheduler 调度结束。

④ TaskScheduler 和 TaskSetManager 根据资源情况将 task 调度到最佳的 Executor 上进行计算。

⑤ DAGScheduler 和 TaskScheduler 通过回调函数获取 Task 和 TaskSet 的状态，以及 Executor 的状态。当 Task 执行完成后，DAGScheduler 会获取 Task 执行结果。对于非 FinalStage 的 Task ，返回的是 MapStatus 对象，存储的是计算结果的位置信息，而对于 ResultTask 类型的 Task 返回的是结果本身，如果结果比较小，则直接放在 DirectTaskResult 中，如果结果很大，则将结果存储在 BlockManager 中，然后将 BlockId 返回给 DAGScheduler 。 

##### 调度策略

在 Spark 程序内部，多个线程提交的任务可以并行执行，是线程安全的。默认的 Spark 任务调度策略是 FIFO 。每个任务被划分到 stage 中，stage 开始时第一个任务获得优先权，然后是第二个，以此类推。如果一个任务没有使用全部资源，下一个任务可以立刻开始执行。如果前边的任务占用了全部的资源，后边的任务只能等待前边的任务完成后再开始执行。

Spark 可以通过配置 `spark.scheduler.mode`  为 FAIR 来让任务公平共享资源。在公平模式下，在任务长时间运行时新提交一个较小的任务也能够马上开始执行，不需要等待之前的任务执行完成再开始执行。和 Hadoop Fair scheduler 类似，公平调用支持将作业分组到调度池中，并且为每个调度池分配不同的选项，比如将重要的任务放在优先级高的池中或是将不同用户的任务放在各自的池中。调度池有 3 个配置选项：

* schedulingMode 可配置任务在池内的调度策略。
* weight 可配置池的获得的资源，即优先级。
* minShare 可配置池的最小资源。

默认情况下，池的属性为 `scheduling mode FIFO weight 1 minShare 0` 。

### 6. 序列化

序列化对任何分布式系统的性能来说都至关重要。序列化对象慢或占用大量的字节将大大降低计算的性能。通常在 Spark 程序中，我们需要首先将序列化调整到最优。Spark 提供了两个序列化类库，以便在易用性和效率之间达到平衡。

默认的，Spark 序列化对象使用 Java 的 ObjectOutputStream ，可以作用于任何实现了 java.io.Serializable 接口的类。虽然 Java 的序列化很灵活，但是性能较低同时占用字节数很大。

Spark 使用 Kryo 序列化来提高序列化的性能。它比 Java 序列化更快占用字节数更少。但是 Kryo 并不支持所有的Serializable 类型，需要手动注册。

要使用 Kryo 作为序列化库，需要配置 

```scala
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

这个配置不仅用于节点之间 shuffle 数据，还会影响 RDDs 序列化到磁盘。因为要手动注册，所以 Kryo 不是默认的序列化方案，但还是推荐使用 Kryo ，尤其是网络密集型程序。从 Spark 2.0.0 开始，简单类型，简单类型的数组及字符串的 shuffle 操作时使用 Kryo。

通过以下方式对自定义类进行注册，如果对象很大，需要加上 spark.kryoserializer.buffer 的配置。

```scala
val conf = new SparkConf().setMaster(...).setAppName(...).set("spark.kryoserializer.buffer", "64m")
conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
val sc = new SparkContext(conf)
```

如果不注册自定义类，Kryo 仍然会工作，不过需要将每一个对象的完全的类名保存起来，这会非常浪费资源。

### 7. 存储系统

Spark 任务会从外部存储系统中读取数据，所以将它放在尽可能近的地方是非常重要的。一般来说 Spark 程序和数据放在相同节点是效率最高的，如果无法保证相同节点，退一步讲，应将程序和数据放在同一网络环境下。

#### 本地磁盘

虽然 Spark 是基于内存的运算，但是在数据无法放置到内存或保存 stage 间的中间数据还是需要使用磁盘。推荐每个节点 4-8 块磁盘，单独挂载不需要配置 RAID ，使用 noatime 选项减少不必要的写操作。 磁盘信息通过设置 spark.local.dir 参数配置。如果运行在 HDFS 上，可以使用和 HDFS 相同的配置。

#### 内存

Spark 可以使用 8 到上百 GB 的内存，但是推荐分配 75% 的内存给 Spark，其他的留给操作系统和缓存使用。JVM 在 200 GB 以上的机器上表现并不理想，如果机器内存超过 200 GB ，可以在节点上运行多个 worker JVMs 。

#### 网络

Spark 在运行期间是网络绑定的，使用万兆或更大的带宽程序会执行更快。对 group-bys，reduce-bys，joins 等分布式减少的程序尤其适用。

#### CPU

Spark 可以在每个节点上扩展到十多个 CPU 内核，因为它是线程之间最小共享的。

### 8. Shuffle

Shuffle 是 Spark 中重新分布数据的机制，它会在机器之间复制数据，是一个复杂且开销很大的操作。

### 9. 持久化

将数据集持久化或缓存到内存里是 Spark 的重要能力之一。在持久化数据集时，在内存中参与计算的数据集被存储在每一个节点上，其他的 action 可以快速的重用这些数据集。缓存是迭代算法和快速交互的关键。

#### 数据块



#### 持久化数据集

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

#### 持久化 shuffle

Spark 会在执行 shuffle 操作时自动持久化一些中间数据。这样可以避免 shuffle 失败时重新计算整个输入。如果确定结果会被重用，建议对数据集进行持久化。

#### 共享变量

通常，函数传递到 Spark 集群节点执行，函数中使用所有变量的独立副本，这些变量被拷贝到每一个节点，集群节点上的变量变更不会影响驱动程序，因为读写共享的变量访问时低效的。但 Spark 提供了两种有限类型的共享变量：广播变量和累加器。

##### 广播变量

广播变量允许程序保持一份只读的变量缓存在机器上，而不是把变量拷贝到任务中。Spark 的 action 是分阶段执行的，每个阶段任务需要使用的数据被自动广播。广播数据以序列化的形式缓存并在任务执行前反序列化。但多个阶段的任务需要相同的数据时，广播变量更加有效。当广播变量创建后，会在方法中代替变量值，以保证不会被多次发送。广播变量在发送到节点后不能再被修改。

##### 累加器

### 10. Spark 调优

Spark 调优可以分几个方面来说，首先是程序和资源调优，需要遵守一些基本规范，是所有调优的基础。再就是针对作业运行过程中出现的问题如数据倾斜、shuffle 进行调优，减少或避免这些问题从而提高执行效率。

#### 程序调优

这部分主要是要遵守和使用一些基本的开发规范和技巧，根据业务场景合理运用。

##### 1. 避免创建重复的 RDD

通常在开发 Spark 作业时，会首先基于某个数据源创建一个初始的 RDD ，然后对这个 RDD 一步一步进行操作，直到得到最终结果。在这个过程中，应该避免创建多个 RDD 来表示同一份数据。

##### 2. 尽可能复用 RDD

这样可以尽可能地减少 RDD 的数量，从而尽可能减少算子执行的次数。比如说，有一个 RDD 的数据格式是 key-value 类型的，另一个是单 value 类型的，这两个 RDD 的 value 数据是完全一样的。那么此时我们可以只使用 key-value 类型的那 个 RDD，因为其中已经包含了另一个的数据。

##### 3. 对多次使用的 RDD 进行持久化

Spark 中对于一个 RDD 执行多次，默认会重新从源头处计算一遍，计算出这个 RDD 来，然后再对这个 RDD 执行操作。这时应该将 RDD 持久化，这样 Spark 会直接使用持久化的数据，而不会从头计算一边。

##### 4. 合理选择持久化策略

Spark 提供了多种持久化策略，默认是 MEMORY_ONLY 。MEMORY_ONLY 效率最高，如果内存充足，优先使用这个策略。如果 MEMORY_ONLY 不能满足，可尝试使用 MEMORY_ONLY_SER 。MEMORY_ONLY_SER 会将数据序列化后保存在内存中，比 MEMORY_ONLY 多了序列化开销，但是节省了内存。如果内存持久化不能满足，建议使用 MEMORY_AND_DISK_SER ，序列化后能够节省内存和磁盘占用。其他策略不建议使用。

##### 5. 尽可能避免使用 shuffle 算子

shuffle 算子可能导致数据倾斜。对于其中一个 RDD 的数据量比较小的场景，可以使用 broadcast 来避免 join 。将数据量小的 RDD 作为广播变量，在 map 中进行遍历比较，这样可以避免 shuffle 。

##### 6. 使用 Map-Side 预聚合的 Shuffle 操作

如果因为业务需要，必须使用 shuffle 操作，尽量使用 map-side 预聚合的算子。比如使用 reduceByKey 或 aggregateByKey 代替 groupByKey ，因为 reduceByKey 或 aggregateByKey 都会使用用户自定义函数对节点本地的相同 key 进行聚合，而 groupByKey 不会。

##### 7. 使用高性能的算子

reduceByKey 或 aggregateByKey 比 groupByKey 性能高，原因见第 6 条。

使用 mapPartitions 代替 map 。mapPartitions 会一次处理一个 partition 的数据，效率高，但是需要的内存更多。

用 foreachPartitions 代替 foreach 。

使用 filter 之后进行 coalesce 操作。

使用 repartitionAndSortWithinPartitions 代替 repartition 和 sort 。repartitionAndSortWithinPartitions 可以一边进行重分区的 shuffle 操作，一边进行排序，比先进行分区再进行排序性能要高。

##### 8. 广播大变量

在开发过程中，需要在算子中使用外部变量时，可以使用 broadcast 来提升性能。如果不使用 broadcast ，Spark 会将变量复制成多个副本，通过网络传输到 task 。如果变量较大，会增加网络开销，还会增加 executor 的内存占用。使用 broadcat 每个 executor 中只保存一份副本，所有 task 共享同一份副本。

##### 9. 使用 Kryo 优化序列化性能

在 Spark 中有 3 个地方用到了序列化：

1. 在算子中使用外部变量，变量会被序列化后再进行网络传输。
2. 将自定义类型作为 RDD 的泛型时，自定义类型的对象都会进行序列化。
3. 使用序列化的持久策略时。

以上几种都可以使用 Kryo 进行序列化。Spark 默认使用 Java 的序列化。Kryo 比 Java 序列化的性能要高，但是要求注册需要序列化的类型。

##### 10. 优化数据结构

Spark 是基于内存的，所以合理地管理内存会提高程序的性能。想要有效地管理内存，就要知道对象占用了多少内存，访问对象的开销以及 GC 的执行情况。

首先，要知道对象占了多少内存，我们需要清楚有哪些地方会消耗掉内存。默认情况下，Java 可以快速访问对象，但是需要消耗的空间是原生数据类型2-5倍。Java 对象为什么会比原生数据类型多消耗这么多空间呢？原因有以下几点：

* 每个 Java 对象都有一个“对象头”，对象头大约 16 个字节，包含类似指向类的指针信息。如果一个对象很小，对象头可能比对象包含的数据还要大。
* Java 的 String 在原始的 String 数据上会额外占用大约 40 字节 ，用来存储如长度之类的信息。由于 Java 使用 UTF-16 编码，所以每个字符会占用 2 个字节。一个 10 个字符的 String 大概会占用 60 个字节。
* HashMap 和 LinkedList 这样常见的数据结构使用了链表数据结构，每一个 entry 都是一个包装对象，这些对象不光包含对象头，还包含了指向下一个对象的引用，这个引用会额外占用 8 个字节。
* 常见的原始数据类型集合都会以包装类形式存储，比如 Integer。

因此，降低内存消耗的一种方法是不使用带有头和额外开销的数据结构。比如

* 使用数组和原始数据类型代替集合和包装类。
* 尽量避免嵌套的数据结构。
* 使用数值型和枚举型代替字符串作为 id 。
* 对于内存小于 32 G的机器，使用 JVM 的 ```XX:+UseCompressedOops``` 参数限制引用长度为 4 字节。

#### 资源调优

##### num-executors

Spark 启动多少个 executor 来执行作业。如果不设置则只会启动很少的 executor 来执行。一般设置 50-100 比较合适，设置太少，无法充分利用集群资源，设置太多，大部分无法获得需要的资源。

##### executor-memory

每个 executor 所占内存。num-executors * executor-memory 不要超过最大内存总量。

##### executor-core

于设置每个 Executor 进程的 CPU core 数量。每个 executor 的 CPU core 2-4 个，最好的应该就是一个 CPU core 对应两到三个 task 。

##### spark.default.parallelism

设置每个 stage 的默认 task 数量。如果不设置这个参数，Spark 默认会根据 hdfs 的 block 数量来设置 task 的数量，默认是一个 block 对应一个 task 。建议设置为 num-executors * executor-memory 的 2-3 倍。一般500 - 1000 是比较合理的范围。

##### spark.storage.memoryFraction

在 Spark 中内存被用来执行计算（shuffle、join、sort、aggregations）和存储数据（缓存和广播数据）。执行计算和存储数据使用的是同一块区域（M），如果执行计算的内存没有被使用，存储数据可以占用这些内存，反之，执行计算也可以使用没有被存储占用的内存。如果一个程序没有使用缓存，那么程序可以将所有内存用来执行计算。虽然执行计算可以释放存储以获取更多的内存，但是缓存在不超过阈值（R）的情况下，才会被释放。

关于内存的划分，Spark 提供了两个参数，但大多数场景下，用户是不需要调整 Spark 的配置的。

* spark.memory.fraction  默认为 0.6 ，即 M 的 60% 。剩余的 40% 空间用来存储。
* spark.memory.storageFraction 默认为 0.5 ，即 R 占 M 的 50% 。 


##### GC 

当您的程序中存储的 RDDs 有很大的“搅动”时，JVM 垃圾收集可能会成为一个问题。当 Java 需要清除旧对象为新对象腾出空间时，它需要跟踪所有的 Java 对象并找到未使用的对象。这里要记住的要点是，垃圾收集的成本与 Java 对象的数量成比例，所以使用较少对象的数据结构（例如，一个 Ints 数组而不是一个 LinkedList ）可以大大降低这个成本。一种更好的方法是将以序列化的形式存储对象，如上所述：现在每个RDD分区只会有一个对象（一个字节数组）。在尝试其他技术之前，如果GC是一个问题，首先要尝试的是使用序列化缓存。

GC 调优的第一步是收集有关垃圾收集发生频率和花费的时间的统计信息。这可以通过添加 ```-verbose:gc``` ```-XX:+PrintGCDetails``` ```-XX:+printgctimetstamp``` 选项来完成。

在进行调优之前，需要了解 Java 的 GC 模型。

* Java 堆空间分为两个区域，分别是 Young 和 Old。Young 用来保存寿命较短的对象，而 Old 则用来保存具有较长寿命的对象。
* Young 被进一步划分为三个区域：Eden, Survivor1, Survivor2 。
* 当 Eden 满时，会运行一次小的 GC ，Eden 和 Survivor1 中没有被回收的对象被复制到 Survivor2 。当对象寿命足够长或 Survivor2 满了，对象将被移到 Old 。最后，当 Old 接近饱和时，将调用 full GC 。

在Spark中，GC 调优的目标是确保长寿的 RDD 只存储在 Old 中，同时 Young 有足够的空间来存储寿命较短的 RDD 。这样有助于避免在执行期间调用 full GC 来回收临时对象。

以下方法可以参考：

* 通过收集 GC 统计信息检查是否有太多的 GC。如果在任务执行过程中多次出现 full GC，说明没有足够的内存可用来执行任务。
* 如果有大量的小 GC ，可以调整 Eden 的大小。可以将 Eden 的大小设置为任务所需要内存的最大值。如果将 Eden 的大小确定为 E ，那么 Young 的大小可以设置为 -Xmn=4/3 E。例如，从 HDFS 上读取数据，可以根据 HDFS 使用的数据块来估算需要的内存。如果有 3-4 个 HDFS 数据块，每个块的大小是 128 M，那么需要的 Eden 大小大概是 4 \* 3 \* 128  M 。（压缩块的大小是普通块大小的 2-3 倍）
* 在打印的 GC 统计数据中，如果 Old 快满了，那么通过降低 ```spark.memory.fraction``` 来减少用于缓存的内存的数量或者减小 ```-Xmn``` 的值。如果无效，可以尝试调大 JVM 的 NewRatio 参数。
* 使用 G1 垃圾回收器。设置参数 ```-XX:+UseG1GC``` 。使用 ```-XX: g1heap``` 增加 G1 的大小。

剩下的工作就是在我们修改 GC 的参数后，GC 统计信息是否有变化。

#### 数据倾斜

在进行 shuffler 时，需要将各节点相同的 key 拉到同一个节点的同一个 task 进行处理，如果一个 key 的数据量特别大，就会发生数据倾斜。一个 stage 的 task 要全部执行完成才能进行下一个 stage ，所以一个 task 执行缓慢会导致整个作业执行耗时变长。

解决数据倾斜的方案：

1. 数据预处理。由于数据分布不均，可以先对数据进行预处里，使数据均匀分布，再使用 Spark 进行处理，这样就可以解决 Spark 作业执行慢的问题。但是本身并没有解决数据分布不平均，只是将数据倾斜出现的时间提前到预处理阶段。
2. 过滤导致数据倾斜的 key。如果有少量的 key 导致数据倾斜，并且去除这些 key 对结果影响不大。
3. 提高 shuffle 的并行度。增加并行度，会减少每个 task 处理数据的数量，可以缓解数据倾斜的影响。
4. 局部聚合+全局聚合。比如对数据倾斜的 key 加上随机数前缀，先在本地进行聚合，然后再去掉前缀再进行全局聚合。仅适用于 join 类的 sheffle 操作。
5. 将 reduce join 转化成 map join 。如果 join 操作的其中一个集合较小，可以使用这种方法，通过遍历比较，再 map 端完成 join 操作。

## Part II. Spark SQL

### 简介

Spark SQL 是 Spark 的一个分布式 SQL 查询引擎模块，提供了一个 编程抽象 DataFrame 。和 Hive 类似，Spark SQL 将 SQL 转化为 RDD 并提交到集群上运行。

DataFrame 是一个数据集，概念上类似数据库表。

DataSet 是 1.6 添加的新接口，是 DataFrame 之上更高一级的抽象。

### DataFrame 操作

#### 创建 DataFrames

## Part III. Spark Streaming

