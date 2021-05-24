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



## Part I. Hadoop Fundamentals

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

HDFS 中的块丢失率达到阈值（0.1%）时，NameNode 就会进入安全模式。

NameNode 在刚启动时也会处于安全模式下。

在安全模式下，客户端只能读，不能写。

##### 3.2.4 数据块

HDFS 读写数据的最小单位，默认大小 128MB 。文件被划分成多个块进行存储，小于块大小的文件不会占据整个块的空间。HDFS 的块不能太大也不能太小，太小会增加寻址时间，太大 MapReduce 的任务数太少（一个 map 通常只处理一个块），增加任务运行时长。

使用数据块的优点：

1. 文件大小可以大于磁盘大小。
2. 简化了存储子系统的设计。将数据和元数据分离进行管理。
3. 适合用于备份和容错，提高可用性。

##### 3.2.5 元数据管理

```
${dfs.namenode.name.dir}/
├── current
│   ├── VERSION
│   ├── edits_0000000000000000001-0000000000000000019
│   ├── edits_inprogress_0000000000000000020
│   ├── fsimage_0000000000000000000
│   ├── fsimage_0000000000000000000.md5
│   ├── fsimage_0000000000000000019
│   ├── fsimage_0000000000000000019.md5
│   └── seen_txid
└── in_use.lock
```

VERSION 记录了 HDFS 集群的版本信息。

```
#Mon Sep 29 09:54:36 BST 2014
namespaceID=1342387246	#文件系统命名空间的唯一标识，在 namenode 首次格式化时创建
clusterID=CID-01b5c398-959c-4ea8-aae6-1e0d9bd8b142	#HDFS 集群的唯一标识
cTime=0	# 存储系统的创建时间，刚格式化就是0
storageType=NAME_NODE
blockpoolID=BP-526805057-127.0.0.1-1411980876842	#数据块池的唯一标识
layoutVersion=-57	#描述 HDFS 持久性数据结构的版本，变更就会递减
```

编辑日志

HDFS 的写操作通过预写日志 WAL 来更新，首先会写到 edits 文件（编辑日志）中。

edits 文件是磁盘上的多个文件，每一个文件称为一个段（segment）。

文件系统镜像

edits 文件会被定期合并成 fsimage 文件。

checkpoint 

SecondaryNameNode 定期会将 NameNode 的上的 edits 和 fsimage 下载到本地，然后加载到内存进行合并，这个过程叫作 checkpoint 。

<a href="https://imgtu.com/i/gi4zcT"><img src="https://z3.ax1x.com/2021/04/28/gi4zcT.png" alt="gi4zcT.png" border="0" /></a>

在故障恢复时，fsimage 文件会被加载到内存以恢复元数据到最近状态，再从该点向前执行编辑日志中的每个事务。

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

<a href="https://imgtu.com/i/gioMtA"><img src="https://z3.ax1x.com/2021/04/28/gioMtA.png" alt="gioMtA.png" border="0" /></a>

1. 客户端调用 DistributedFileSystem 的 create() 方法新建文件。
2. DistributedFileSystem 通过 RPC 调用 NameNode ，在文件系统的命名空间中创建一个文件。
3. 检查权限，通过后创建一条新建文件记录。
4. DistributedFileSystem 返回 FSDataOutputStream 给客户端，客户端开始写数据。
5. 要写入数据的 DataNode 组成 data queue 和 ack queue，FSDataOutputStream 将数据分成 packet 写入 data queue ，写入数据的 ack 会写入 ack queue 。
6. 客户端完成写入后，调用 FSDataOutputStream 的 close() 方法将剩余数据写入 data queue 。
7. 客户端通知 NameNode 写入完成。

##### 3.3.3 副本存放策略

HDFS 数据块和副本的存放是保证可靠性的关键。每个数据块都有多个副本，存储在不同的机器上。

<a href="https://imgtu.com/i/gio31P"><img src="https://z3.ax1x.com/2021/04/28/gio31P.png" alt="gio31P.png" border="0" /></a>

副本存放策略：

1. 第一个副本在和客户端相同的节点上。
2. 第二个副本在第一个副本的存储节点相邻的机架的任意节点上。
3. 第三个副本存储在第二个副本所在机架的不同节点。

##### 3.3.4 一致性

HDFS 新建的文件是立即可见的，但是写入的内容不是。当写入内容超过一个块，对于新的 reader 才是可见的。hflush() 可以使写入内容立即可见，数据在 DataNode 的内存中但是不一定写到磁盘了，可能存在丢失。hsync() 可以确保数据写入磁盘。hflush() 和 hsync() 都会有额外性能损耗。

### 4. MapReduce

##### 4.1 MapReduce 作业运行机制

<a href="https://imgtu.com/i/gioGX8"><img src="https://z3.ax1x.com/2021/04/28/gioGX8.png" alt="gioGX8.png" border="0" /></a>

