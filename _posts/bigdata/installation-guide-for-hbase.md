---
title: Installation Guide for Hbase
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-04-18 23:55:38
thumbnail:
---



### Hbase 安装

#### 1. 解压

```
tar -zxf hbase-2.3.5-bin.tar.gz
```



#### 2. 配置环境变量

```
export HBASE_HOME=/opt/hbase
export PATH=.:$PATH:$HBASE_HOME/bin
```

#### 3. 修改配置文件

（1）vim conf/hbase-env.sh

```
export JAVA_HOME=/opt/java/
export HBASE_MANAGES_ZK=false
```

（2）vim conf/hbase-site.xml

```
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop-cluster/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop1,hadoop2,hadoop3</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/zookeeper</value>
  </property>
</configuration>
```

（3）拷贝 hadoop 的 core-site.xml 和 hdfs-site.xml 到 hbase 的 conf 目录。

（4）vim conf/regionservers

```
hadoop2
hadoop3
```

#### 4. 复制 Hbase 到其他节点

```
scp -r /opt/hbase-2.3.5 hadoop2:/opt/
scp -r /opt/hbase-2.3.5 hadoop3:/opt/
```

#### 5. 启动

```
start-hbase.sh
```

#### 6. 启动 shell

```
hbase shell
```

