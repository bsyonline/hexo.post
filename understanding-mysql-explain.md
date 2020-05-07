---
title: Understanding MySQL Explain
tags:
  - Interview
category:
  - MySQL
author: bsyonline
lede: 没有摘要
date: 2020-04-01 20:25:15
thumbnail:
---

Explain 会展示 MySQL 优化器关于语句执行计划的信息，也就是说会解释 MySQL 将如何处理语句。
我们先来试一下。
```
mysql> explain select * from t_user where name='tom';
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t_user | NULL       | ref  | i_name        | i_name | 202     | const |    1 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
```

Explain 结果显示了 12 列（黑体是我们需要重点关注的列）。
<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">列</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">说明</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">id</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">序号</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">**select_type**</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">select 的类型</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">table</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">表名</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">partitions</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">匹配的分区</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">**type**</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">查找的类型</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">possible_keys</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">可能选择的索引</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">key</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">实际选择的索引</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">key_len</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">实际使用的索引的长度</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">**ref**</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">与索引比较的列</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">rows</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">读取的行数</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">filtered</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">按条件过滤后的行数占比</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">**Extra**</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">附加信息</td></tr></table><br>


##### **select_type**
select_type 有以下一些值：
1. SIMPLE：简单 SELECT（不使用 UNION 或子查询）。
```
mysql> explain select * from t_user;
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | t_user | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  100 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
```
2. PRIMARY：最外层 SELECT 。
```
mysql> explain select t1.* from t_user t1 where t1.id =(select id from t_user t2 where t2.name='user1');
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | t1    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | t2    | NULL       | ref   | i_name        | i_name  | 202     | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
```
3. UNION：UNION 中第二个及以后的 SELECT 语句。
```
mysql> explain select * from t_user t1 where t1.id=1 union select * from t_user t2 where t2.id=2;
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
| id | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
|  1 | PRIMARY      | t1         | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL            |
|  2 | UNION        | t2         | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL            |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
```
union 的结果 UNION RESULT 放在临时表 <union1,2> 中，所以 id 为 NULL 。union 要对结果去重，所以需要临时表，union all 不需要去重，所以不需要临时表。
```
mysql> explain select * from t_user t1 where t1.id=1 union all select * from t_user t2 where t2.id=2;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | t1    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  2 | UNION       | t2    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```
4. DEPENDENT UNION：UNION 中的第二个及以后的 SELECT 语句，依赖外层查询。
```
mysql> explain select t1.* from t_user t1 where t1.id in (select id from t_user t2 union select id from t_user t3);
+----+--------------------+------------+------------+--------+---------------+---------+---------+------+------+----------+-----------------+
| id | select_type        | table      | partitions | type   | possible_keys | key     | key_len | ref  | rows | filtered | Extra           |
+----+--------------------+------------+------------+--------+---------------+---------+---------+------+------+----------+-----------------+
|  1 | PRIMARY            | t1         | NULL       | ALL    | NULL          | NULL    | NULL    | NULL |  100 |   100.00 | Using where     |
|  2 | DEPENDENT SUBQUERY | t2         | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | func |    1 |   100.00 | Using index     |
|  3 | DEPENDENT UNION    | t3         | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | func |    1 |   100.00 | Using index     |
| NULL | UNION RESULT       | <union2,3> | NULL       | ALL    | NULL          | NULL    | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------------+------------+------------+--------+---------------+---------+---------+------+------+----------+-----------------+
```
5. UNION RESULT：UNION 的结果。
6. SUBQUERY：子查询中第一个 SELECT 语句。
7. DEPENDENT SUBQUERY：子查询中第一个 SELECT 语句，依赖外部查询。
8. DERIVED：derived 表
```
mysql> explain select * from (select id from t_user group by id) A;
+----+-------------+------------+------------+-------+------------------------+---------+---------+------+------+----------+-------------+
| id | select_type | table      | partitions | type  | possible_keys          | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+------------+------------+-------+------------------------+---------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | <derived2> | NULL       | ALL   | NULL                   | NULL    | NULL    | NULL |  100 |   100.00 | NULL        |
|  2 | DERIVED     | t_user     | NULL       | index | PRIMARY,i_name,i_skill | PRIMARY | 4       | NULL |  100 |   100.00 | Using index |
+----+-------------+------------+------------+-------+------------------------+---------+---------+------+------+----------+-------------+
```
9. MATERIALIZED：
10. UNCACHEABLE SUBQUERY：子查询结果无法缓存，必须针对外部查询的每一行重新进行评估。
11. UNCACHEABLE UNION：UNION 中的第二个及以后的 SELETE 语句，并且这个 UNION 属于 UNCACHEABLE SUBQUERY 。

##### **table**
table 对应的是表名，比如 t1 ，或者是临时表的名字。将查询的结果保存到临时表叫做物化。如果是物化的方式，则显示为 ```<derivedN>``` ，表示需要依赖 id 为 N 的查询。当使用 union 时显示 ```<union1,2>``` 。 


