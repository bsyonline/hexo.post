---
title: 读书笔记：hadoop 权威指南（Hadoop The Definitive Guide）
date: 2015-11-19 15:45:44
tags:
 - Hadoop
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---


### 第3章 Hadoop分布式文件系统

#### 3.1 Hadoop的概念
##### 3.1.1 数据块
数据块默认大小64M，修改？？？
修改数据块的大小是为了减少磁盘寻址时间带来的消耗，但块过大会使map的数量变少（map一次处理一个block），MapReduce的效率降低。
##### 3.1.2 NameNode和DataNode
HDFS的特点是采用主从结构（Master/Slave）。NameNode存储2类文件：命名空间镜像文件和操作日志文件，用于记录文件系统数及树的文件和目录。NameNode是HDFS的核心，NameNode损坏将导致整个HDFS不可用。所以需要进行备份，备份的机制有2种：将NameNode信息同时写入NFS和使用SecondaryNameNode。SecondaryNameNode通常在单独节点运行，因为需要大量的CUP与和NameNode相同大小的内存用于进行命名空间合并。SecondaryNameNode的数据滞后于NameNode，所以在恢复时需要将NFS的数据拷贝到SecondaryNameNode。
##### 3.1.3 命令行
```
hadoop fs [-fs <local | file system URI>] [-conf <configuration file>]
[-D <property=value>] [-ls <path>] [-lsr <path>] [-du <path>]
[-dus <path>] [-mv <src> <dst>] [-cp <src> <dst>] [-rm [-skipTrash] <src>]
[-rmr [-skipTrash] <src>] [-put <localsrc> ... <dst>] [-copyFromLocal <localsrc> ... <dst>]
[-moveFromLocal <localsrc> ... <dst>] [-get [-ignoreCrc] [-crc] <src> <localdst>
[-getmerge <src> <localdst> [addnl]] [-cat <src>]
[-copyToLocal [-ignoreCrc] [-crc] <src> <localdst>] [-moveToLocal <src> <localdst>]
[-mkdir <path>] [-report] [-setrep [-R] [-w] <rep> <path/file>]
[-touchz <path>] [-test -[ezd] <path>] [-stat [format] <path>]
[-tail [-f] <path>] [-text <path>]
[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
[-chown [-R] [OWNER][:[GROUP]] PATH...]
[-chgrp [-R] GROUP PATH...]
[-count[-q] <path>]
[-help [cmd]]
```