1. 客户端程序创建一个 JobSummiter 实例。
2. JobSummiter 向 ResourceManager 申请 Application ID 。
3. JobSummiter 将运行作业需要的资源复制到共享文件系统中，路径以 Application ID 命名。
4. 提交作业，提交之后 waitForCompletion() 每秒轮询作业进度。
5. ResourceManager 将请求交给 Yarn 调度器，调度器分配 NodeManager 来运行容器并启动 application master —— MRAppMaster 。
6. MRAppMaster 对作业进行初始化，创建 bookkeeping objects 完成作业进行跟踪。
7. MRAppMaster 从共享文件系统获取输入分片，并对每个分片创建一个 map 任务及多个 reduce 任务，同时分配任务 ID 。
8. 如果需要在其他节点运行任务， MRAppMaster 向 ResourceManager 请求资源。先请求 map 资源，再请求 reduce 资源。
9. ResourceManager 为任务分配资源，MRAppMaster 再NodeManager 上创建容器。
10. YarnChild 将任务需要的资源本地化，再运行 map 或 reduce 任务。 
11. map 或 reduce 任务运行时，YarnChild 和 MRAppMaster 通过 umbilical 接口通信，每隔 3 秒 向MRAppMaster 上报进度和状态。
12. 当 MRAppMaster 收到作业最后一个任务的完成通知，会更新作业状态，waitForCompletion() 方法退出。

##### 4.2 Shuffle

经过排序，将 map 的输出作为输入传给 reduce 的过程叫做 shuffle 。shuffle 分为 map 端和 reduce 端。

<a href="https://imgtu.com/i/giorcV"><img src="https://z3.ax1x.com/2021/04/28/giorcV.png" alt="giorcV.png" border="0" /></a>

在 map 端，每个 map 任务都有一个环形缓冲区，默认大小 100 MB 。map 将数据先写到缓冲区，当缓冲区的数据达到阈值，后台线程就将缓冲区的数据写到磁盘，这个过程叫溢写。在溢写过程中，map 还可以继续往缓冲区中写数据，如果缓冲区满了，map 会被阻塞。

在写磁盘之前，根据 reduce 把对数据进行 partition 。在每个 partition 中进行内存排序。

如果有 combiner ，则会在排序之后运行。如果有溢写的文件数量较多，会多次运行 combiner 。

每次溢写会生成一个新的文件，这些文件会被合并成输出文件。

默认情况下，map 的输出是不压缩的， 但可以通过配置开启。压缩可以减少传入 reduce 的数据量。

在 reduce 端，通过 MRAppMaster 获取 map 的位置，通过复制线程并行地将 map 输出复制到 reduce 的内存或磁盘。

复制完所有 map 输出后，reduce 对 map 输出循环的进行合并，并会保留排序。

最后 reduce 阶段，将数据输入 reduce 函数，结果会输出到 HDFS 。

##### 4.3 MapReduce 的类型

map ：(K1, V1) ->  LIST(K2, V2)

combiner : (K2, LIST(V2))  -> LIST(K2, V2) 

reduce : (K2, LIST(V2)) -> LIST(K3, V3)

##### 4.4 计数器

任务计数器

由任务维护，定期发送给 application master 。

作业计数器

由 application master 维护。

用户定义的计数器

##### 4.5 排序

局部排序

根据输入的键排序。

全局排序

使用单个 partition ，但效率极低。更好的方式是使用 partitioner ，先分区排序，再串联。

##### 4.6 连接

map 端连接

连接操作由 mapper 执行。

执行 map 端连接的要求：

1. 各 map 的输入数据必须先分区且已排序。
2. 各输入数据集被划分成相同数量的分区，且按连接键排序。
3. 同一个键的所有记录会放在同一个分区。

reduce 端连接

连接操作由 reducer 执行。reduce 端没有输入数据集类型限制，所以 reduce 端的连接更常用，但是两个数据集在到达 reduce 之前都需要经历 shuffle ，所以 reduce 端连接的效率要低一些。通常使用连接键作为 map 的输出键，使相同的记录放到同一个 reduce

##### 4.7 序列化

基于 writable 实现。

<a href="https://imgtu.com/i/gio86f"><img src="https://z3.ax1x.com/2021/04/28/gio86f.png" alt="gio86f.png" border="0" /></a>

##### 4.8 输入输出格式

基于文件

基于文本

基于数据库

多个输入输出

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
4. application master 获得起源后，启动 container 执行作业。

##### 5.2 调度

Yarn 有 3 种调度器

1. FIFO Scheduler 先入先出调度器
2. Capacity Scheduler 容量调度器
3. Fair Scheduler 公平调度器

## Part II. Hadoop Related Projects

### 6. Hive



### 7. HBase



### 8. Flume