##### **type**
1. system：该表只有一行记录。system 是 const 的特例。
2. const：全表最多只有一行匹配。
```
mysql> explain select * from t_user where id=1;
+----+-------------+--------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table  | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t_user | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+--------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
```
3. eq_ref：A 连接 B ，B 的连接字段是主键或唯一索引。
```
mysql> explain select * from t_dept join t_employee on t_dept.dept_id=t_employee.id;
+----+-------------+------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra |
+----+-------------+------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------+
|  1 | SIMPLE      | t_dept     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t_employee | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.t_dept.dept_id |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------+
```
4. ref：A 连接 B ，B 的连接字段是普通索引。
```
mysql> explain select * from t_dept join t_employee on t_dept.dept_id=t_employee.dept_id;
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
| id | select_type | table      | partitions | type | possible_keys | key       | key_len | ref                 | rows | filtered | Extra |
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
|  1 | SIMPLE      | t_dept     | NULL       | ALL  | PRIMARY       | NULL      | NULL    | NULL                |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t_employee | NULL       | ref  | i_dept_id     | i_dept_id | 5       | test.t_dept.dept_id |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
```
5. fulltext：使用 fulltext 索引进行连接。
6. ref_or_null：类似 ref ，但是还会搜索包含 NULL 的行。
```
mysql> explain select * from t_user where skill='aaa' or skill is null;
+----+-------------+--------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
| id | select_type | table  | partitions | type        | possible_keys | key     | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+--------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | t_user | NULL       | ref_or_null | i_skill       | i_skill | 43      | const |    2 |   100.00 | Using index condition |
+----+-------------+--------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
```
7. index_merge：
```
mysql> explain select * from t_user where name='user1' or skill is null;
+----+-------------+--------+------------+-------------+----------------+----------------+---------+------+------+----------+------------------------------------------+
| id | select_type | table  | partitions | type        | possible_keys  | key            | key_len | ref  | rows | filtered | Extra                                    |
+----+-------------+--------+------------+-------------+----------------+----------------+---------+------+------+----------+------------------------------------------+
|  1 | SIMPLE      | t_user | NULL       | index_merge | i_name,i_skill | i_name,i_skill | 202,43  | NULL |    2 |   100.00 | Using union(i_name,i_skill); Using where |
+----+-------------+--------+------------+-------------+----------------+----------------+---------+------+------+----------+------------------------------------------+
```
8. unique_subquery：
9. index_subquery：
10. range：
```
mysql> explain select * from t_user where id>10;
+----+-------------+--------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_user | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |   90 |   100.00 | Using where |
+----+-------------+--------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
```
11. index：全表扫描，查询字段全都有索引。
```
mysql> explain select name from t_user;
+----+-------------+--------+------------+-------+---------------+--------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type  | possible_keys | key    | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+-------+---------------+--------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_user | NULL       | index | NULL          | i_name | 202     | NULL |  100 |   100.00 | Using index |
+----+-------------+--------+------------+-------+---------------+--------+---------+------+------+----------+-------------+
```
12. ALL：全表扫描，查询的字段不全都有索引。
```
mysql> explain select * from t_user;
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | t_user | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  100 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
```

##### **possible_keys**
如果查询没有使用索引，则为 NULL ，可以在 where 中添加一些条件来使用索引。

##### **key_len**
key_len 是实际使用的 key 的长度，tinyint 是 1 字节 ，smallint 是 2 字节，int 是 4 字节，bigint 是 8 字节，timestamp 是 3 字节，datetime 是 8 字节 ，char(n) 是 n 字节，varchar(n) 是 x\*n + 2 字节，x 根据字符集而定。

##### **rows**
rows 是查询计划预估的记录数，不是真正的结果行数。如果没有走索引，是全表扫描的行数，如果走索引，是预估的索引扫描行数。

##### **filtered**
filtered 主要用于表连接。
```
mysql> explain select * from t_employee,t_dept where t_employee.dept_id=t_dept.dept_id;
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
| id | select_type | table      | partitions | type | possible_keys | key       | key_len | ref                 | rows | filtered | Extra |
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
|  1 | SIMPLE      | t_dept     | NULL       | ALL  | PRIMARY       | NULL      | NULL    | NULL                |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t_employee | NULL       | ref  | i_dept_id     | i_dept_id | 5       | test.t_dept.dept_id |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+------+---------------+-----------+---------+---------------------+------+----------+-------+
```
在使用表连接时，如果是 A left join B ，则 A 是驱动表，B 是被驱动表。如果是 A right join B ，则 B 是驱动表，A 是被驱动表。如果是 ```select * from A,B```  ，则小表是驱动表，大表是被驱动表。上边的例子，t_dept 是驱动表，t_employee 是被驱动表，t_employee 的连接行数 = 1 行 (t_dept rows \* t_dept filtered=1 \* 100% = 1) 。

##### Extra
1. NULL：查询的列没有被索引覆盖，必须通过回表进行查询。
```
mysql> explain select * from t_user where name='tom';
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t_user | NULL       | ref  | i_name        | i_name | 202     | const |    1 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
```
1. Using index：只使用索引查询列的信息，这种情况一般都是使用了索引覆盖。
```
mysql> explain select name from t_user where name='tom';
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | t_user | NULL       | ref  | i_name        | i_name | 202     | const |    1 |   100.00 | Using index |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
```
2. Using where：where 条件是索引之一。
```
mysql> explain select name from t_user where name='tom' and age=10;
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | t_user | NULL       | ref  | i_name        | i_name | 202     | const |    1 |    10.00 | Using where |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------------+
```
3. Using index condition：使用了索引下推优化。
4. Using temporary：查询使用了临时表，临时表创建和维护成本很高，所以一般需要考虑优化。
5. Using filesort：filesort 耗时，一般需要考虑优化成索引排序。
6. Using join buffer：连接条件没有使用索引，需要优化。