---
title: Definitive Guide for Spark
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-30 18:31:35
thumbnail:
---





### Spark 简介



### Spark 安装



### Spark 运行机制



### Spark RDD

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



### Spark SQL



### Spark Streaming

