---
title: MySQL In Action
tags:
  - MySQL
category:
  - MySQL
author: bsyonline
lede: 没有摘要
date: 2099-05-05 14:00:50
thumbnail:
---




## Part I. 概念

### 1. MySQL 架构

<img src="https://z3.ax1x.com/2021/09/26/4yaqld.png" alt="4yaqld.png" border="0" />

连接器（connectors）
系统管理和控制工具（services & utilies）
连接池（connection pool）
sql接口（sql interface）
解析器（parser）

	1. 词法分析 分词，获取子句
	2. 语法分析 检查是否符合sql92语法
	形成语法树
查询优化器
	1. 索引优化
	2. 多表关联 小表驱动大表
	3. where条件，从左到右，选择过滤力度最大的条件先执行
查询缓存（cache buffer）
	执行的sql hash 作为key， 结果是value
	当数据改变，清除缓存
存储引擎
	5.6 默认 innoDB
	MyISAM 查询和插入速度快，但不支持事务，行锁，外键；
	InnoDB 支持事务，行锁，外键；

执行流程
 	1. 连接管理模块
	2. 连接进程模块
	3. 用户模块
	4. 命令分发器
	5. 记录日志
	6. 命令解析器
	7. 查询优化器
	8. 访问控制 grant
	9. 表管理模块
	10. 存储引擎接口

物理架构
	日志文件和索引文件 /var/lib/mysql
	日志用顺序IO方式存储，数据使用随机IO方式存储
	顺序IO和随机IO的优劣









innodb_file_per_table=OFF 表示表数据存储在共享表空间中，推荐的做法是设置为 NO，表示每个 InnoDB 表数据单独存储在 .ibd 文件中。



#### InnoDB 引擎的逻辑存储结构

表（tablespace）>段（segment）>区（extent）>页（page）。

删除数据表空间不会减小，空间会被标记为可重用。

表空间由段组成，数据段是 B+ 树的叶子节点，索引段是 B+ 树的非叶子节点。

一个区由 64 个连续的页组成。段开始不会立刻创建 64 个连续的页，会先创建 32 个碎片页，碎片页用完了，才会创建 64 个连续页，这有利于小表数据存放，节省空间。

页是磁盘管理的最小单位，默认16 KB 。页有很多类型，常见的有：

数据页

undo 页

系统页

事务数据页

插入缓冲位图页

插入缓冲空闲列表页

未压缩的二进制大对象页

压缩的二进制大对象页

**空洞的形成**

InnoDB 使用 B+ 树存储数据，如果一条记录被删除，该记录在B+树的节点会被标记为删除，但是磁盘空间不会被回收。如果再插入新的记录，可以复用这个空间。同样如果 delete 了全表的数据，则所有的数据页都是被标记为可复用，磁盘文件不会变小。没有被复用的空间就形成空洞。

插入数据同样会形成空洞，如果一个数据页已经满了，再插入新的数据，就会新申请一个新的数据页，造成页分裂。

更新也会造成空洞，旧值标记为删除，增加新值的索引数据。主键索引不会变，二级索引会变。

#### 重建表

重建表可以回收表空间。

如果我们自己操作，步骤如下：

1. 新建一个与表 A 结构相同的表 B
2. 按照主键 ID 递增的顺序，把数据一行一行地从表 A 里读出来再插入到表 B 中。

> 在往表 B 插入数据的过程中，表 A 有新的数据写入，就会造成数据丢失。为什么？
>
> 

如果让 MySQL 来回收表空间，可以使用下面语句重建表，MySQL 会自动完成转存数据、交换表名、删除旧表的操作。

```
alter table A engine=InnoDB
```

> 这两种有什么区别？
>
> 我们自己创建临时表，是在 server 层创建的。相当于 
>
> ```
> alter table t engine=innodb,ALGORITHM=copy;
> ```
>
> MySQL 重建是在 InnoDB 内部通过临时文件做的，这个也叫 inplace 。
>
> ```
> alter table t engine=innodb,ALGORITHM=inplace;
> ```

> optimize table、analyze table 和 alter table 这三种方式重建表的区别？
>
> alter table t engine = InnoDB 是重建表。
>
> analyze table t 是对索引重新统计，没有修改数据。
>
> optimize table t 等于 alter table + analyze table 。

online DDL 流程

1. 建立一个临时文件，扫描表 A 主键的所有数据页。
2. 用数据页中表 A 的记录生成 B+树，存储到临时文件中。
3. 生成临时文件的过程中，将所有对 A 的操作记录在 row log 中。
4. 临时文件生成后，将 row log 重放到临时文件中，形成最终结果。
5. 用临时文件替换表 A 的数据文件。

这个过程是允许对表 A 进行增删改操作的。

> 大表的重建推荐用 gh-ost 来做。

### 2. 索引

#### 什么是索引

索引是关系数据库中的一种存储数据结构，针对数据库表的一列或多列进行排序，从而快速定位，加速检索。

B+树是 MySQL 中使用最广泛的索引数据结构。B+树是一种出度很大，高度很小，已排序的 n 叉平衡树。

#### 索引分类

**按数据结构分**：B+树索引，Hash 索引，fulltext 索引。

B+树索引可以用于列相等的比较，范围查找，Like 前半部分的比较，B+树的任何最左前缀都可以用来查找。Hash 索引仅能用于相等的比较，不能用于范围查找，但是速度更快。

> B树节点可以存放 key 和数据，B+树的内节点只能存放 key ，数据只能存放在叶子节点。

> 为什么使用 B+树而不使用二叉树或是红黑树
>
> 和计算机处理数据的原理有关系。计算机会在两个地方存储数据，一个是内存，一个是磁盘。当我们使用的数据没有在内存中时，计算机就会报一个缺页异常，这时候内存和磁盘之间就会进行数据交换。计算机会在磁盘上找到数据存储的位置。由于磁盘在读取数据成本很高，为了提高性能，通常计算机会以这个位置为开始位置，向后预读页的整数倍大小数据，将这些数据加载到内存。如果使用二叉树或是红黑树，树的高度会很大，逻辑上很近的数值可能在物理上很远，预读的优势无法体现。B树相对来说，出度很大，树的高度相对较小，逻辑相邻的数据物理上也相距很近，可以充分发挥预读的优势。B+树相对于B树来说中间节点只存储 key ，占用空间更小，所以在相同大小的页中包含的节点就更多，一次 I/O 预读的数据就更多。

**按存储结构分**：聚簇索引，非聚簇索引。

在 InnoDB 存储引擎中，B+树的叶子节点保存了完整的数据这种方式叫做聚集索引。

在 MyISAM 存储引擎中 B+树在叶子节点存储的是数据的地址信息，这种方式叫做非聚集索引（二级索引）。如果一个表有多个列使用非聚集索引，那么主键索引和非主键索引结构上是一致的。如果使用聚集索引，那么主键索引的叶子节点存储了完整的数据信息，非主键索引的叶子节点存储的是列信息和主键的信息。

**按功能分**：主键索引， 唯一索引，普通索引。

普通索引和唯一索引的区别

查询效率基本无差别，更新效率有差别。

唯一索引的字段更新不会使用 change buffer ，普通索引的字段可以使用 change buffer 。如果目标页在内存中，那么这唯一索引和普通索引的更新效率不会有太大差别。如果目标页不在内存中，唯一索引的字段更新需要将数据页加载到内存中判断有没有冲突，没有冲突才能插入，普通索引的字段将更新操作缓存到 change buffer 中更新操作就结束了，最终 change buffer 在 merge 的时候才真正触发更新操作。所以这种情况下普通索引的更新效率要高于唯一索引。

> 如果在更新后立刻进行查询，那么在写入 change buffer 后立刻会触发 merge ，则 change buffer 的效果无法体现。

**按列个数分**：单列索引，组合索引。

组合索引和几个单个的索引有什么区别？

组合索引在进行多个条件查询时可以根据最左前缀规则进行索引，单个索引在多个查询条件时，索引可能只会使用一个索引（实际还要看查询计划）。

#### 什么情况下应该或不应该使用索引。

1. 记录数较少，不用使用索引。
2. 频繁写的表，不应该使用索引。
3. 数据重复且分布平均，比如性别字段，不应该使用索引。

#### 加了索引为什么没有生效

索引是否生效，可以通过执行计划进行查看。

在一些情况下即使加了索引，也是不会生效的，比如：

1. 关联表使用的列的数据类型不相同或大小不同；
2. 字符串进行比较时字符集不同；
4. 列进行了函数操作。

