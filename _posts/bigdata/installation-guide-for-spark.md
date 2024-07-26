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

spark。

### 1. 下载解压

```
tar -zxf spark-2.4.8-bin-hadoop2.7.tgz
```

### 2. 修改配置文件

```
cd spark-2.4.8-bin-hadoop2.7/conf
cp spark-env.sh.template spark-env.sh
```

添加如下配置

```
export SPARK_MASTER_HOST=hadoop1
export SPARK_MASTER_PORT=7077
```

spark 有默认配置，如果不显式配置，也是可以运行的。

```
cp slaves.template slaves
```

添加如下配置

```
hadoop2
hadoop3
```

### 3. 复制到其他节点

```
scp -r spark-2.4.8-bin-hadoop2.7 hadoop2:/opt/
scp -r spark-2.4.8-bin-hadoop2.7 hadoop3:/opt/
```

### 4. 启动
```
./sbin/start-all.sh
```

启动后可访问 [http://hadoop1:8081](http://hadoop1:8081) 查看信息。


### 5. 启动 shell

```
./bin/spark-shell --master spark://hadoop1:7077
```
### 6. 测试

执行 wordcount 程序，需要先启动 hadoop ，
```shell
sc.textFile("hdfs://hadoop1:8020//wordcount/wordcount.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

执行结果如下：
```
res0: Array[(String, Int)] = Array((mapreduce,1), (hello,3), (java,1), (hadoop,1))
```
