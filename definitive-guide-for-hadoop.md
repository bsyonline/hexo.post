---
title: Definitive Guide for Hadoop
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-04-24 08:42:23
thumbnail:
---



# Part I. Hadoop Fundamentals



### 1. Hadoop 是什么

Hadoop 是一个开源的软件框架，包含基础组件 Common、分布式文件存储系统 HDFS 、分布式计算引擎 MapReduce 及2.0 以后加入的资源调度系统 Yarn 。 

常见的发行版本有 Apache、Cloudera、Hortonworks 。

### 2. 安装 Hadoop 及 WorkCount 示例 

#### 2.1 分布式集群搭建

#### 2.2 高可用集群搭建

#### 2.3 WordCount 示例

### 3. HDFS

#### 3.1 设计思想：

1. 将大文件切分成小文件，分块存储。
2. 一次写入，多次读取。
3. 运行在普通的机器上，通过冗余备份保证高可用。
4. 高吞吐，高延迟。
5. 单用户写入，只能追加。

#### 3.2 基本概念

##### 3.2.1 Master/Slave 架构

<a href="https://imgtu.com/i/czqUZd"><img src="https://z3.ax1x.com/2021/04/26/czqUZd.png" alt="czqUZd.png" border="0"></a>

Master 上有 NameNode 和 ResourceManager ，Slave 上有 DataNode 和 NodeManager 。一个 NameNode 管理多个 DataNode ，NameNode 管理元数据，DataNode 存储数据。如果 NameNode 宕机，整个集群将不可用，所以 NameNode 需要容错。Hadoop 有两种机制：

1. 备份元数据信息，比如在多个文件系统上保存元数据。
2. 使用 Secondary NameNode （高可用集群叫 Standby NameNode ）。Secondary NameNode 用来合并编辑日志和命名空间镜像。

##### 3.2.2 心跳机制

Master 启动时会启动一个 IPC server 等待 Slave 连接，Slave 启动时会连接到 Master 的 IPC server ，每隔 3 秒向 Master 发一次心跳。Slave 通过心跳将信息上报给 Master ，Master 通过心跳将命令发送给 Slave 。

NameNode 通过心跳获取 DataNode 的状态，ResourceManager 通过心跳获取 NodeManager 的信息。如果长时间（默认 630 秒）没有收到 Slave 的心跳，Master 就认为 Slave 宕机了。

##### 3.2.3 安全模式

HDFS 中的块丢失率达到阈值（0.1%）时，NameNode 就会进入安全模式。在安全模式下，客户端只能读，不能写。

##### 3.2.4 数据块

HDFS 读写数据的最小单位，默认大小 128MB 。文件被划分成多个块进行存储，小于块大小的文件不会占据整个块的空间。HDFS 的块不能太大也不能太小，太小会增加寻址时间，太大 MapReduce 的任务数太少（一个 map 通常只处理一个块），增加任务运行时长。

使用数据块的优点：

1. 文件大小可以大于磁盘大小。
2. 简化了存储子系统的设计。将数据和元数据分离进行管理。
3. 适合用于备份和容错，提高可用性。

##### 3.2.5 元数据管理

元数据信息会被保存在磁盘上的元数据镜像文件 fsimage 中，同时也会全部加载到内存中。通过预写日志 WAL 来更新，首先会写到 edits 文件中，再更新内存，edits 文件会被定期合并成 fsimage 文件。SecondaryNameNode 定期会将 NameNode 的上的 edits 和 fsimage 下载到本地，然后加载到内存进行合并，这个过程叫作 checkpoint 。

##### 3.2.6 高可用

为了解决 NameNode 的 SPOF 问题，Hadoop 2.0 以后加入了 HA 支持。通过 active-standby NameNode 实现。NameNode 之间通过共享存储（比如 QJM，quorum journal manager）实现编辑日志共享，始终由 active NameNode 写入日志。DataNode 需要同时向两个 NameNode 发送信息。standby NameNode 负责设置 checkpoint ，并在 active NameNode 失效时，自动切换成 active 。如果两个 NameNode 都失效了，只能冷启动。

##### 3.2.7 集群联邦

元数据信息是要加载到内存中的，随着集群规模的增长，内存会成为瓶颈。为了解决内存首先的问题，使用集群联邦。在 HDFS 集群中可以存在多个 NameNode ，每个 NameNode 存一部分元数据信息。