最终是否使用索引，是由查询优化器决定的，会根据检索条件，全表扫描的代价，各种执行方案对比等最终选择一个优化器觉得成本最低的方案执行。

#### 回表

在使用非主键索引查询时，通过索引查到主键，然后在通过主键进行一次查询，这个过程叫做回表。使用非主键索引进行查询时，可以使用索引覆盖（covering index）来避免回表。

#### 索引覆盖

从辅助索引中就可以查询到要查询的列。

#### 索引下推 ICP

不使用索引下推时，查询结果拿到 server 层，回表之后再根据 where 条件进行过滤。使用索引下推，在 innoDB 层就对查询结果进行过滤，过滤之后的结果返回到 server 层进行回表。

使用索引下推优化可以减少回表的数量，在查询计划的 extra 列显示 using index condition 。

#### 最左前缀原则



#### change buffer

当需要更新一个数据页时，如果数据页在内存中就直接更新，如果数据没有在内存中，就需要将数据页加载到内存中进行更新。在不影响一致性的前提下，InnoDB 会将更新操作缓存起来，这样就不需要再从磁盘加载数据页。当需要查询该数据页时，先将数据页加载到内存，再将之前缓存的更新操作与数据页进行 merge 。这个缓存叫做 change buffer 。change buffer 的前身是 insert buffer ，只能对 insert 进行优化，后来增加了 update/delete 的支持，改名叫 change buffer 。

change buffer 会持久化到 redo log 中。 所以即使 MySQL 宕机，也可以从 redo log 进行恢复。

change buffer 除了在读数据页到内存时触发 merge ，后台线程也会定期进行 merge 。

使用 change buffer 的条件：

1. 索引是非聚集索引

2. 索引不是唯一的

> change buffer 的提升体现在哪？
>
> 更新一条数据，如果数据页不在内存中，会先写 change buffer ，然后写 redo log 。change buffer 避免从磁盘随机读数据页，redo log （顺序写）避免了随机写。

#### order by

order by 的执行流程（单行长度比较小）：

1. 初始化 sort_buffer，确定要放入的查询字段。
2. 从索引中找到第一条满足条件记录的主键 id ，再到主键索引中取出整行，找出查询字段放入 sort_buffer 。
3. 从索引中取下一条记录的主键 id ，重复步骤 2 ， 直到找到不满足条件的记录为止。
4. 对 sort_buffer 中的数据按照排序字段做快速排序。
5. 按排序结果取 limit 行返回客户端。 

上边的流程是一行记录的长度小于 MySQL 的规定值，如果单行长度超过这个值，执行流程略有不同：

1. 初始化 sort_buffer，只放入排序字段和主键 id 。
2. 从索引中找到第一条满足条件记录的主键 id ，再到主键索引中取出整行，找出排序字段和主键 id 放入 sort_buffer 。
3. 从索引中取下一条记录的主键 id ，重复步骤 2 ， 直到找到不满足条件的记录为止。
4. 对 sort_buffer 中的数据按照排序字段做排序。
5. 按排序结果取 limit 行，并按照主键 id 回原表中取出查询字段返回客户端。 

> 修改一行记录长度的判定阈值：
>
> SET max_length_for_sort_data = 16；

### 3. 执行计划

Explain 会展示 MySQL 优化器关于语句执行计划的信息，也就是说会解释 MySQL 将如何处理语句。

#### 3.1 执行计划字段含义

我们先来试一下。

```
mysql> explain select * from t_user where name='Tom'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 10.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

Explain 结果显示了 12 列（黑体是我们需要重点关注的列）。

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">列</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">说明</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">id</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">序号</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"><b>select_type</b></td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">select 的类型</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">table</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">表名</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">partitions</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">匹配的分区</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"><b>type</b></td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">查找的类型</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">possible_keys</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">可能选择的索引</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">key</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">实际选择的索引</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">key_len</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">实际使用的索引的长度</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"><b>ref</b></td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">与索引比较的列</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">rows</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">读取的行数</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">filtered</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">按条件过滤后的行数占比</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"><b>Extra</b></td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">附加信息</td></tr></table><br>
##### **select_type**（重要）

select_type 有以下一些值：

**1. SIMPLE**

简单 SELECT（不使用 UNION 或子查询）。

```
mysql> explain select * from t_user\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```



**2. PRIMARY**

最外层 SELECT 。

```
mysql> explain select t1.* from t_user t1 where t1.id =(select id from t_user t2 where t2.name='user1')\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: no matching row in const table
*************************** 2. row ***************************
           id: 2
  select_type: SUBQUERY
        table: t2
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 10.00
        Extra: Using where
2 rows in set, 1 warning (0.02 sec)
```



**3. UNION**

UNION 中第二个及以后的 SELECT 语句。

```
mysql> explain select * from t_user t1 where t1.id=1 union select * from t_user t2 where t2.id=2\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 2
  select_type: UNION
        table: t2
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union1,2>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Using temporary
3 rows in set, 1 warning (0.01 sec)
```

union 的结果 UNION RESULT 放在临时表 <union1,2> 中，所以 id 为 NULL 。union 要对结果去重，所以需要临时表，union all 不需要去重，所以不需要临时表。

```
mysql> explain select * from t_user t1 where t1.id=1 union all select * from t_user t2 where t2.id=2\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 2
  select_type: UNION
        table: t2
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.00 sec)
```



**4. DEPENDENT UNION**

UNION 中的第二个及以后的 SELECT 语句，依赖外层查询。

```
mysql> explain select t1.* from t_user t1 where t1.id in (select id from t_user t2 union select id from t_user t3)\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t2
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: func
         rows: 1
     filtered: 100.00
        Extra: Using index
*************************** 3. row ***************************
           id: 3
  select_type: DEPENDENT UNION
        table: t3
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: func
         rows: 1
     filtered: 100.00
        Extra: Using index
*************************** 4. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union2,3>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Using temporary
4 rows in set, 1 warning (0.00 sec)
```

**5. UNION RESULT**

UNION 的结果。

```
mysql> explain select * from t_user where id=1 union select * from t_user where id=2\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t_user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 2
  select_type: UNION
        table: t_user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union1,2>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Using temporary
3 rows in set, 1 warning (0.00 sec)
```



**6. SUBQUERY**

子查询中第一个 SELECT 语句。

```
mysql> explain select * from t_employee where dept_id = (select dept_id from t_dept limit 1) \G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t_employee
   partitions: NULL
         type: ALL
possible_keys: i_dept_id
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 110
     filtered: 94.55
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: SUBQUERY
        table: t_dept
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 8
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)
```



**7. DEPENDENT SUBQUERY**

子查询中第一个 SELECT 语句，依赖外部查询。

```
mysql> explain select (select dept_name from t_dept where dept_id=e.dept_id),e.* from t_employee e \G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: e
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 110
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t_dept
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 8
          ref: test.e.dept_id
         rows: 1
     filtered: 100.00
        Extra: NULL
2 rows in set, 2 warnings (0.00 sec)
```



**8. DERIVED**

derived 表

```
mysql> explain select * from (select id from t_user group by id) A\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: <derived2>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 2
  select_type: DERIVED
        table: t_user
   partitions: NULL
         type: index
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)
```



**9. MATERIALIZED**

**10. UNCACHEABLE SUBQUERY**

子查询结果无法缓存，必须针对外部查询的每一行重新进行评估。

**11. UNCACHEABLE UNION**

UNION 中的第二个及以后的 SELETE 语句，并且这个 UNION 属于 UNCACHEABLE SUBQUERY 。

```
mysql> explain select * from t_user where gender in (select gender from t_user where id=1 union select gender from t_user where id=2)\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 101
     filtered: 100.00
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t_user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: 3
  select_type: DEPENDENT UNION
        table: t_user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 4. row ***************************
           id: NULL
  select_type: UNION RESULT
        table: <union2,3>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Using temporary
4 rows in set, 1 warning (0.00 sec)
```



##### **table**

table 对应的是表名，比如 t1 ，或者是临时表的名字。将查询的结果保存到临时表叫做物化。如果是物化的方式，则显示为 ```<derivedN>``` ，表示需要依赖 id 为 N 的查询。当使用 union 时显示 ```<union1,2>``` 。 

##### **type**（重要）

**1. system**

该表只有一行记录。system 是 const 的特例。

**2. const（重要）**

全表最多只有一行匹配。

```
mysql> explain select * from t_user where id=1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```



**3. eq_ref（重要）**

A 连接 B ，B 的连接字段是主键或唯一索引。

```
mysql> explain select * from t_dept a join t_employee b on a.dept_id=b.id\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_dept
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_employee
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 8
          ref: test.t_dept.dept_id
         rows: 1
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.01 sec)
```



**4. ref（重要）**

字段是普通索引。

```
mysql> explain select * from t_employee where dept_id=1 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_employee
   partitions: NULL
         type: ref
