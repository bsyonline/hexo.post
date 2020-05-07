---
title: Hadoop 2.2.0 安装
date: 2015-10-13 15:45:30
tags:
 - Hadoop
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

### 1. 下载

[https://archive.apache.org/dist/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz](https://archive.apache.org/dist/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz "hadoop-2.2.0")

### 2. 解压
```
tar -zxf hadoop-2.2.0.tar.gz
```
### 3. 配置环境变量
```
HADOOP_HOME=/usr/hadoop/hadoop-2.2.0  
配置文件路径 $HADOOP_HOME/etc/hadoop
```
### 4. 修改配置文件
#### 4.1 修改core-site.xml

./etc/hadoop/core-site.xml
```
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node1:8020</value>`  
      </property>
  	<property>
      	<name>hadoop.tmp.dir</name>
          <value>/home/tmp/hadoop2.0</value>
      </property>
	<property>
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>
</configuration>
```
#### 4.2 修改 hdfs-site.xml

./etc/hadoop/hdfs-site.xml
```
<configuration>
      <property>
      	<name>dfs.replication</name>
              <value>1</value>
      </property>
      <property>
      	<name>dfs.namenode.name.dir</name>
          <value>/usr/hadoop/hadoop-2.2.0/name</value>
      </property>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>/usr/hadoop/hadoop-2.2.0/data</value>
      </property>
      <property>
          <name>dfs.permissions</name>
          <value>false</value>
      </property>
</configuration>
```
#### 4.3 修改 mapred-site.xml

./etc/hadoop/mapred-site.xml
```
<configuration>
      <property>
      	<name>mapreduce.framework.name</name>
      	<value>yarn</value>
      </property>
</configuration>
```
#### 4.4 修改 yarn-site.xml

./etc/hadoop/yarn-site.xml
```
<configuration>
      <property>
      	<name>yarn.resourcemanager.address</name>
		<value>node1:8032</value>
      </property>
      <property>
		<name>yarn.resourcemanager.scheduler.address</name>
		<value>node1:8030</value>
      </property>
      <property>
		<name>yarn.resourcemanager.resource-tracker.address</name>
		<value>node1:8031</value>
      </property>
      <property>
		<name>yarn.resourcemanager.admin.address</name>
		<value>node1:8033</value>
      </property>
      <property>
		<name>yarn.resourcemanager.webapp.address</name>
		<value>node1:8088</value>
      </property>
      <property>
		<name>yarn.resourcemanager.scheduler.class</name>
		<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
      </property>
      <property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
      </property>
      <property>
		<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
</configuration>
```
### 5. 设置互信
#### 5.1 生成 ssh key
```
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa  
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys  
chmod 0600 ~/.ssh/authorized_keys
```
#### 5.2 测试 ssh
>ssh localhost
### 6. 格式化 namenode
```
./bin/hdfs namenode -format
```
### 7. 启动 namenode 和 datanode
```
./sbin/hadoop-daemon.sh start namenode  
./sbin/hadoop-daemon.sh start datanode  
./sbin/yarn-daemon.sh start resourcemanager  
./sbin/yarn-daemon.sh start nodemanager  
./sbin/yarn-daemon.sh start proxyserver  
./sbin/mr-jobhistory-daemon.sh start historyserver
```
### 8. 验证
```
./bin/hdfs dfs -mkdir /user/rolex/input/  
./bin/hdfs dfs -put ../README.txt /user/rolex/input/  
./bin/hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar grep /user/rolex/input/README.txt /user/rolex/output/ 'code'  
./bin/hdfs dfs -cat /user/rolex/output/part-r-00000
```
### 9. 关闭
```
./sbin/hadoop-daemon.sh stop namenode  
./sbin/hadoop-daemon.sh stop datanode  
./sbin/yarn-daemon.sh stop resourcemanager  
./sbin/yarn-daemon.sh stop nodemanager  
./sbin/yarn-daemon.sh stop proxyserver  
./sbin/mr-jobhistory-daemon.sh stop historyserver  
```
