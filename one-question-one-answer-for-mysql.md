---
title: One Question One Answer for MySQL
tags:
  - MySQL
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-07-28 11:26:51
thumbnail:
---



Q: 什么是索引

A: 索引是关系数据库中的一种存储数据结构，针对数据库表的一列或多列进行排序，从而快速定位，加速检索。



Q: MySQL 有哪些索引

A: 主键索引， 唯一索引，哈希索引，fulltext 索引。



Q: B+Tree 索引和Hash 索引的区别

A: B+Tree 索引可以用于列相等的比较，范围查找，Like 前半部分的比较，B+Tree 的任何最左前缀都可以用来查找。Hash 索引仅能用于相等的比较，不能用于范围查找，但是速度更快。



Q: 加了索引为什么没有生效

A: 索引是否生效，可以通过执行计划进行查看。

在一些情况下即使加了索引，也是不会生效的，比如：
1. 关联表使数据类型不相同或大小不同；
2. 字符串进行比较时字符集不同；
3. 数据类型不同的列，没有进行转换的情况下。

最终是否使用索引，是由查询优化器决定的，会根据检索条件，全表扫描的代价，各种执行方案对比等最终选择一个优化器觉得成本最低的方案执行。


Q: B+Tree 数据结构都可以存储哪些内容

A: BTree 节点可以存放 key 和数据，B+Tree 的内节点只能存放 key ，数据只能存放在叶子节点。



Q: 为什么使用 B+Tree 而不使用二叉树或是红黑树

A: 和计算机处理数据的原理有关系。计算机会在两个地方存储数据，一个是内存，一个是磁盘。当我们使用的数据没有在内存中时，计算机就会报一个缺页异常，这时候内存和磁盘之间就会进行数据交换。计算机会在磁盘上找到数据存储的位置。由于磁盘在读取数据成本很高，为了提高性能，通常计算机会以这个位置为开始位置，向后预读页的整数倍大小数据，将这些数据加载到内存。如果使用二叉树或是红黑树，树的高度会很大，逻辑上很近的数值可能在物理上很远，预读的优势无法体现。BTree 相对来说，出度很大，树的高度相对较小，逻辑相邻的数据物理上也相距很近，可以充分发挥预读的优势。B+Tree 相对于 BTree 来说中间节点只存储 key ，占用空间更小，所以在相同大小的页中包含的节点就更多，一次 I/O 预读的数据就更多。



Q: 什么是聚集索引和非聚集索引，他们有什么区别

A: 在 InnoDB 存储引擎中，B+Tree 的叶子节点保存了完整的数据这种方式叫做聚集索引。MyISAM 存储引擎中 B+Tree 在叶子节点存储的是数据的地址信息，这种方式叫做非聚集索引（二级索引）。如果一个表有多个列使用非聚集索引，那么主键索引和非主键索引结构上是一致的。如果使用聚集索引，那么主键索引的叶子节点存储了完整的数据信息，非主键索引的叶子节点存储的是列信息和主键的信息。



Q: 什么是回表

A: 在使用非主键索引查询时，通过索引查到主键，然后在通过主键进行一次查询，这个过程叫做回表。使用非主键索引进行查询时，可以使用索引覆盖（covering index）来避免回表。



Q: 什么是索引下推

A: 不使用索引下推时，查询结果拿到 server 层，回表之后再根据 where 条件进行过滤。使用索引下推，在 innoDB 层就对查询结果进行过滤，过滤之后的结果返回到 server 层进行回表。使用索引下推优化可以减少回表的数量。



Q: InnoDB 和 MyISAM 的区别。

A: <table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">InnoDB</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">MyISAM</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持事务</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">不支持</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">锁</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">行级锁</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">表级锁</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持MVCC</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">不支持</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持外键</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">不支持</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持全文索引</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">不支持</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">支持</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">索引</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">聚簇索引</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">非聚簇索引</td></tr></table>



Q: 什么情况下应该或不应该使用索引。