possible_keys: i_dept_id
          key: i_dept_id
      key_len: 9
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

等值的非主键连接。

```
mysql> explain select * from t_employee e join t_dept d where d.dept_name=e.name \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: d
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: ref
possible_keys: i_name
          key: i_name
      key_len: 203
          ref: test.d.dept_name
         rows: 1
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.00 sec)
```



**5. fulltext**

使用 fulltext 索引。

**6. ref_or_null**

类似 ref ，但是还会搜索包含 NULL 的行。

```
mysql> explain select * from t_user where skill='aaa' or skill is null\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 19.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

**7. unique_subquery**

**8. index_subquery**

**9. index_merge**

```
mysql> explain select * from t_user where name='Tom' or skill is null\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 19.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```



**10. range（重要）**

范围查询。

```
mysql> explain select * from t_user where id>10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

最左前缀匹配。

```
mysql> explain select * from t_user where name like 'a%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: range
possible_keys: i_name
          key: i_name
      key_len: 202
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```



**11. index（重要）**

查询字段全都有索引（即不用回表）。

```
mysql> explain select name from t_user\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: index
possible_keys: NULL
          key: name
      key_len: 1023
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.00 sec)
```



**12. ALL**

全表扫描，查询的字段不全都有索引，是在 server 层过滤数据。

```
mysql> explain select * from t_user\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```



##### **possible_keys**

优化器可能用到的索引。如果查询没有使用索引，则为 NULL ，可以在 where 中添加一些条件来使用索引。

##### key

优化器最终使用的索引。主键索引>唯一索引>普通索引

##### **key_len**

key_len 是实际使用的 key 的长度，tinyint 是 1 字节 ，smallint 是 2 字节，int 是 4 字节，bigint 是 8 字节，timestamp 是 3 字节，datetime 是 8 字节 ，char(n) 是 n 字节，varchar(n) 是 x\*n + 2 字节，x 根据字符集而定。

##### **rows**

rows 是查询计划预估的记录数，不是真正的结果行数。如果没有走索引，是全表扫描的行数，如果走索引，是预估的索引扫描行数。

##### **filtered**

filtered 主要用于表连接。

```mysql
mysql> explain select * from t_employee,t_dept where t_employee.dept_id=t_dept.dept_id\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_dept
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_employee
   partitions: NULL
         type: ALL
possible_keys: i_dept_id
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 9
     filtered: 25.00
        Extra: Using where; Using join buffer (Block Nested Loop)
2 rows in set, 1 warning (0.00 sec)
```

在使用表连接时，如果是 A left join B ，则 A 是驱动表，B 是被驱动表。如果是 A right join B ，则 B 是驱动表，A 是被驱动表。如果是 ```select * from A,B```  ，则小表是驱动表，大表是被驱动表。上边的例子，t_dept 是驱动表，t_employee 是被驱动表，t_employee 的连接行数 = 1 行 (t_dept rows \* t_dept filtered=1 \* 100% = 1) 。

##### Extra（重要）

**1. NULL**

查询的列没有被索引覆盖，必须通过回表进行查询。

```mysql
mysql> explain select * from t_user where name='tom'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ref
possible_keys: name
          key: name
      key_len: 1023
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

> name 有索引

**2. Using index**

只使用索引查询列的信息，这种情况一般都是使用了索引覆盖。

```mysql
mysql> explain select name from t_user where name='tom'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ref
possible_keys: name
          key: name
      key_len: 1023
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.00 sec)
```

> name 有索引

**3. Using where**

where 条件是索引之一。索引失效，在 server 层进行过滤。

```mysql
mysql> explain select name from t_user where name='tom' and age=10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
     filtered: 10.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

> age 没有索引

**4. Using index condition**

使用了索引下推优化。

