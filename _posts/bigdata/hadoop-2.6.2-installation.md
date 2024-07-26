---
title: Hadoop 2.6.2 安装
date: 2015-10-20 15:45:55
tags:
 - Hadoop
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



#### **1. 下载**
[]()

#### **2. 解压缩**
```
tar -zxf hadoop-2.6.2.tar.gz
```
#### **3. 修改配置文件**
./etc/hadoop/core-site.xml
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
./etc/hadoop/hdfs-site.xml
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
./etc/hadoop/mapred-site.xml
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
./etc/hadoop/yarn-site.xml
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
#### **4. 设置互信**
```
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa  
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys  
chmod 600 ~/.ssh/authorized_keys
```
#### **5. 格式化 namenode**
```
./bin/hdfs namenode -format
```
#### **6. 启动 yarn**
```
./sbin/start-yarn.sh
```
#### **7. 启动 hdfs**
```
./sbin/start-dfs.sh
```
#### **8. 执行 wordcount**
```
./bin/hdfs dfs -mkdir /user  
./bin/hdfs dfs -mkdir /user/rolex  
./bin/hdfs dfs -mkdir /user/rolex/input  
./bin/hdfs dfs -put ./NOTICE.txt /user/rolex/input/  

./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.2.jar wordcount /user/rolex/input/NOTICE.txt /user/rolex/output/

./bin/hdfs dfs -cat /user/rolex/output/part-r-00000
```
