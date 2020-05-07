---
title: Spark standalone 安装
date: 2016-05-04 16:00:22
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "spark standalone 即伪分布式部署,即在单节点部署 spark 。"
---

spark standalone 即伪分布式部署,即在单节点部署 spark ,部署方式比较简单。

### 1. 下载 spark 安装包

[http://www.apache.org/dyn/closer.lua/spark/spark-1.6.1/spark-1.6.1-bin-hadoop2.6.tgz](http://www.apache.org/dyn/closer.lua/spark/spark-1.6.1/spark-1.6.1-bin-hadoop2.6.tgz)

### 2. 解压
```
tar -zxf spark-1.6.1-bin-hadoop2.6.tgz
```

### 3. 修改配置文件

在 $SPARK_HOME/conf/spark-env.sh 后添加如下配置。

```
export SPARK_MASTER_IP=localhost
export SPARK_MASTER_PORT=7077
export SPARK_WORKER_CORES=1
export SPARK_WORKER_INSTANCES=1
export SPARK_WORKER_MEMORY=512M
```

spark 有默认配置，如果不显式配置，也是可以运行的。

### 4. 启动
```
./sbin/start-all.sh
```

启动后可访问[http://localhost:8080](http://localhost:8080)查看信息


### 5. 启动 shell

```
./bin/spark-shell --master spark://localhost:7077
```
### 6. 测试

执行 wordcount 程序，需要先启动 hadoop ，
```
val rdd=sc.textFile("hdfs://localhost:9000/user/rolex/input/NOTICE.txt")
rdd.cache()
val wordcount=rdd.flatMap(_.split(" ")).map(x=>(x,1)).reduceByKey(_+_)
wordcount.take(10)
val wordsort=wordcount.map(x=>(x._2,x._1)).sortByKey(false).map(x=>(x._2,x._1))
wordsort.take(10)
```

执行详细结果如下：
```
scala> val rdd=sc.textFile("hdfs://localhost:9000/user/rolex/input/NOTICE.txt")
rdd: org.apache.spark.rdd.RDD[String] = hdfs://localhost:9000/user/rolex/input/NOTICE.txt MapPartitionsRDD[5] at textFile at <console>:27

scala> rdd.cache()
res1: rdd.type = hdfs://localhost:9000/user/rolex/input/NOTICE.txt MapPartitionsRDD[5] at textFile at <console>:27

scala> val wordcount=rdd.flatMap(_.split(" ")).map(x=>(x,1)).reduceByKey(_+_)
wordcount: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[8] at reduceByKey at <console>:29

scala> wordcount.take(10)
res2: Array[(String, Int)] = Array((Apache,1), (developed,1), (This,1), (includes,1), (Software,1), (The,1), ((http://www.apache.org/).,1), (by,1), (software,1), (product,1))

scala> val wordsort=wordcount.map(x=>(x._2,x._1)).sortByKey(false).map(x=>(x._2,x._1))
wordsort: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[13] at map at <console>:31

scala> wordsort.take(10)
res3: Array[(String, Int)] = Array((Apache,1), (developed,1), (This,1), (includes,1), (Software,1), (The,1), ((http://www.apache.org/).,1), (by,1), (software,1), (product,1))
```
