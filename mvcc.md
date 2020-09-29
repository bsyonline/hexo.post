---
title: MVCC
tags:
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-03-19 21:51:55
thumbnail:
---

数据库有 4 种隔离级别，RU 、RC 、RR 和 Serializable ，MySQL 默认为 Repeatable Read 。t1 表有两条记录，如果有两个事务并发操作 t1 表如下： 

<img src="http://bsyonline.gitee.io/pic/20200502/20200929221226.png" alt="20200929221405" border="0" style="width:500px">

Client1 开启事务，将记录 2 的 name 修改为 C 。Client2 也开启事务，查询 t1 。不管 Client1 的事务是否提交，Client2 查询的结果和 Client1 开启事务之前是相同的，等 Client2 提交之后才能看到 Client1 修改的内容。
如果将 MySQL 的隔离级别改成 Read Commited ，效果又会有不同。
<img src="http://bsyonline.gitee.io/pic/20200502/20200929221405.png" alt="20200929221405" border="0" style="width:500px">

这就是不可重复读的现象。

为什么当数据库的隔离级别为 RC 时，会出现不可重复读，通过修改隔离级别为 RR ，可以避免不可重复读呢？答案是因为在 MySQL 中，RC 和 RR 隔离级别是通过 MVCC 实现的。MVCC (Multi-Version Concurrency Control) 是 InnoDB 使用的一种并发控制协议，作用于 RC RR 两个级别。
#### MVCC 的作用
你一定想问，MVCC 有什么用呢？我们知道如果要实现并发，第一个想到的就是加锁。如果加锁，那么当 Client1 开启了事务，Client2 就无法获得锁，就会阻塞，这样性能就会很差。MVCC 并不会加锁，而是允许 Client2 在 Client1 事务期间依旧是可读的，这样就提高了读写的并发度。
#### MVCC 是如何实现的
对于 t1 表，我们看到的是这样的：

```
mysql> desc t1;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   | PRI | NULL    |       |
| name  | varchar(10) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

实际在 MySQL 中的样子这样的：

<img src="https://s1.ax1x.com/2020/03/20/86OpNR.png" alt="20200320100637" border="0" style="width:500px">

MySQL 会增加 3 个隐藏列：
1. DB_TRX_ID：表示插入或更新的的最后一个事务 id 。
2. DB_ROLL_PTR：回滚指针，指向 Undo Log 的记录。
3. DB_ROW_ID：行 id ，单调递增。

每次开启事务，都会生成对应的 Undo Log 。在事务中做的任何修改都会在 Undo Log 中保存下来，类似链表。然后在 select 的时候会生成对应查询视图 ReadView 。**ReadView 由当前未提交的事务 id 列表和最大事务 id 组成**。用 select 的结果的 DB_TRX_ID 和 ReadView 比较来判断事务的状态，从而确定返回的结果集。比较规则如下：
1. 当 DB_TRX_ID 小于 ReadView 的最小 ID ，则说明事务已经提交，则数据是可见的。
2. 当 DB_TRX_ID 大于等于 ReadView 的最小 ID 小于等于 ReadView 的最大 ID 时，
    2.1) 如果 DB_TRX_ID 等于当前的 DB_TRX_ID ，说明是自己的事务，则数据可见。 
	2.2) 如果 DB_TRX_ID 在 ReadView 的未完成事务列表中，说明事务未提交，则数据不可见。如果不在，则说明事务已提交，数据可见。
	
**在 RR 隔离级别下，ReadView 会延用事务第一次生成的 ReadView ，而 RC 隔离级别下，每次查询会生成新的 ReadView **。
有了上边的概念，我们来分析一下前边的例子，假设 t1 表中有 2 条记录，是由 DB_TRX_ID=100 的事务插入的，事务已经提交。

<img src="http://bsyonline.gitee.io/pic/20200502/20200929230446.png" alt="20200929230446" border="0" style="width:500px">


##### 首先看 RR 级别：
&ensp;&ensp;1)  Client1 开启了事务，假设它的 DB_TRX_ID=200 。Client1 修改了 id=12 的 name 为 C ，MySQL 会将 id=12 这条记录拷贝到 Undo Log 中，然后将 DB_ROLL_PTR 指向 Undo Log 中的这条记录，然后将 name 修改成 C ，这样就形成一个版本链。

<img src="http://bsyonline.gitee.io/pic/20200502/20200929230614.png" alt="20200929230446" border="0" style="width:700px">

&ensp;&ensp;2)  当 Client1 查询时，会生成查询视图 ReadView([200],200) 。id=11 这条记录的 DB_TRX_ID=100 ，小于最小的未提交 DB_TRX_ID ，所以 id=11 这条记录是可见的。id=12 这条记录的 DB_TRX_ID=200 ，等于最大的未提交 DB_TRX_ID 说明是自己的事务，所以也是可见的。所以查到的结果就是 A 和 C 。
&ensp;&ensp;3)  当 Client2 开启了事务，假设它的 DB_TRX_ID=300 ，生成对应的查询视图 ReadView([200],300) 。Client2 执行查询时，id=11 这条记录的 DB_TRX_ID=100 ，小于最小的未提交 DB_TRX_ID ，所以 id=11 这条记录是可见的。id=12 这条记录的 DB_TRX_ID=200 ，在未提交列表中，所以需要去 Undo Log 中找历史版本 DB_TRX_ID=100 ，最终查到的结果是 A 和 B 。
&ensp;&ensp;4)  当 Client1 提交后，Client2 再次查询，ReadView 还是 ([200],300) ，所以查到的结果还是 A 和 B 。
##### 与此类似，我们再来看看 RC 级别： 
&ensp;&ensp;1) 、 2） 、3）和上边是相同的。
&ensp;&ensp;4)  当 Client1 提交后，Client2 再次查询，此时 ReadView 是新生成的 ([300],300) ，DB_TRX_ID=200 已经提交，所以 C 是可见的，最终查到的结果是 A 和 C 。


>MVCC 只作用于 RC 和 RR ，那 RU 和 Serializable 是如何实现的？
RU 每次都读最新记录。
Serializable 通过互斥锁。
