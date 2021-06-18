---
title: MySQL Log
tags:
  - Interview
category:
  - MySQL
author: bsyonline
lede: 没有摘要
date: 2020-03-21 10:30:14
thumbnail:
---

#### Bin Log
Bin Log 是 MySQL Server 层的一种日志，用来保存数据库的更改（表的更改和数据的更改）事件。Bin Log 不会记录 select 操作。Bin Log 的作用主要用来做复制。
Bin Log 有 3 种格式：
1. statement ，记录每一条会修改数据的 sql 。日志量少，但是需要额外保存 sql 执行时的信息。
2. row ，记录修改的记录。日志量大，每一行数据的修改都会保存。
3. mixed 。

可以通过命令查看 MySQL 的 Bin Log 格式：
```
mysql> show global variables like '%binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
```

Bin Log 为什么不合适做异常恢复？
Bin Log 是通过 Bin Log 文件来恢复数据，这些数据是持久化到磁盘的，而异常掉电时，内存中的数据是没有写到磁盘的，所以内存中的这些数据是无法通过 Bin Log 来恢复的，会造成数据丢失。
Redo Log 记录的是内存中还没有写到磁盘的数据，所以 Redo Log 用来做异常恢复不会造成内存中的数据丢失。


#### Redo Log
Redo Log 是一种基于磁盘的数据结构，用来保证崩溃恢复期间未提交事务数据的正确性。Redo Log 包括两部分：Redo Log Buffer 和 Redo Log File 。事务日志先写到 Redo Log Buffer ，再刷到 Redo Log File 。有 3 种方式：
1. 每次提交都会将 Redo Log Buffer 中的数据写到 OS Buffer 并写入 Redo Log File 。默认。
2. 每次提交日志会写入 Redo Log Buffer，每秒从 Redo Log Buffer 刷到 OS Buffer 并写入 Redo Log File 。
3. 每次提交日志会写入 OS Buffer ，每秒从 OS Buffer 写入 Redo Log File 。

Redo Log 是以 block 为存储单位，包括 block header ，block tailer 和 block body 。

Redo Log 和 Bin Log 的区别
1. Bin Log 是 Server 层的日志，Redo Log 是 InnoDb 引擎的日志。
2. Bin Log 是逻辑日志，Redo Log 是物理日志。
3. Bin Log 是每次提交写入，Redo Log 是先写入 Redo Log Buffer ，然后再写入。
4. Bin Log 是追加写，Redo Log 是循环写。

#### Undo Log
Undo Log 是 Undo Log Record 的集合。Undo Log Record 包含事务回滚到最近一次修改的信息。每个事务有独立的 Undo Log 。Undo Log 存在于 Undo Log Segment ，Undo Log Segment 存在于 rollback segments 。
当事务提交时，Undo Log 不会立刻删除，会放在删除列表中，通过 purge 删除。
MySQL 利用 Undo Log 和 MVCC 来实现 RC 和 RR 隔离级别。