---
title: Definitive Guide for Flink
tags:
  - Flink
category:
  - Bigdata
author: bsyonline
lede: 没有摘要
date: 2021-05-30 18:31:35
thumbnail:
---



## Part I. Flink

### 1. Flink 简介



### 2. Flink 安装



### 3. 核心概念

架构

有状态

分布式

并行度

task与slot的关系

4层图结构

### 4. 数据源

文件

socket

集合

自定义

collector

### 5. 操作

map

filter

flatMap

union

collect

conMap

conFlatMap

split

select

### 6. sink

print

writeAsText

自定义

### 7. State

State 一般指一个具体的 task/operator 的状态。State 可以被记录，失败时可基于 State 进行恢复。

State 分为原始状态（raw state）和托管状态（managed state）。

原始状态：由用户自行管理具体数据结构，框架在做 checkpoint 时，使用 byte[] 来读写状态内容，并不关心内部的数据结构。

托管状态：由 Flink 框架管理的状态。

#### State 类型

State 有两种类型：

##### Operator State

Operator State 是 task 级别的状态，每一个 task 都对应一个 state 。比如 Kafka Connector Source 中，每个分区都会保存消费的 topic 的 offset 信息。

Operator State 只有 ValueState 一种。

##### Keyed State

Keyed State 记录每一个 key 的状态。有 6 种类型。

1. ValueState

2. ListState

3. MapState

4. ReducingState

5. AggregatingState

6. FoldingState

#### Statebackend

Flink 的 checkpoint 是通过 StateBackend 实现的。

StateBackend 可以在 flink-conf.xml 中进行全局进行配置，也可以在代码中配置。

基于内存（默认）

```java
// 内存方式
MemoryStateBackend memoryStateBackend = new MemoryStateBackend();
env.setStateBackend(memoryStateBackend);
```

基于文件

```java
// 文件方式
StateBackend fsStateBackend = new FsStateBackend("hdfs://hadoop1:8020/flink/checkpoint");
env.setStateBackend(fsStateBackend);
```

基于分布式数据库

```java
// rocksDB方式
StateBackend rocksDBStateBackend = new RocksDBStateBackend("hdfs://10.0.0.9:8020/flink/checkpoint");
env.setStateBackend(rocksDBStateBackend);
```

重启策略

Flink 的重启策略有：

1. noRestart
2. fixedDelayRestart（常用）
3. failureRateRestart

### 8. Watermark

定义

Time 的种类

1. Event Time：事件产生的时间，通常为事件中的时间戳
2. Ingestion Time：事件进入 Flink 的时间
3. Prosessing Time：事件被处理时当前系统的时间

有序数据



无序数据



延迟处理策略

watermark 可以触发比它小的窗口去执行。如果事件延迟太多，处理方式有三种：

1. 丢弃
2. allowedLateness 再次指定允许延迟时间
3. sideOutputLateData 收集延迟的数据

多并行度下的 WaterMark

在同一个线程中，大的 watermark 会覆盖小的 watermark 。

对于不同的线程， 取最小的 watermark 。

### 9. Window

概述

窗口划分

时间驱动 数据驱动

类型

tumblingwindows

slidingwindows

sessionwindows

globalwindows

window 操作

trigger

evictor

增量聚合

全量聚合