A: 
1. 记录数较少，不用使用索引。
2. 频繁写的表，不应该使用索引。
3. 数据重复且分布平均，比如性别字段，不应该使用索引。



Q: 隔离级别

A: 隔离级别有 4 种：
1. Read Uncommitted: 最低级别，任何情况都无法保证。
2. Read Committed: 可避免脏读的发生。
3. Repeatable Read: 可避免脏读、不可重复读的发生。
4. Serializable: 可避免脏读、不可重复读、幻读的发生。


Q: 什么是 MVCC ?

A: MVCC (Multi-Version Concurrency Control) 是 InnoDB 使用的一种并发控制协议。在 RC RR 两种隔离级别下，select 会查询版本链中的数据，从而实现读写并发。与之相对的是 LBCC (Lock-Based Concurrency Control) 。



Q: SQL语言包括哪几类？

A: DDL ，DCL ，DML ，DQL ，TPL 。


Q: 脏读、幻读、不可重复读

A: 脏读：事务 T1 将某一值修改，然后事务 T2 读取该值，然后 T1 回滚，导致 T2 所读取到的数据是无效的。
幻读：事务 T1 修改了全表数据，然后事务 T2 插入了一条新记录，对 T1 来说还有一条记录没有改。
不可重复读：事务 T1 查询一次，然后事务 T2 修改了一条记录，然后 T1 再次查询，两次查询得到的结果不一致。



Q: delete、drop、truncate 区别？

A: 删除表用 drop ，删除表中的全部数据用 truncate ，删除表的部分数据用 delete 。drop 和 truncate 是 DDL ，不能回滚，delete 是 DML ，可以回滚。



Q: MySQL 有哪些锁？

A: MySQL 锁分为 lock 和 latch 。lock 针对的是事务，用来锁定数据库中的对象，latch 针对的是线程，用来保证并发的正确性。latch 是一种轻量级锁，又分为互斥锁 (mutex) 和 读写锁 (rw-lock) 。lock 又分为 表锁，页锁，行锁。行锁里又分 record lock (行锁，锁定一行)，gap lock (间隙锁，锁定范围行) ，next-key lock (临键锁，record lock + gap lock) 。
按模式分成共享锁和独占锁。
按使用机制分成乐观锁和悲观锁。

Q: 什么是 ACID？

A: 事务要么同时成功要么同时失败，这是原子性 atomic 。成功了 commit 失败了 rollback ，不管 commit 还是 rollback ，一旦执行，结果就是不可变的，这是持久性 Durability 。在事务开始和结束之间，存在中间状态，这些中间状态对事务之外是不可见的，这是隔离性 isolation 。事务前后数据必须是完整一致的，不能存在中间状态，这是一致性 consistency 。

Q: 什么是 B+ 树？

A: B+ 树是一种出度很大，高度很小，已排序的 n 叉树。

Q: AUTO_INCREMENT 的原理？

A: 传统的方式，MySQL 会维护一个计数器来记录当前最大的 id 值，当新增一条记录时，先获取 AUTO-INC 锁直到语句结束。AUTO-INC 是一个表级锁，性能不好，所以有了改进方式。如果能确定插入行数，则使用轻量级的互斥锁，如果是批量插入，还是使用 AUTO-INC 锁。


Q: 组合索引和几个单个的索引有什么区别？

A: 组合索引在进行多个条件查询时可以根据最左前缀规则进行索引，单个索引在多个查询条件时，索引可能只会使用一个索引（实际还要看查询计划）。

Q: 临时表有什么用，什么时候删除？

A: 临时表用于保存临时数据，连接断开时会删除临时表。

Q: 数据库的3个范式是什么？

A: 第一范式：关系中的每个属性都不可再分。第二范式：要有主键。第三范式：消除冗余。

Q: MySQL 主从复制是怎么做的？

A: 当有从库连接到主库时，主库会创建一个线程将 Bin Log 发送到从库。从库创建线程接收 Bin Log 并回放 sql 。