#### 3.3 I/O 操作

##### 3.3.1 文件读取

<a href="https://imgtu.com/i/gpf64U"><img src="https://z3.ax1x.com/2021/04/27/gpf64U.png" alt="gpf64U.png" border="0"></a>

1. 客户端调用 DistributedFileSystem 的 open() 方法打开读取的文件。
2. DistributedFileSystem 通过 RPC 调用 NameNode ，获得文件起始块的位置以及每个副本所在的 DataNode 的位置。
3. DistributedFileSystem 创建 FSDataInputStream 对象来管理客户端与 DataNode 的 I/O ，客户端调用。
4. FSDataInputStream 连接到第一个块所在的 DataNode ，通过 read() 方法将数据读取到客户端。
5. FSDataInputStream 关闭与 DataNode 的连接，通过询问 NameNode 获取下一个 DataNode 的位置并建立连接。
6. 客户端读取完成后，调用 FSDataInputStream 的 close() 方法关闭连接。

##### 3.3.2 文件写入

<a href="https://imgtu.com/i/gpfsEV"><img src="https://z3.ax1x.com/2021/04/27/gpfsEV.png" alt="gpfsEV.png" border="0"></a>

1. 客户端调用 DistributedFileSystem 的 create() 方法新建文件。
2. DistributedFileSystem 通过 RPC 调用 NameNode ，在文件系统的命名空间中创建一个文件。
3. 检查权限，通过后创建一条新建文件记录。
4. DistributedFileSystem 返回 FSDataOutputStream 给客户端，客户端开始写数据。
5. 要写入数据的 DataNode 组成 data queue 和 ack queue，FSDataOutputStream 将数据分成 packet 写入 data queue ，写入数据的 ack 会写入 ack queue 。
6. 客户端完成写入后，调用 FSDataOutputStream 的 close() 方法将剩余数据写入 data queue 。
7. 客户端通知 NameNode 写入完成。

##### 3.3.3 副本存放策略

HDFS 数据块和副本的存放是保证可靠性的关键。每个数据块都有多个副本，存储在不同的机器上。

<a href="https://imgtu.com/i/czqadA"><img src="https://z3.ax1x.com/2021/04/26/czqadA.png" alt="czqadA.png" border="0"></a>

副本存放策略：

1. 第一个副本在和客户端相同的节点上。
2. 第二个副本在第一个副本的存储节点相邻的机架的任意节点上。
3. 第三个副本存储在第二个副本所在机架的不同节点。

##### 3.3.4 一致性

HDFS 新建的文件是立即可见的，但是写入的内容不是。当写入内容超过一个块，对于新的 reader 才是可见的。hflush() 可以使写入内容立即可见，数据在 DataNode 的内存中但是不一定写到磁盘了，可能存在丢失。hsync() 可以确保数据写入磁盘。hflush() 和 hsync() 都会有额外性能损耗。

### 4. MapReduce



### 5. Yarn

##### 5.1 Yarn 和 MapReduce 1 比较

1. 组成

   MapReduce 1 由 jobtracker、tasktracker 和 slot 组成。Yarn 由 ResourceManager、application master、timeline server 及 container 组成。

2. 性能

   在 MapReduce 1 中，jobtracker 负责作业的调度和管理，因此会有性能瓶颈。Yarn 将这些职责拆分为 ResourceManager 、application master 、timeline server 等，每个负责不同的任务。

3. 资源利用率

   在 MapReduce 1 中，每个 tasktracker 都配有固定数量的 slot ，slot 是静态分配的，只能运行 map 任务或 reduce 任务。Yarn 是通过资源池管理资源，可按需请求资源。

4. 多租户

   Yarn 除了支持 MapReduce 任务，还支持其他类型的分布式任务。

##### 5.1 运行机制

<a href="https://imgtu.com/i/gpfyNT"><img src="https://z3.ax1x.com/2021/04/27/gpfyNT.png" alt="gpfyNT.png" border="0"></a>

1. 客户端向 ResourceManager 申请提交作业。
2. ResourceManager 寻找一个能够运行 application master 的节点，启动 container 运行 application master 。
3. application master 向 ResourceManager 申请运行作业的资源。
4. ResourceManager 寻找一个能够运行作业的的节点，启动 container 执行作业。

##### 5.2 调度

# Part II. Hadoop Related Projects



