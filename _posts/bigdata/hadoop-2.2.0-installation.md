---
title: Hadoop 2.10.1 分布式集群搭建
date: 2015-10-13 15:45:30
tags:
 - Hadoop
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

### 1. 安装JDK

```
tar -zxf jdk-8u281-linux-x64.tar.gz
mv jdk-8u281-linux-x64 java
```

配置环境变量

```
export JAVA_HOME=/opt/java/
export PATH=.:$PATH:$JAVA_HOME\bin
```



### 2. 安装Hadoop

#### 2.1 解压

```
tar -zxf hadoop-2.10.1.tar.gz
```
#### 2.2 设置环境变量

```
export HADOOP_HOME=/opt/hadoop-2.10.1/
export PATH=.:$PATH:$JAVA_HOME\bin:$HADOOP_HOME/sbin
```
#### 2.3 修改配置文件
##### 2.3.1 修改core-site.xml

./etc/hadoop/core-site.xml
```
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://hadoop1:8020</value>`  
    </property>
    <property>
    	<name>ha.zookeeper.quorum</name>
    	<value>hadoop1:2181,hadoop2:2181,hadoop3:2181</value>
   	</property>
</configuration>
```
##### 2.3.2 修改 hdfs-site.xml

./etc/hadoop/hdfs-site.xml
```
<configuration>
   <property>
     <name>dfs.datanode.data.dir</name>
     <value>file:///opt/hadoop-2.10.1/data/datanode</value>
   </property>
   <property>
     <name>dfs.namenode.name.dir</name>
     <value>file:///opt/hadoop-2.10.1/data/namenode</value>
   </property>
   <property>
     <name>dfs.namenode.http-address</name>
     <value>hadoop1:50070</value>
   </property>
   <property>
     <name>dfs.namenode.secondary.http-address</name>
     <value>hadoop2:50090</value>
   </property>
   <property>
     <name>dfs.replication</name>
     <value>1</value>
   </property>
</configuration>
```
##### 2.3.3 修改 mapred-site.xml

./etc/hadoop/mapred-site.xml
```
<configuration>
      <property>
      	<name>mapreduce.framework.name</name>
      	<value>yarn</value>
      </property>
</configuration>
```
##### 2.3.4 修改 yarn-site.xml

./etc/hadoop/yarn-site.xml
```
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hadoop1:8025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hadoop1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hadoop1:8050</value>
  </property>
</configuration>
```
#### 2.4 设置互信
生成 ssh key

```
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa  
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys  
chmod 0600 ~/.ssh/authorized_keys
```
测试 ssh

>ssh localhost
#### 2.5 格式化 namenode
```
./bin/hdfs namenode -format
```
#### 2.6 启动 namenode 和 datanode
```
./sbin/start-all.sh
```
#### 2.7 验证
```shell
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar wordcount  /wordcount/README.txt /output
./bin/hdfs dfs -cat /output/p*
```
#### 2.8 关闭
```
./sbin/stop-all.sh 
```
