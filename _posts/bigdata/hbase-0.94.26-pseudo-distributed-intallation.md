---
title: Hbase 0.94.26 伪分布模式安装
date: 2015-10-27 15:49:41
tags:
 - Hbase
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


#### **1. 解压缩**
```
tar -zxvf hbase-0.94.26.tar.gz
```
#### **2. 修改配置文件**
hbase-env.sh
```
# The java implementation to use.  Java 1.6 required.
export JAVA_HOME=/usr/java/jdk1.6.0_45/
# Extra Java CLASSPATH elements.  Optional.
export HBASE_CLASSPATH=/user/hadoop/hadoop-1.2.1/conf
# Tell HBase whether it should manage it's own instance of Zookeeper or not.
export HBASE_MANAGES_ZK=true
```
hbase-site.xml
```
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>localhost</value>
    </property>
</configuration>
```
#### **3. 运行**
````
/usr/hadoop/hadoop-1.2.1/bin
[rolex@node2 bin]$ ./start-all.sh （先运行hdfs）
/usr/hbase/hbase-0.94.27/bin
[rolex@node2 bin]$ ./start-hbase.sh （启动hbase）
[rolex@node2 bin]$ ./local-master-backup.sh start 1  （启动master备份）
[rolex@node2 bin]$ ./local-regionservers.sh start 1  （启动regionserver）
````