```
mysql> explain select * from t_user where name='a' and age>10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t_user
   partitions: NULL
         type: range
possible_keys: i_name_age
          key: i_name_age
      key_len: 207
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

> name 和 age 是组合索引

**5. Using temporary**

查询使用了临时表，临时表创建和维护成本很高，所以一般需要考虑优化。

**6. Using filesort**

文件排序。filesort 耗时，一般需要考虑优化成索引排序。

**7. Using join buffer**

连接条件没有使用索引，需要优化。

#### 3.2 影响优化器的因素

执行计划可以看到 MySQL 执行使用的索引，选择索引就是由优化器决定的。优化器选择索引的目的是为了找到最优化的执行方案并用最小的代价执行。所以优化器的选择会受到扫描行数、是否使用临时表、是否排序等因素的综合影响。

##### 扫描行数

MySQL 优化器无法精确知道执行的记录数，只能根据索引的区分度进行估算。同一个索引上不同的值越多，区分度就越好。同一个索引上有哪些值，也是通过采样统计出来的。因此，这个结果很容易不准。

通常 MySQL 估算的行数与实际差别不大，也有差别比较大的时候，如果 explain 的结果和实际情况差距较大，可以使用 analyze table t1 来修正。

#### 3.3 索引选择异常处理

##### force index 

强制使用一个索引。如果指定了 force index ，MySQL 就不在评估其他索引。

缺点：1. 索引变更，sql 语句也需要变更。2. 数据库迁移可能不兼容。

##### 修改 sql 语句

通过修改 sql 语句，引导优化器选择合适索引。具体问题具体分析。

##### 删除无用索引

如果优化器选择了一个错误的索引，但是这个索引对业务没有什么用，可以删除索引让优化器选择正确的索引。





### 4. 日志

#### Binary Log

Binary Log 是 MySQL Server 层的一种日志，用来保存数据库的更改（表的更改和数据的更改）事件。Binlog 不会记录 select 操作。Binlog 的作用主要用来做复制。

##### Binlog 格式

Binlog 有 3 种格式：

1. statement ，记录每一条会修改数据的 sql 。日志量少，但是需要额外保存 sql 执行时的信息。但是由于相同的sql语句在主备执行结果可能不一致（delete from t1 where a>=xx and b>=xx limit 1），所以出现了 row 格式。
2. row ，记录修改的记录。日志量大，每一行数据的修改都会保存。row 格式不会造成主备不一致，但是可能会占用极大空间（比如删除 100 万条记录，就需要把这 100 万记录都存下来），所以就有了第三种方式 mixed 。
3. mixed 。MySQL 会自己判断 sql 语句是否会造成主备不一致，如果可能不一致就用 row ，如果不会就用 statement 。

可以通过命令查看 MySQL 的 binlog 格式：

```
mysql> show global variables like '%binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
```

##### 数据恢复

通常情况下用的较多的是 row 格式。因为 row 格式记录修改所有行的信息，在数据恢复时可以非常方便的操作。mixed 格式虽然可以利用 statment 格式的优点，同时又避免了数据不一致的风险，但是在数据恢复时需要结合上下文而不能直接回放 sql 。标准的做法是用 mysqlbinlog 工具解析之后，再把结果放到 MySQL 中执行。

##### 循环复制

循环复制就是在双M架构 AB 互为主备，A 的更新操作会生成一条 binlog ，这条 binlog 会发送给 B ，B 执行完 binlog 后也会生成一条 binlog ，B 生成的 binlog 又会发给 A ，A 又会再次执行更新，然后生成新的 binlog 再发给 B ，循环往复，造成循环复制。

如何解决循环复制？

给每条 binlog 加上一个 server_id （机器编号），A 生成 binlog ，server_id 假设记为 001 。binlog 发给 B 后，B 发现 001 和自己的机器编号不一致，则执行 binlog ，然后生成 binlog ，但是这个 binlog 的 server_id 还是 001 。当这个 binlog 在发给 A 后，A 发现 server_id 和自己的机器编号相同，则丢弃这条 binlog 。

##### Binlog 为什么不合适做异常恢复？

Binlog 是通过 Binlog 文件来恢复数据，这些数据是持久化到磁盘的，而异常掉电时，内存中的数据是没有写到磁盘的，所以内存中的这些数据是无法通过 Binlog 来恢复的，会造成数据丢失。
Redo Log 记录的是内存中还没有写到磁盘的数据，所以 Redo Log 用来做异常恢复不会造成内存中的数据丢失。

##### 怎么知道 binlog 是完整的？

statement 格式最后会有 COMMIT；row 格式最后会有 XID event 。

##### Binlog 的写入机制

先把日志写到 binlog cache，事务提交时再把 binlog  cache 写到 binlog 文件中，然后清空 binlog cache 。

一个事务的 binlog 不能被拆开，要确保一次写入。

每个线程都有自己的 binlog cache ，但是共享同一个 binlog 文件。

写 binlog 包括 2 个阶段的工作：

1. write page cache ，写系统内存
2. fsync ，持久化到磁盘

sync_binlog 参数控制这两个阶段的时机：

1. sync_binlog=0，表示每次提交事务都只 write，不 fsync；
2. sync_binlog=1，表示每次提交事务都会执行 fsync；
3. sync_binlog=N，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

##### Binlog 复制过程

1. 备库设置从主库什么位置开始同步 binlog 数据。
2. 备库启动两个线程 io_thread 和 sql_thread ，io_thread 负责和主库建立连接。
3. 建立连接后，主库读取 binlog 传给备库。
4. 备库收到 binlog 后写到本地 relay log 。
5. sql_thread 从 relay log 中读取日志，进行解析并执行。

#### Redo Log

Redo Log 是一种基于磁盘的数据结构，用来保证崩溃恢复期间未提交事务数据的正确性。Redo Log 包括两部分：Redo Log Buffer 和 Redo Log File 。事务日志先写到 redo log buffer ，再刷到 redo log file 。有 3 种方式：

1. 每次提交都会将 redo log buffer 中的数据写到 OS Buffer 并写入 redo log file 。默认。
2. 每次提交日志会写入 redo log buffer，每秒从 redo log buffer 刷到 OS Buffer 并写入 redo log file 。
3. 每次提交日志会写入 OS Buffer ，每秒从 OS Buffer 写入 redo log file 。

Redo Log 是以 block 为存储单位，包括 block header ，block tailer 和 block body 。

#### Redo Log 和 Bin Log 的区别：

1. Binlog 是 Server 层的日志，redo log 是 InnoDb 引擎的日志。
2. Binlog 是逻辑日志，redo log 是物理日志。
3. Binlog 是先写 binlog buffer，再写 binlog file ，redo log 是先写入 redo log buffer ，然后再写入 redo log file 。
4. Binlog 是追加写，Redo log 是循环写。
5. binlog 只在事务提交时写一次，redo log 在事务过程中都会写。

#### Undo Log

Undo Log 是 Undo Log Record 的集合。Undo Log Record 包含事务回滚到最近一次修改的信息。每个事务有独立的 Undo Log 。Undo Log 存在于 Undo Log Segment ，Undo Log Segment 存在于 rollback segments 。
当事务提交时，Undo Log 不会立刻删除，会放在删除列表中，通过 purge 删除。MySQL 利用 Undo Log 和 MVCC 来实现 RC 和 RR 隔离级别。

#### 两阶段提交



#### flush 操作

将内存中的脏页写到磁盘的过程就是 flush 。4 种情况会触发 flush 操作：

1. redo log 满了

   redo log 写满了，checkpoint 就需要前移，移动前后两点之间的日志对应的脏页都要刷到磁盘。

   此过程整个系统不再接收新的更新，直到 flush 结束。

2. 内存满了

   当内存不足时，系统需要淘汰旧的数据页（最久不使用）来创建新的数据页，淘汰的旧的数据页如果是脏页，就需要刷到磁盘。

3. 系统空闲时

4. MySQL 关闭时

> 内存中的数据页和磁盘数据页不一致时，内存的数据页成为脏页。脏页写入磁盘后，内存的数据页就和磁盘数据页一致了，内存数据页成为干净页。

##### flush 策略

1. innodb_io_capacity 参数控制 flush 的速度，建议设置成磁盘的 IOPS 。

> IOPS 可以通过 fio 工具测试
>
> ```
> fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest 
> ```

2. 脏页的阈值默认为 75%，应尽量保证不要经常接近阈值。
3. innodb_flush_neighbors=1 参数会将相邻的脏页一起刷到磁盘。机械硬盘 IO 不高，连带刷写相邻的脏页，能提高性能。SSD 的 IOPS 很高，不需要刷写相邻脏页，能减少响应时间。

### 5. 锁

#### 分类

MySQL 锁分为 lock 和 latch 。latch 是一种轻量级锁，又分为互斥锁 (mutex) 和 读写锁 (rw-lock) 。lock 又分为 表锁，页锁，行锁。行锁里又分共享锁（S Lock）和独占锁（X Lock）。

行锁的算法有 record lock (行锁，锁定一行)，gap lock (间隙锁，锁定范围行，但不包含本身) ，next-key lock (临键锁，record lock + gap lock) 。

lock 针对的是事务，用来锁定数据库中的对象，latch 针对的是线程，用来保证并发的正确性。

gap lock 为了解决幻读。


按使用机制分成乐观锁和悲观锁。

锁
	悲观锁
		表锁
		元数据锁（meta data lock）
		意向锁
		行锁
			共享锁
			排他写锁
			锁定范围
				记录锁record locks 主键指定
				间隙锁 gap locks 锁定记录前 记录中 记录后的行
				next-key 记录锁+间隙锁
			功能
				共享读锁 lock in share mode
				排他写锁

					1. dml 自动加
					2. for update
			两阶段锁：加锁阶段 解锁阶段
			没索引不支持行锁
			主键锁定一行，不产生间隙锁
			主键范围锁定，产生间隙锁



#### 加锁的原则：

查找过程中访问到的对象才会加锁，加锁的基本单位是 next-key lock ，next-key lock 是左开右闭区间。

索引上的等值查询，给唯一索引加锁时，next-key lock 退化为行锁。

索引上的等值查询，向右遍历时，且最后一个值不满足等值条件时，next-key lock 退化成间隙锁。

#### 一致性非锁定读和一致性锁定读

一致性非锁定读（consistent nonlocking read）：是指InnoDB存储引擎通过多版本控制的方式来读取当前行的数据。如果当前行有 X 锁，则会读取该行的快照数据。

一致性锁定读：加了锁定读操作。

select for update 对一行加 X 锁。

select lock in share mode 对一行加 S 锁。

必须在一个事务中。

#### 锁导致的问题：

脏读：一个事务读到了其他事务未提交的数据。

幻读：当同一条查询在不同的时间产生不同的结果集。不可重复读也属于幻读。

更新丢失：事务同时更新同一个记录，先提交的结果丢失。

### 6. 事务





#### 事务特性

事务要么同时成功要么同时失败，这是原子性 atomic 。

成功了 commit 失败了 rollback ，不管 commit 还是 rollback ，一旦执行，结果就是不可变的，这是持久性 Durability 。

在事务开始和结束之间，存在中间状态，这些中间状态对事务之外是不可见的，这是隔离性 isolation 。

事务前后数据必须是完整一致的，不能存在中间状态，这是一致性 consistency 。

#### 隔离级别

MySQL 有 4 种隔离级别，RU 、RC 、RR 和 Serializable ，默认为 Repeatable Read 。

1. Read Uncommitted: 最低级别，任何情况都无法保证。
2. Read Committed: 可避免脏读的发生。
3. Repeatable Read: 可避免脏读、不可重复读、幻读的发生。
4. Serializable: 可避免脏读、不可重复读、幻读的发生。

t1 表有两条记录，如果有两个事务并发操作 t1 表如下： 

<img src="https://z3.ax1x.com/2021/05/30/2VCorQ.png" alt="20200929221226.png" border="0" style="width:600px">

Client1 开启事务，将记录 2 的 name 修改为 C 。Client2 也开启事务，查询 t1 。不管 Client1 的事务是否提交，Client2 查询的结果和 Client1 开启事务之前是相同的，等 Client2 提交之后才能看到 Client1 修改的内容。
如果将 MySQL 的隔离级别改成 Read Commited ，效果又会有不同。

<img src="https://z3.ax1x.com/2021/05/30/2VCTbj.png" alt="20200929221405.png" border="0" style="width:600px">

这就是不可重复读的现象。

为什么当数据库的隔离级别为 RC 时，会出现不可重复读，通过修改隔离级别为 RR ，可以避免不可重复读呢？答案是因为在 MySQL 中，RC 和 RR 隔离级别是通过 MVCC 实现的。

#### MVCC 

**MVCC** (**M**ulti-**V**ersion **C**oncurrency **C**ontrol) 是 InnoDB 使用的一种并发控制协议。在 RC 和 RR 两种隔离级别下，select 会查询版本链中的数据，从而实现读写并发。与之相对的是 LBCC (Lock-Based Concurrency Control) 。

##### MVCC 的作用

你一定想问，MVCC 有什么用呢？我们知道如果要实现并发，第一个想到的就是加锁。如果加锁，那么当 Client1 开启了事务，Client2 就无法获得锁，就会阻塞，这样性能就会很差。MVCC 并不会加锁，而是允许 Client2 在 Client1 事务期间依旧是可读的，这样就提高了读写的并发度。

##### MVCC 是如何实现的

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

<img src="https://s1.ax1x.com/2020/03/20/86OpNR.png" alt="20200320100637.png" border="0" style="width:500px">

MySQL 会增加 3 个隐藏列：

1. DB_TRX_ID：表示插入或更新的的最后一个事务 id 。
2. DB_ROLL_PTR：回滚指针，指向 Undo Log 的记录。
3. DB_ROW_ID：行 id ，单调递增。

每次开启事务，都会生成对应的 Undo Log 。在事务中做的任何修改都会在 Undo Log 中保存下来，类似链表。然后在 select 的时候会生成对应查询视图 ReadView 。**ReadView 由当前未提交的事务 id 列表和最大事务 id 组成**。用 select 的结果的 DB_TRX_ID 和 ReadView 比较来判断事务的状态，从而确定返回的结果集。比较规则如下：

1. 当 DB_TRX_ID 小于 ReadView 的最小 ID ，则说明事务已经提交，则数据是可见的。
2. 当 DB_TRX_ID 大于等于 ReadView 的最小 ID 小于等于 ReadView 的最大 ID 时，
   2.1) 如果 DB_TRX_ID 等于当前的 DB_TRX_ID ，说明是自己的事务，则数据可见。 
   2.2) 如果 DB_TRX_ID 在 ReadView 的未完成事务列表中，说明事务未提交，则数据不可见。如果不在，则说明事务已提交，数据可见。

**在RR 隔离级别下，ReadView 会延用事务第一次生成的 ReadView ，而 RC 隔离级别下，每次查询会生成新的 ReadView**。
有了上边的概念，我们来分析一下前边的例子，假设 t1 表中有 2 条记录，是由 DB_TRX_ID=100 的事务插入的，事务已经提交。

<img src="https://z3.ax1x.com/2021/05/30/2VP8Rf.png" alt="20200929230446.png" border="0" style="width:500px"/>

首先看 RR 级别：

&ensp;&ensp;1)  Client1 开启了事务，假设它的 DB_TRX_ID=200 。Client1 修改了 id=2 的 name 为 C ，MySQL 会将 id=2 这条记录拷贝到 Undo Log 中，然后将 DB_ROLL_PTR 指向 Undo Log 中的这条记录，然后将 name 修改成 C ，这样就形成一个版本链。

<img src="https://z3.ax1x.com/2021/05/30/2VPdds.png" alt="20200929230614.png" border="0" style="width:700px"/>

&ensp;&ensp;2)  当 Client1 查询时，会生成查询视图 ReadView([200],200) 。id=1 这条记录的 DB_TRX_ID=100 ，小于最小的未提交 DB_TRX_ID=200 ，所以 id=1 这条记录是可见的。id=2 这条记录的 DB_TRX_ID=200 ，等于最大的未提交 DB_TRX_ID 说明是自己的事务，所以也是可见的。所以查到的结果就是 A 和 C 。
&ensp;&ensp;3)  当 Client2 开启了事务，假设它的 DB_TRX_ID=300 ，生成对应的查询视图 ReadView([200],300) 。Client2 执行查询时，id=1 这条记录的 DB_TRX_ID=100 ，小于最小的未提交 DB_TRX_ID=200 ，所以 id=11 这条记录是可见的。id=2 这条记录的 DB_TRX_ID=200 ，在未提交列表中，所以需要去 Undo Log 中找历史版本 DB_TRX_ID=100 ，最终查到的结果是 A 和 B 。
&ensp;&ensp;4)  当 Client1 提交后，Client2 再次查询，ReadView 还是 ([200],300) ，所以查到的结果还是 A 和 B 。

与此类似，我们再来看看 RC 级别： 

&ensp;&ensp;1) 、 2） 、3）和上边是相同的。
&ensp;&ensp;4)  当 Client1 提交后，Client2 再次查询，此时 ReadView 是新生成的 ([300],300) ，DB_TRX_ID=200 已经提交，所以 C 是可见的，最终查到的结果是 A 和 C 。

> MVCC 只作用于 RC 和 RR ，那 RU 和 Serializable 是如何实现的？
> RU 每次都读最新记录。
> Serializable 通过互斥锁。

## Part II. Action

### 1. MySQL安装

Windows 上使用 zip 版的 MySQL 配置服务方法：

配置

解压缩后，根路径创建配置文件 my.ini

```
# The following options will be passed to all MySQL clients  
[client]  
#password   = your_password  
port        = 3306  
[mysql]  
#设置mysql客户端的字符集  
default-character-set = utf8  
# The MySQL server  
[mysqld]  
port        = 3306  
#设置mysql的安装目录  
basedir = D:\Dev\MySQL\mysql-5.5.58
#设置mysql数据库的数据存放目录,必须是data或者\xxx-data  
datadir = D:\Dev\MySQL\mysql-5.5.58\data  
#设置服务器段的字符集  
character_set_server = utf8  
```

安装服务

使用管理员权限打开 CMD ，输入命令：

```
mysqld --install MySQL --defaults-file=D:\Dev\MySQL\mysql-5.5.58\my.ini
```

安装完成。

将 bin 目录加入环境变量即可在 CMD 中使用 MySQL 命令。

### 2. 备份与恢复

使用 mysqldump 命令

备份

```
mysqldump -uroot -p123456 -h 192.168.201.73 meta > meta_dump_`date +%F`.sql
```

压缩备份

```
mysqldump -uroot -p123456 -h 192.168.201.73 meta | gzip > backupfile.sql.gz
```

还原

```
mysql -uroot -p123456 -h192.168.201.74 meta < backupfile.sql
```

顺便贴一下 mysqldump 的帮助文档。

```
[ops@d20 ~]$ mysqldump
Usage: mysqldump [OPTIONS] database [tables]
OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
OR     mysqldump [OPTIONS] --all-databases [OPTIONS]
For more options, use mysqldump --help
[ops@d20 ~]$ mysqldump --hrlp
mysqldump: unknown option '--hrlp'
[ops@d20 ~]$ mysqldump --help
mysqldump  Ver 10.13 Distrib 5.6.31, for Linux (x86_64)
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Dumping structure and contents of MySQL databases and tables.
Usage: mysqldump [OPTIONS] database [tables]
OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
OR     mysqldump [OPTIONS] --all-databases [OPTIONS]

Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
The following groups are read: mysqldump client
The following options may be given as the first argument:
--print-defaults        Print the program argument list and exit.
--no-defaults           Don't read default options from any option file,
                        except for login file.
