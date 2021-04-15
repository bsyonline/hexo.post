---
title: Building HA Hadoop Cluster
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-04-10 09:08:12
thumbnail:
---



准备3台机器。

### 1. 安装 JDK

解压，配置环境变量。

```
export JAVA_HOME=/opt/java/
export PATH=.:$PATH:$JAVA_HOME/bin
```



### 2. 安装 ZooKeeper

1）解压，配置环境变量。

```
export JAVA_HOME=/opt/java/
export ZOOKEEPER_HOME=/opt/zookeeper
export PATH=.:$PATH:$JAVA_HOME/bin:ZOOKEEPER_HOME/bin
```

2）配置 zoo.cfg

最后加入

```
dataDir=/opt/zookeeper/data
server.1=hadoop1:2888:3888
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888
```

2888 是通信端口，3888 是选举端口。

3）配置 myid

创建文件 /opt/zookeeper/data/myid ，每个节点设置一个唯一 id （1-255），节点之间不重复即可。

4）启动

3 个节点都要启动。启动后可用 zkServer.sh status 查看状态。

### 3. 安装 Hadoop Cluster

#### 1）解压并配置环境变量

```
export JAVA_HOME=/opt/java/
export HADOOP_HOME=/opt/hadoop-2.10.1
export PATH=.:$PATH:$JAVA_HOME/bin:$HADOOP_HOME/sbin
```

#### 2）创建 ha 数据存储路径

#### 3）配置 hadoop-env.sh

```
export JAVA_HOME=/opt/java/
```

#### 4）配置 core-site.xml 

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-cluster/</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>hadoop1:2181,hadoop2:2181,hadoop3:2181</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/ha/hadoop/data/tmp</value>
    </property>
</configuration>
```

#### 5）配置 hdfs-site.xml 

```xml
<configuration>
    <!-- 完全分布式集群名称 -->
    <property>
        <name>dfs.nameservices</name>
        <value>hadoop-cluster</value>
    </property>
    <!-- 集群中 NameNode 节点都有哪些 -->
    <property>
        <name>dfs.ha.namenodes.hadoop-cluster</name>
        <value>nn1,nn2</value>
    </property>
    <!-- nn1 的 RPC 通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.hadoop-cluster.nn1</name>
        <value>hadoop1:9000</value>
    </property>
    <!-- nn2 的 RPC 通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.hadoop-cluster.nn2</name>
        <value>hadoop2:9000</value>
    </property>
    <!-- nn1 的 http 通信地址 -->
    <property>
        <name>dfs.namenode.http-address.hadoop-cluster.nn1</name>
        <value>hadoop1:50070</value>
    </property>
    <!-- nn2 的 http 通信地址 -->
    <property>
        <name>dfs.namenode.http-address.hadoop-cluster.nn2</name>
        <value>hadoop2:50070</value>
    </property>
    <!-- NameNode 元数据在 JournalNode 上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://hadoop1:8485;hadoop2:8485;hadoop3:8485/hadoop-cluster</value>
    </property>
    <!-- 声明 journalnode 服务器存储目录-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/opt/ha/hadoop/data/jn</value>
    </property>
    <!-- 开启 NameNode 失败自动切换 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!-- 配置失败自动切换实现方式-->
    <property>
        <name>dfs.client.failover.proxy.provider.hadoop-cluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <!-- 配置隔离机制，同一时刻只能有一台服务器对外响应 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>
    <!-- 使用隔离机制时需要 ssh 免密登录-->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_dsa</value>
    </property>
    <!-- 配置 sshfence 隔离机制超时时间 -->
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

#### 6）配置 mapred-site.xml 

```xml
<configuration>
    <!-- 指定 mr 框架为 yarn 方式 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- 设置 mapreduce 的历史服务器地址和端口号 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop3:10020</value>
    </property>
    <!-- mapreduce 历史服务器的 web 访问地址 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop3:19888</value>
    </property>
</configuration>
```

#### 7）配置 yarn-site.xml 

```xml
<configuration>
    <!-- 开启 RM 高可用 -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <!-- 指定 RM 的 cluster id -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn1</value>
    </property>
    <!-- 指定 RM 的名字 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <!-- 分别指定 RM 的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop1</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>hadoop1:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>hadoop2:8088</value>
    </property>
    <!--指定 zookeeper 集群的地址-->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop1:2181,hadoop2:2181,hadoop3:2181</value>
    </property>
    <!-- 要运行 MapReduce 程序必须配置的附属服务 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 开启 YARN 集群的日志聚合功能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- YARN 集群的聚合日志最长保留时长 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>86400</value>
    </property>
    <!--启用自动恢复-->
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
    <!--指定 resourcemanager 状态信息存储在 zookeeper 集群-->
    <property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>
    <property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop3:19888/jobhistory/logs</value>
    </property>
</configuration>
```

#### 8）启动 journalnode

```
/opt/hadoop-2.10.1/bin/hadoop-daemon.sh start journalnode
```

#### 9）格式化 NameNode 

```
/opt/hadoop-2.10.1/bin/hdfs namenode -format
```

将 hadoop1 的 NameNode 数据拷贝到 hadoop2

```
scp -r /opt/ha/hadoop/data hadoop2:/opt/ha/hadoop
```

#### 10） 初始化 ZooKeeper 的元数据

```
/opt/hadoop-2.10.1/bin/hdfs zkfc -formatZK
```

#### 11）启动 hdfs 集群

```
/opt/hadoop-2.10.1/sbin/start-dfs.sh 
```

#### 12）启动 yarn 集群

```
/opt/hadoop-2.10.1/sbin/start-yarn.sh
```

#### 13）启动 jobHistory

```
/opt/hadoop-2.10.1/bin/mr-jobhistory-daemon.sh start historyserver
```



#### 14）查看 hdfs 的 web 页面

NameNode1: [http://hadoop1:50070/dfshealth.html#tab-overview](http://hadoop1:50070/dfshealth.html#tab-overview )

NameNode2: [http://hadoop2:50070/dfshealth.html#tab-overview](http://hadoop2:50070/dfshealth.html#tab-overview) 

#### 15）查看 yarn 的 web 页面

http://hadoop1:8088/cluster



