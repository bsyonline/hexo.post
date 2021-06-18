---
title: Spark 序列化
date: 2016-05-11 11:50:28
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

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