--defaults-file=#       Only read default options from the given file #.
--defaults-extra-file=# Read this file after the global files are read.
--defaults-group-suffix=#
                        Also read groups with concat(group, suffix)
--login-path=#          Read this path from the login file.
  -A, --all-databases Dump all the databases. This will be same as --databases
                      with all databases selected.
  -Y, --all-tablespaces
                      Dump all the tablespaces.
  -y, --no-tablespaces
                      Do not dump any tablespace information.
  --add-drop-database Add a DROP DATABASE before each create.
  --add-drop-table    Add a DROP TABLE before each create.
                      (Defaults to on; use --skip-add-drop-table to disable.)
  --add-drop-trigger  Add a DROP TRIGGER before each create.
  --add-locks         Add locks around INSERT statements.
                      (Defaults to on; use --skip-add-locks to disable.)
  --allow-keywords    Allow creation of column names that are keywords.
  --apply-slave-statements
                      Adds 'STOP SLAVE' prior to 'CHANGE MASTER' and 'START
                      SLAVE' to bottom of dump.
  --bind-address=name IP address to bind to.
  --character-sets-dir=name
                      Directory for character set files.
  -i, --comments      Write additional information.
                      (Defaults to on; use --skip-comments to disable.)
  --compatible=name   Change the dump to be compatible with a given mode. By
                      default tables are dumped in a format optimized for
                      MySQL. Legal modes are: ansi, mysql323, mysql40,
                      postgresql, oracle, mssql, db2, maxdb, no_key_options,
                      no_table_options, no_field_options. One can use several
                      modes separated by commas. Note: Requires MySQL server
                      version 4.1.0 or higher. This option is ignored with
                      earlier server versions.
  --compact           Give less verbose output (useful for debugging). Disables
                      structure comments and header/footer constructs.  Enables
                      options --skip-add-drop-table --skip-add-locks
                      --skip-comments --skip-disable-keys --skip-set-charset.
  -c, --complete-insert
                      Use complete insert statements.
  -C, --compress      Use compression in server/client protocol.
  -a, --create-options
                      Include all MySQL specific create options.
                      (Defaults to on; use --skip-create-options to disable.)
  -B, --databases     Dump several databases. Note the difference in usage; in
                      this case no tables are given. All name arguments are
                      regarded as database names. 'USE db_name;' will be
                      included in the output.
  -#, --debug[=#]     This is a non-debug version. Catch this and exit.
  --debug-check       Check memory and open file usage at exit.
  --debug-info        Print some debug info at exit.
  --default-character-set=name
                      Set the default character set.
  --delayed-insert    Insert rows with INSERT DELAYED.
  --delete-master-logs
                      Delete logs on master after backup. This automatically
                      enables --master-data.
  -K, --disable-keys  '/*!40000 ALTER TABLE tb_name DISABLE KEYS */; and
                      '/*!40000 ALTER TABLE tb_name ENABLE KEYS */; will be put
                      in the output.
                      (Defaults to on; use --skip-disable-keys to disable.)
  --dump-slave[=#]    This causes the binary log position and filename of the
                      master to be appended to the dumped data output. Setting
                      the value to 1, will printit as a CHANGE MASTER command
                      in the dumped data output; if equal to 2, that command
                      will be prefixed with a comment symbol. This option will
                      turn --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump - don't
                      forget to read about --single-transaction below). In all
                      cases any action on logs will happen at the exact moment
                      of the dump.Option automatically turns --lock-tables off.
  -E, --events        Dump events.
  -e, --extended-insert
                      Use multiple-row INSERT syntax that include several
                      VALUES lists.
                      (Defaults to on; use --skip-extended-insert to disable.)
  --fields-terminated-by=name
                      Fields in the output file are terminated by the given
                      string.
  --fields-enclosed-by=name
                      Fields in the output file are enclosed by the given
                      character.
  --fields-optionally-enclosed-by=name
                      Fields in the output file are optionally enclosed by the
                      given character.
  --fields-escaped-by=name
                      Fields in the output file are escaped by the given
                      character.
  -F, --flush-logs    Flush logs file in server before starting dump. Note that
                      if you dump many databases at once (using the option
                      --databases= or --all-databases), the logs will be
                      flushed for each database dumped. The exception is when
                      using --lock-all-tables or --master-data: in this case
                      the logs will be flushed only once, corresponding to the
                      moment all tables are locked. So if you want your dump
                      and the log flush to happen at the same exact moment you
                      should use --lock-all-tables or --master-data with
                      --flush-logs.
  --flush-privileges  Emit a FLUSH PRIVILEGES statement after dumping the mysql
                      database.  This option should be used any time the dump
                      contains the mysql database and any other database that
                      depends on the data in the mysql database for proper
                      restore.
  -f, --force         Continue even if we get an SQL error.
  -?, --help          Display this help message and exit.
  --hex-blob          Dump binary strings (BINARY, VARBINARY, BLOB) in
                      hexadecimal format.
  -h, --host=name     Connect to host.
  --ignore-table=name Do not dump the specified table. To specify more than one
                      table to ignore, use the directive multiple times, once
                      for each table.  Each table must be specified with both
                      database and table names, e.g.,
                      --ignore-table=database.table.
  --include-master-host-port
                      Adds 'MASTER_HOST=<host>, MASTER_PORT=<port>' to 'CHANGE
                      MASTER TO..' in dump produced with --dump-slave.
  --insert-ignore     Insert rows with INSERT IGNORE.
  --lines-terminated-by=name
                      Lines in the output file are terminated by the given
                      string.
  -x, --lock-all-tables
                      Locks all tables across all databases. This is achieved
                      by taking a global read lock for the duration of the
                      whole dump. Automatically turns --single-transaction and
                      --lock-tables off.
  -l, --lock-tables   Lock all tables for read.
                      (Defaults to on; use --skip-lock-tables to disable.)
  --log-error=name    Append warnings and errors to given file.
  --master-data[=#]   This causes the binary log position and filename to be
                      appended to the output. If equal to 1, will print it as a
                      CHANGE MASTER command; if equal to 2, that command will
                      be prefixed with a comment symbol. This option will turn
                      --lock-all-tables on, unless --single-transaction is
                      specified too (in which case a global read lock is only
                      taken a short time at the beginning of the dump; don't
                      forget to read about --single-transaction below). In all
                      cases, any action on logs will happen at the exact moment
                      of the dump. Option automatically turns --lock-tables
                      off.
  --max-allowed-packet=#
                      The maximum packet length to send to or receive from
                      server.
  --net-buffer-length=#
                      The buffer size for TCP/IP and socket communication.
  --no-autocommit     Wrap tables with autocommit/commit statements.
  -n, --no-create-db  Suppress the CREATE DATABASE ... IF EXISTS statement that
                      normally is output for each dumped database if
                      --all-databases or --databases is given.
  -t, --no-create-info
                      Don't write table creation info.
  -d, --no-data       No row information.
  -N, --no-set-names  Same as --skip-set-charset.
  --opt               Same as --add-drop-table, --add-locks, --create-options,
                      --quick, --extended-insert, --lock-tables, --set-charset,
                      and --disable-keys. Enabled by default, disable with
                      --skip-opt.
  --order-by-primary  Sorts each table's rows by primary key, or first unique
                      key, if such a key exists.  Useful when dumping a MyISAM
                      table to be loaded into an InnoDB table, but will make
                      the dump itself take considerably longer.
  -p, --password[=name]
                      Password to use when connecting to server. If password is
                      not given it's solicited on the tty.
  -P, --port=#        Port number to use for connection.
  --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
  -q, --quick         Don't buffer query, dump directly to stdout.
                      (Defaults to on; use --skip-quick to disable.)
  -Q, --quote-names   Quote table and column names with backticks (`).
                      (Defaults to on; use --skip-quote-names to disable.)
  --replace           Use REPLACE INTO instead of INSERT INTO.
  -r, --result-file=name
                      Direct output to a given file. This option should be used
                      in systems (e.g., DOS, Windows) that use carriage-return
                      linefeed pairs (\r\n) to separate text lines. This option
                      ensures that only a single newline is used.
  -R, --routines      Dump stored routines (functions and procedures).
  --set-charset       Add 'SET NAMES default_character_set' to the output.
                      (Defaults to on; use --skip-set-charset to disable.)
  --set-gtid-purged[=name]
                      Add 'SET @@GLOBAL.GTID_PURGED' to the output. Possible
                      values for this option are ON, OFF and AUTO. If ON is
                      used and GTIDs are not enabled on the server, an error is
                      generated. If OFF is used, this option does nothing. If
                      AUTO is used and GTIDs are enabled on the server, 'SET
                      @@GLOBAL.GTID_PURGED' is added to the output. If GTIDs
                      are disabled, AUTO does nothing. If no value is supplied
                      then the default (AUTO) value will be considered.
  --single-transaction
                      Creates a consistent snapshot by dumping all tables in a
                      single transaction. Works ONLY for tables stored in
                      storage engines which support multiversioning (currently
                      only InnoDB does); the dump is NOT guaranteed to be
                      consistent for other storage engines. While a
                      --single-transaction dump is in process, to ensure a
                      valid dump file (correct table contents and binary log
                      position), no other connection should use the following
                      statements: ALTER TABLE, DROP TABLE, RENAME TABLE,
                      TRUNCATE TABLE, as consistent snapshot is not isolated
                      from them. Option automatically turns off --lock-tables.
  --dump-date         Put a dump date to the end of the output.
                      (Defaults to on; use --skip-dump-date to disable.)
  --skip-opt          Disable --opt. Disables --add-drop-table, --add-locks,
                      --create-options, --quick, --extended-insert,
                      --lock-tables, --set-charset, and --disable-keys.
  -S, --socket=name   The socket file to use for connection.
  --secure-auth       Refuse client connecting to server if it uses old
                      (pre-4.1.1) protocol.
                      (Defaults to on; use --skip-secure-auth to disable.)
  --ssl               Enable SSL for connection (automatically enabled with
                      other flags).
  --ssl-ca=name       CA file in PEM format (check OpenSSL docs, implies
                      --ssl).
  --ssl-capath=name   CA directory (check OpenSSL docs, implies --ssl).
  --ssl-cert=name     X509 cert in PEM format (implies --ssl).
  --ssl-cipher=name   SSL cipher to use (implies --ssl).
  --ssl-key=name      X509 key in PEM format (implies --ssl).
  --ssl-crl=name      Certificate revocation list (implies --ssl).
  --ssl-crlpath=name  Certificate revocation list path (implies --ssl).
  --ssl-verify-server-cert
                      Verify server's "Common Name" in its cert against
                      hostname used when connecting. This option is disabled by
                      default.
  --ssl-mode=name     SSL connection mode.
  -T, --tab=name      Create tab-separated textfile for each table to given
                      path. (Create .sql and .txt files.) NOTE: This only works
                      if mysqldump is run on the same machine as the mysqld
                      server.
  --tables            Overrides option --databases (-B).
  --triggers          Dump triggers for each dumped table.
                      (Defaults to on; use --skip-triggers to disable.)
  --tz-utc            SET TIME_ZONE='+00:00' at top of dump to allow dumping of
                      TIMESTAMP data when a server has data in different time
                      zones or data is being moved between servers with
                      different time zones.
                      (Defaults to on; use --skip-tz-utc to disable.)
  -u, --user=name     User for login if not current user.
  -v, --verbose       Print info about the various stages.
  -V, --version       Output version information and exit.
  -w, --where=name    Dump only selected records. Quotes are mandatory.
  -X, --xml           Dump a database as well formed XML.
  --plugin-dir=name   Directory for client-side plugins.
  --default-auth=name Default authentication client-side plugin to use.
  --enable-cleartext-plugin
                      Enable/disable the clear text authentication plugin.

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
all-databases                     FALSE
all-tablespaces                   FALSE
no-tablespaces                    FALSE
add-drop-database                 FALSE
add-drop-table                    TRUE
add-drop-trigger                  FALSE
add-locks                         TRUE
allow-keywords                    FALSE
apply-slave-statements            FALSE
bind-address                      (No default value)
character-sets-dir                (No default value)
comments                          TRUE
compatible                        (No default value)
compact                           FALSE
complete-insert                   FALSE
compress                          FALSE
create-options                    TRUE
databases                         FALSE
debug-check                       FALSE
debug-info                        FALSE
default-character-set             utf8
delayed-insert                    FALSE
delete-master-logs                FALSE
disable-keys                      TRUE
dump-slave                        0
events                            FALSE
extended-insert                   TRUE
fields-terminated-by              (No default value)
fields-enclosed-by                (No default value)
fields-optionally-enclosed-by     (No default value)
fields-escaped-by                 (No default value)
flush-logs                        FALSE
flush-privileges                  FALSE
force                             FALSE
hex-blob                          FALSE
host                              (No default value)
include-master-host-port          FALSE
insert-ignore                     FALSE
lines-terminated-by               (No default value)
lock-all-tables                   FALSE
lock-tables                       TRUE
log-error                         (No default value)
master-data                       0
max-allowed-packet                25165824
net-buffer-length                 1046528
no-autocommit                     FALSE
no-create-db                      FALSE
no-create-info                    FALSE
no-data                           FALSE
order-by-primary                  FALSE
port                              0
quick                             TRUE
quote-names                       TRUE
replace                           FALSE
routines                          FALSE
set-charset                       TRUE
single-transaction                FALSE
dump-date                         TRUE
socket                            (No default value)
secure-auth                       TRUE
ssl                               FALSE
ssl-ca                            (No default value)
ssl-capath                        (No default value)
ssl-cert                          (No default value)
ssl-cipher                        (No default value)
ssl-key                           (No default value)
ssl-crl                           (No default value)
ssl-crlpath                       (No default value)
ssl-verify-server-cert            FALSE
tab                               (No default value)
triggers                          TRUE
tz-utc                            TRUE
user                              (No default value)
verbose                           FALSE
where                             (No default value)
plugin-dir                        (No default value)
default-auth                      (No default value)
enable-cleartext-plugin           FALSE
```



### 3. 主从复制配置

使用 docker 可以快速的搭建 mysql 主从复制环境，便于我们快速开发。接下来就让我们看看如何操作。

1. 准备 my.cnf 文件

准备两份 my.cnf 文件，这样做的好处是我们可以在启动 docker 镜像的时候挂载而不用登录到 docker 镜像内部去修改 my.cnf 文件，而且镜像删除配置信息也不会丢失。
master:

```
[mysqld]
server_id = 1
log-bin= mysql-bin
read-only=0

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

slave:

```
[mysqld]
server_id = 2
log-bin= mysql-bin
read-only=1

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

2. 创建镜像

```
docker run --name mysql-master -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -v D:/Dev/mysql/master/data:/var/lib/mysql -v D:/Dev/mysql/master/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

```
docker run --name mysql-slave -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -v D:/Dev/mysql/slave/data:/var/lib/mysql -v D:/Dev/mysql/slave/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

如果本地没有镜像，docker 会先下载。

3. 在 master 上创建主从复制用户

登录到 mysql-master 

```
docker exec -it mysql-master /bin/bash
```

登录到 mysql

```
mysql -uroot -p123456
```

创建用户 slave 用来同步数据

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456';
```

查看 master 信息

```
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 684
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```

File 和 Position 的值需要记录下来，在配置 salve 的时候会用到。

4. 在 slave 上配置同步信息

登录到 mysql-slave 

```
docker exec -it mysql-slave /bin/bash
```

登录 mysql

```
mysql -uroot -p123456
```

设置同步配置

```
change master to master_host='172.17.0.3',master_user='slave',master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=684,master_port=3306;
```

> 这里有几个点需要注意：

1. master_host 是 mysql-master 的 IP ，可用 docker inspect --format='{ { .NetworkSettings.IPAddress } }' mysql-master 查看。
2. master_log_file 和 master_log_pos 要和 mysql-master 的信息一致。
3. master_port docker 镜像之间通信走的是 docker 内部的网络，所以端口是 3306 而不是 3307 。

启动主从同步

```
start slave;
```

查看 slave 状态

```
mysql> show slave status\G;                                                            
*************************** 1. row ***************************                         
               Slave_IO_State: Waiting for master to send event                        
                  Master_Host: 172.17.0.3                   
                  Master_User: slave                         
                  Master_Port: 3306                         
                Connect_Retry: 60                           
              Master_Log_File: mysql-bin.000003             
          Read_Master_Log_Pos: 684                           
               Relay_Log_File: c491b3d4773f-relay-bin.000002 
                Relay_Log_Pos: 320                           
        Relay_Master_Log_File: mysql-bin.000003             
             Slave_IO_Running: Yes                           
            Slave_SQL_Running: Yes                           
              Replicate_Do_DB:                               
          Replicate_Ignore_DB: mysql,sys,information_schema,performance_schema         
           Replicate_Do_Table:                               
       Replicate_Ignore_Table:                               
      Replicate_Wild_Do_Table:                               
  Replicate_Wild_Ignore_Table:                               
                   Last_Errno: 0                             
                   Last_Error:                               
                 Skip_Counter: 0                             
          Exec_Master_Log_Pos: 684                           
              Relay_Log_Space: 534                           
              Until_Condition: None                         
               Until_Log_File:                               
                Until_Log_Pos: 0                             
           Master_SSL_Allowed: No                           
           Master_SSL_CA_File:                               
           Master_SSL_CA_Path:                               
              Master_SSL_Cert:                               
            Master_SSL_Cipher:                               
               Master_SSL_Key:                               
        Seconds_Behind_Master: 0                             
Master_SSL_Verify_Server_Cert: No                           
                Last_IO_Errno: 0                             
                Last_IO_Error:                               
               Last_SQL_Errno: 0                             
               Last_SQL_Error:                               
  Replicate_Ignore_Server_Ids:                               
             Master_Server_Id: 1                             
                  Master_UUID: fd482d37-146c-11ea-9944-0242ac110003                    
             Master_Info_File: /var/lib/mysql/master.info   
                    SQL_Delay: 0                             
          SQL_Remaining_Delay: NULL                         
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates  
           Master_Retry_Count: 86400                         
                  Master_Bind:                               
      Last_IO_Error_Timestamp:                               
     Last_SQL_Error_Timestamp:                               
               Master_SSL_Crl:                               
           Master_SSL_Crlpath:                               
           Retrieved_Gtid_Set:                               
            Executed_Gtid_Set:                               
                Auto_Position: 0                             
         Replicate_Rewrite_DB:                               
                 Channel_Name:                               
           Master_TLS_Version:                               
1 row in set (0.00 sec)                                                                
```

Slave_IO_Running 和 Slave_SQL_Running 都为 YES 则配置成功。

> 如果 Slave_IO_Running 为 connecting 可根据  Last_IO_Error 检查



### 4. 主从切换

可靠性优先策略

1. 判断备库的 seconds_behind_master ，是否小于某值（比如 3 秒），如果不小于则等待，直到小于某值再进行第二步。
2. 把主库设置成 readonly = true 。
3. 判断备库的 seconds_behind_master 是否等于 0 ，如果不等于则等待，直到等于 0 将备库设置成 readonly = false 。
4. 把业务切换到备库。

可用性优先策略

可用性优先就是忽略主备延迟，第一时间将业务切换到备库，这样可能会造成数据不一致。

### 5. 高可用方案

高可用架构分成以下几部分：UI 层，API 层，监控服务层，管理数据库，生产数据库集群。

管理数据库存储数据库集群信息，vip 映射信息，日志信息。数据库集群对外有一个 vip，通过 LVS 进行负载，LVS 主备防止单点故障。

<img src="https://s1.ax1x.com/2020/09/05/wVLx9f.png" alt="20200905215639" border="0">

流程如下：

1. 鉴权。
2. 一个线程从管理数据库读取集群信息到队列中。
3. 一个线程从队列中消费。拿到一组主从节点信息，先对主库进行 read 操作（select test 表），如果读成功，继续测试写。如果读失败，可能是网络抖动，不能断定主库挂了，此时就向从库节点发一条命令，从从库读主库，如果读成功，说明主库没挂，如果读失败，则说明主库挂了，此时再将这组集群的信息发送到故障队列。
4. 测试写，对主库进行写操作（insert into test 表）。如果主库可写，则这组集群状态正常，继续下一组检查。如果写主库失败，也可能是网络抖动，此时就向主库发起一组写操作（每 5 秒写一次，持续 30 秒），如果超过半数成功，则认为状态正常，如果超过半数失败，则认为主库故障，加入到故障队列中。
5. 加入到故障队列后，通知相关人员。从故障队列中取出一组主从信息。查询每个从库的 GTID ，GTID 大的作为新的主库，如果 GTID 相同，则随机选一台。
6. 把从库指向主库信息换成新的，将 vip 切换到新的主库，更新原来的主从信息（管理数据库的元数据）。
7. 如果成功，此次故障切换完成，通知相关人员。如果主从切换失败，记录失败日志，告警人工介入。

<img src="https://s1.ax1x.com/2020/09/05/wVLz38.png" alt="20200905215732" border="0">

此方案的问题：

1. 在健康检查的过程中，如果主库挂了，会有 30 秒的时间不可写，读不影响。
2. 管理数据库是单点，需要定期监控。但没必要对管理数据库进行自动主从切换。原因一是管理数据库压力不大，故障概率小，二是，管理数据库挂了，影响健康检查，但是不影响业务数据库使用，用自动主从切换增加了复杂度。
3. 为什么检查主库读，要从从库读，而检查主库写，是循环写？第一次读失败了，也可以多读几次，按半数来统计。



## Part III. Q/A

#### AUTO_INCREMENT 的原理

InnoDB 引擎，8.0 版本前自增的最大值是在内存中维护的，每次插入如果没有指定主键，则会分配一个最大值插入，并更新内存中的最大值。如果指定了主键，并且比内存值小，则不更新内存值，如果比内存值大，则更新内存值。

自增是不能保证连续的，因为插入是先申请最大 id 在进行 insert ，如果 insert 失败，内存中的值是不会回滚的。如果事务回滚，最大 id 同样不会回滚。以此来获得更好的并发性能。

#### 临时表有什么用，什么时候删除？

临时表用于保存临时数据（中间结果）。

临时表存放在 tmpdir 下，文件名是 #{进程id}_{线程id}.序列号.frm 。

临时表不能 rename 。

临时表是 session 绑定的，连接断开时会删除临时表。

#### group by 怎么优化？

如果没有排序要求，在语句后面加上 order by null 。

尽量让 group by 用上索引。

如果 group by 结果不大，尽量使用内存临时表（调大 tmp_table_size 参数），如果数据量大，直接使用磁盘临时表（sql 语句中加入 SQL_BIG_RESULT）。

#### delete、drop、truncate 区别？

A: 删除表用 drop ，删除表中的全部数据用 truncate ，删除表的部分数据用 delete 。drop 和 truncate 是 DDL ，不能回滚，delete 是 DML ，可以回滚。

#### 数据库的3个范式是什么？

A: 第一范式：关系中的每个属性都不可再分。第二范式：要有主键。第三范式：消除冗余。

#### SQL语言包括哪几类？

A: DDL ，DCL ，DML ，DQL ，TPL 。

#### update 语句的执行过程



#### “N叉树”的N值在MySQL中是可以被人工调整的么？

  1， 通过改变key值来调整
N叉树中非叶子节点存放的是索引信息，索引包含Key和Point指针。Point指针固定为6个字节，假如Key为10个字节，那么单个索引就是16个字节。如果B+树中页大小为16K，那么一个页就可以存储1024个索引，此时N就等于1024。我们通过改变Key的大小，就可以改变N的值
2， 改变页的大小
页越大，一页存放的索引就越多，N就越大。

数据页调整后，如果数据页太小层数会太深，数据页太大，加载到内存的时间和单个数据页查询时间会提高，需要达到平衡才行。  

#### count(*)、count(1)、count(主键)、count(字段) 的区别

count(*) MySQL 进行了特殊优化，遍历每一行只计数但不取值。

count(1) 遍历每一行但不取值，每一行放一个 1 ，server 层判断不为空，然后进行累加。

count(主键) 遍历每一行取出每行的主键，server 层判断不为空，然后进行累加。MySQL 的优化是会选择比较小的索引树来统计（主键索引数存储的数据多，会比二级索引树大）。

count(字段) 如果字段定义为 not null ，遍历每一行，读出字段，server 判断不为空，然后进行累加。如果字段定义为允许 null ，遍历每一行，读出字段，取出字段的值判断是否为 null ，不是 null 再累加。如果字段没有加索引，会统计主键索引树。

#### join 为什么要小表驱动大表？

如果被驱动表能用上索引，会使用 Index Nested-Loop Join 算法。

join 语句执行过程中，驱动表是走全表扫描，而被驱动表是走树搜索。假设驱动表的行数是 N，执行过程就要扫描驱动表 N 行，然后对于每一行，到被驱动表上匹配一次。假设被驱动表的行数是 M。每次在被驱动表查一行数据，要先搜索索引 a，再搜索主键索引。每次搜索一棵树近似复杂度是以 2 为底的 M 的对数，记为 log2M，所以在被驱动表上查一行的时间复杂度是 2*log2M。因此整个执行过程，近似复杂度是 N + N\*2\*log2M 。N 对扫描行数的影响更大，因此应该让小表来做驱动表。

如果被驱动表没有用索引，会使用 Block Nested-Loop Join 算法。

执行过程是将驱动表加载到内存 join_buffer 中，然后扫描驱动表的每一行和 join_buffer 做比较，如果满足条件就加入到结果集。如果驱动表数据量大，不能一次放入 join_buffer ，则会分段放入。如果驱动表行数是 N ，被驱动表行数是 M ，一次放入 join_buffer 中，那么两个表都进行全表扫描，行数为 N + M 。此时谁驱动谁都是一样的。如果是分段放入，次数记为 K ，N 驱动 M 则扫描行数是 N + K\*N\*M ，M 驱动 N 则扫描行数为 M + K\*M\*N ，结果取决于驱动表的行数，所以驱动所以驱动表小一些，扫描行数也会小一些。

> 在 Block Nest-Loop Join 算法中，影响 join 的关键是 join_buffer_size

> 在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表。