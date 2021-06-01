---
title: Spark 运行模式
date: 2016-05-07 11:50:28
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
<!-- toc -->
Spark 程序是一组运行在集群上的独立进程，进程之间通过 SparkContext 协调。Spark 程序包含两部分：Driver 程序（SparkContext）和 Executor 。SparkContext 能连接多种类型的 Cluster Manager （spark standalone cluster、Mesos、YARN），虽然 cluster manager 的类型多样，但是 Spark 运行机制基本相同。<!--more-->Driver 程序由用户启动，通过资源调度模块和 Executor 通信。在连接到 Cluster Manager 后，Spark 获得集群 Worker 节点上的 Executor （负责计算和存储数据的进程），将需要运行的代码发送到 Executor ，然后 SparkContext 发送 Task 到 Executor 去执行。示意如图：

<img src="https://z3.ax1x.com/2021/06/01/2KYw9J.png" alt="234941814.png" border="0" />

每个 Spark 程序有自己的 Executor 进程，并且在程序生命周期内都是相互独立的，这也意味着不同的 Spark 程序之间无法共享数据，除非借助其他存储系统，如 HDFS 。

Spark 对于 Cluster Manager 是不可知的，只要能获取到 Executor 进程并能相互通信即可，所以更换Cluster Manager 并不需要修改程序。

在整个生命周期内，Worker 节点和 Driver 程序之间需要保持联通。

在距离 Worker 的机器上运行 Driver 程序会更有效率。

### Local 模式

本地运行，可快速验证代码。

### Spark Standalone 模式

Spark 提供的一种简单部署模式，通常用来进行测试。

### YARN 模式

通过 YARN 来调度 Spark 程序所需要的资源。YARN 模式有两种模式来启动 Spark 程序：YARN Cluster 和 YARN Client 。在 Cluster 模式中，Driver 程序运行在 master 进程中，程序初始化完成后 Client 可以结束。在 Client 模式中，Driver 程序运行在 client 进程中，master 只负责向 YARN 请求资源。

![](https://raw.githubusercontent.com/bsyonline/pic/master/spark-on-yarn.png)

YARN cluster：

1. Client 通过 YARN Client 向 Resource Manager 提交 Spark 程序，Resource Manager 在集群中启动一个 Spark Application Master ，注册到 Resource Manager 并初始化 SparkContext 。
2. Spark Application Master 通过 Resource Manager 在分配的 YARN 节点中启动 Container 。
3. 在 Container 中运行 Task 。

YARN client：

1.  Client 在本地初始化 SparkContext 。
2.  通过 YARN Client 在随机一个 YARN 节点上启动 Spark Application Master 。
3.  Spark Application Master 通过 Resource Manager 在分配的 YARN 节点中启动 Container 。
4.  在 Container 中运行 Task 。

### 几种模式比较

#### 环境变量传递

SparkContext 在初始化过程中需要将环境变量传递给 Executor ，如何将环境变量设置到 Executor 的环境中，取决于 Executor 的启动方式。

* Local 模式在单机运行，不存在环境变量传递的问题。
* Spark Standalone 模式中，环境变量被封装在 org.apache.spark.deploy.Command 类中，由 Spark Master 通过 Actor 转发给合适的 Worker ，Worker 通过 ExecutorRunner 构建 java.lang.Process ，再传递给 java.lang.ProcessBuilder.environment 。
* YARN 模式中，环境变量首先通过 YARN client 设置到 Spark Application Manager 的运行环境中，在通过 ExecutorRunnable 设置到运行 Executor 的 ContainerLaunchContext 中。

#### 包分发

* Local 和 Standalone 模式是单机或整个环境部署多个节点，所以不需要分发。


* YARN 模式中，运行库和程序运行依赖的文件首先通过 HDFS 客户端 API 上传到作业 .sparkStaging 目录下，然后将对应的文件和 URL 映射关系通知 YARN ，YARN 的 Node Manager 在启动 Container 的时候会从指定的 URL 下载文件。Spark Application Manager 在创建 Executor 的 Container 的时候通过 setLocalResources 设置 Executor 的环境变量。

### 术语表：

| Term            | Meaning                                  |
| --------------- | ---------------------------------------- |
| Application     | 由 Driver 程序和 Executors 组成。               |
| Application jar | 包含用户程序代码和依赖的 jar 包，Hadoop 和 Spark 的 libraries 应该排除在外，因为它们会在运行时被加入。 |
| Driver program  | 运行主函数的进程。                                |
| Cluster manager | 在集群中获取资源的扩展服务 (e.g. standalone manager, Mesos, YARN)。 |
| Deploy mode     | Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster. |
| Worker node     | 集群中任何可以运行 application code 的节点。          |
| Executor        | 在 Worker 节点上由程序启动的进程，负责执行 Task 和存储数据。    |
| Task            | 发送给 Executor 的一个 work 单元。                |
| Job             | 有多个 Task 组成由 Spark action 触发的并行计算单元。     |
| Stage           | 每个 job 中，相同的 task 组成的集合。Stage 之间相互依赖。    |