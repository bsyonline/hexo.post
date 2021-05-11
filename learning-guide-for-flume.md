---
title: User Guide for Flume
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-02 10:16:30
thumbnail:
---



### Flume 简介

Flume 时一个分布式海量日志聚合系统。

<a href="https://imgtu.com/i/gZccy8"><img src="https://z3.ax1x.com/2021/05/02/gZccy8.png" alt="gZccy8.png" border="0" /></a>

Flume agent 由 Source 、Sink 和 Channel 。Flume 的数据流由 Event 构成，Event 是其基本单位。Event 由 Agent 外部的 Source 生成，并推入到 Channel 中，由 Sink 处理 Event 。

### Flume 部署方案

#### 单 Agent

<a href="https://imgtu.com/i/gZccy8"><img src="https://z3.ax1x.com/2021/05/02/gZccy8.png" alt="gZccy8.png" border="0" /></a>

#### 多 Agent 串联

<a href="https://imgtu.com/i/gZgq3t"><img src="https://z3.ax1x.com/2021/05/02/gZgq3t.png" alt="gZgq3t.png" border="0" /></a>

#### 多 Agent 合并



#### 多路复用

<a href="https://imgtu.com/i/gZgOjf"><img src="https://z3.ax1x.com/2021/05/02/gZgOjf.png" alt="gZgOjf.png" border="0" /></a>

### Flume 安装

#### 解压

```
tar -zxf apache-flume-1.9.0-bin.tar.gz
mv apache-flume-1.9.0-bin flume
```

#### 配置

```
cd flume/conf
mv flume-env.sh.template flume-env.sh
```

设置 JAVA_HOME 。

### Flume 使用示例

#### 1. netcat 测试

创建 conf 文件

```
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动 flume

```
bin/flume-ng agent --conf conf --conf-file job/single-node.conf --name a1 -Dflume.root.logger=INFO,console
```

发送 

```
telnet localhost 44444
hello
world
```

#### 2. 监控目录中的文件

创建 dir-hdfs.conf

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /opt/flume/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop1/flume/upload/%Y%m%d/%H
a3.sinks.k3.hdfs.filePrefix = upload-
a3.sinks.k3.hdfs.round = true
a3.sinks.k3.hdfs.roundValue = 1
a3.sinks.k3.hdfs.roundUnit = hour
a3.sinks.k3.hdfs.useLocalTimeStamp = true
a3.sinks.k3.hdfs.batchSize = 100
a3.sinks.k3.hdfs.fileType = DataStream
a3.sinks.k3.hdfs.rollInterval = 600
a3.sinks.k3.hdfs.rollSize = 134217700
a3.sinks.k3.hdfs.rollCount = 0
a3.sinks.k3.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory
a3.channels.c3.type = memory 
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

启动

```
bin/flume-ng agent --conf conf/ --name a3 --conf-file job/hdfs-dir.conf
```

拷贝文件到指定目录

```
cp ./README.md upload/
```

到 hdfs 上查看文件生成。

#### 3. 监控文件

```
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/flume/mydate.txt
a2.sources.r2.shell = /bin/bash -c 

# Describe the sink
a2.sinks.k2.type = hdfs 
a2.sinks.k2.hdfs.path = hdfs://hadoop1/flume/%Y%m%d/%H
a2.sinks.k2.hdfs.filePrefix = logs-
a2.sinks.k2.hdfs.round = true
a2.sinks.k2.hdfs.roundValue = 1
a2.sinks.k2.hdfs.roundUnit = hour
a2.sinks.k2.hdfs.useLocalTimeStamp = true
a2.sinks.k2.hdfs.batchSize = 200
a2.sinks.k2.hdfs.fileType = DataStream
a2.sinks.k2.hdfs.rollInterval = 600
a2.sinks.k2.hdfs.rollSize = 134217700
a2.sinks.k2.hdfs.rollCount = 0
a2.sinks.k2.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 300

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

启动

```
bin/flume-ng agent --conf conf/ --name a2 --conf-file job/hdfs-file.conf
```

写数据到文件

```
echo `date` >> /opt/flume/mydate.txt
```

到 hdfs 上查看文件生成。