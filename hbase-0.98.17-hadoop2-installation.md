---
title: Hbase-0.98.17-hadoop2 安装
date: 2015-11-01 15:45:50
tags:
 - Hbase
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


#### **1. 下载解压**
```
tar -zxf hbase-0.98.17-hadoop2-bin.tar.gz
```
#### **2. 修改配置**
hbase-env.sh
```
export JAVA_HOME=/usr/java/jdk1.6.0_45
```
hbase-site.xml
```
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://node1:8020/hbase</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>hdfs://node1:60000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1</value>
    </property>
</configuration>
```
检查jar包

检查/usr/hbase/hbase-0.98.17-hadoop2/lib下hadoopjar包的版本是否和安装的hadoop一致。如不一致需要将$HADOOP_HOME/share/hadoop下的hadoop*.jar拷贝到$HBASE_HOME/lib下。
```
find $HADOOP_HOME/share/hadoop -name "hadoop*.jar" | xargs -i cp {} $HABASE_HOME/lib
```

#### **4. 格式化namenode并删除原有数据信息（可选）**

>如之前装过其他版本hadoop或hbase，建议执行该步操作


#### **5. 启动**
```
./start-hbase.sh
./local-regionservers.sh start 1
./local-master-backup.sh start 1
./hbase shell
```