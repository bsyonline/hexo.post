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

容错

checkpoint

重启策略



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

### 7. state

state 类型

Operator State

Keyed State

ValueState

ListState

MapState

ReducingState

AggregatingState

### 8. Watermark

定义

time的种类

有序数据

无序数据



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

