---
title: Installing and Configuring Apache Kylin
tags:
  - Apache Kylin
category:
  - Big Data
author: bsyonline
lede: 没有摘要
date: 2018-04-10 21:02:11
thumbnail:
---

#### **kylin 简介**
Apache Kylin 是一个开源的分布式分析引擎，提供 Hadoop 之上的 SQL 查询接口及多维分析（OLAP）能力以支持超大规模数据，最初由 eBay Inc. 开发并贡献至开源社区。
#### **kylin 安装**

kylin 是基于 hadoop 生态的，所以依赖于 hadoop 、hbase、hive 等组件。这里我们使用的是 hadoop-2.6.2、hbase-1.2.6 、hive-1.2.2 和 kylin-2.3.1 。
##### **安装 hadoop**
1. 下载
```
wget https://archive.apache.org/dist/hadoop/core/hadoop-2.6.2/hadoop-2.6.2.tar.gz
tar -zxf hadoop-2.6.2.tar.gz
cd hadoop-2.6.2
```
2. 修改环境变量
```
sudo echo "HADOOP_HOME=/hadoop-2.6.2" >> /etc/profile.d/hadoop.sh
sudo echo 'PATH=$HADOOP_HOME/bin:$PATH' >> /etc/profile.d/hadoop.sh
sudo echo "export HADOOP_HOME PATH" >> /etc/profile.d/hadoop.sh
. /etc/profile
```
3. 修改配置
hadoop-2.6.2/etc/hadoop/core-site.xml
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
hadoop-2.6.2/etc/hadoop/hdfs-site.xml
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
hadoop-2.6.2/etc/hadoop/mapred-site.xml
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
hadoop-2.6.2/etc/hadoop/yarn-site.xml
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
4. 格式化 namenode
```
cd hadoop-2.6.2/bin
./hdfs namenode -format
```
5. 启动
```
cd hadoop-2.6.2/sbin
./start-yarn.sh
./start-dfs.sh
./mr-jobhistory-daemon.sh start historyserver
```

##### **安装 hbase**
1. 下载
```
wget https://archive.apache.org/dist/hbase/1.2.6/hbase-1.2.6-bin.tar.gz
tar -zxf hbase-1.2.6-bin.tar.gz
```
2. 修改环境变量
```
sudo echo "HBASE_HOME=/hbase-1.2.6" >> /etc/profile.d/hbase.sh
sudo echo 'PATH=$HBASE_HOME/bin:$PATH' >> /etc/profile.d/hbase.sh
sudo echo "export HBASE_HOME PATH" >> /etc/profile.d/hbase.sh
. /etc/profile
```
3. 修改配置
hbase-1.2.6/conf/hbase-env.sh
```
export JAVA_HOME=/u01/java/latest
```
hbase-1.2.6/conf/hbase-site.xml
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>localhost</value>
  </property>
</configuration>
```
4. 启动
```
cd hbase-1.2.6/bin
./start-hbase.sh
./local-regionservers.sh start 1
./local-master-backup.sh start 1
```

##### **安装 hive**
1. 下载
```
wget https://archive.apache.org/dist/hive/hive-1.2.2/apache-hive-1.2.2-bin.tar.gz
tar -zxf apache-hive-1.2.2-bin.tar.gz
```
2. 修改环境变量
```
sudo echo "HIVE_HOME=/apache-hive-1.2.2-bin">/etc/profile.d/hive.sh
sudo echo 'PATH=$PATH:$HIVE_HOME/bin'>>/etc/profile.d/hive.sh
sudo echo "export HIVE_HOME PATH">>/etc/profile.d/hive.sh
. /etc/profile
```
3. 修改配置
```
cd apache-hive-1.2.2-bin/conf
cp hive-default.xml.template hive-default.xml
cp hive-env.sh.template hive-env.sh
cp hive-log4j.properties.template hive-log4j.propertiesn
cp hive-exec-log4j.properties.template hive-exec-log4j.properties
```
4. 配置 MySQL
hive 默认使用 derby 存储元数据，但是 derby 存在多用户访问异常的问题，所以使用 mysql 存储元数据。
安装 mysql 
```
sudo apt-get install mysql-server
```
修改权限
```
mysql -uroot -p123456
mysql> CREATE USER 'hive' IDENTIFIED BY 'hive';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' WITH GRANT OPTION;
mysql> flush privileges;
```
创建 hive 元数据库
```
mysql -uhive -phive
mysql> create database hive;
```
修改 apache-hive-1.2.2-bin/conf/hive-site.xml 。
```
<configuration>
    <!-- mysql 配置 -->
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive?characterEncoding=UTF-8</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>Username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
    <description>password to use against metastore database</description>
  </property>
  <configuration>
```
拷贝 mysql 驱动到 apache-hive-1.2.2-bin/lib 。
4. 启动
```
cd apache-hive-1.2.2-bin/bin
./hive
```

##### **安装 kylin**
1. 下载
```
wget https://archive.apache.org/dist/kylin/apache-kylin-2.3.1/apache-kylin-2.3.1-hbase1x-bin.tar.gz
tar -zxf apache-kylin-2.3.1-hbase1x-bin.tar.gz
```
2. 修改环境变量
```
sudo echo "KYLIN_HOME=/kylin-sample/apache-kylin-2.3.1-bin" >> /etc/profile.d/kylin.sh
sudo echo "export KYLIN_HOME"
. /etc/profile
```
3. 检查环境
```
cd apache-kylin-2.3.1-hbase1x-bin/bin
./check-env.sh
./find-hadoop-conf-dir.sh
./find-hbase-dependency.sh
./find-hive-dependency.sh
```
如果以上都安装成功，此处检查应该没有问题。
4. 启动
```
cd apache-kylin-2.3.1-hbase1x-bin/bin
./kylin start
```
启动后通过 [localhost:7070/kylin](localhost:7070/kylin) 访问，用户名密码： ADMIN/KYLIN。
