---
title: User Guide for Sqoop
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-02 10:16:50
thumbnail:

---



### Sqoop 简介

Sqoop 是一个 Hadoop 和结构化存储系统之间批量传输数据的工具。Sqoop 底层通过 MapReduce 实现数据传输。

Sqoop 有两类操作：

导入 import ：将数据导入到 Hadoop 。

导出 export ：从 Hadoop 中导出到结构化存储系统，比如 MySQL 。

### Sqoop 安装

#### 1. 解压

```
tar -zxf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
mv sqoop-1.4.7.bin__hadoop-2.6.0 sqoop
```

#### 2. 修改配置

```
cd sqoop/conf
mv sqoop-env-template.sh sqoop-env.sh
```

添加

```
export HADOOP_COMMON_HOME=/opt/hadoop-2.10.1
export HADOOP_MAPRED_HOME=/opt/hadoop-2.10.1
export HBASE_HOME=/opt/hbase-2.3.5
export HIVE_HOME=/opt/hive-2.3.8
export ZOOCFGDIR=/opt/zookeeper/conf
```

#### 3. 加驱动

将数据库驱动拷贝到 lib 目录。

#### 4. 配置环境变量

```
export SQOOP_HOME=/opt/sqoop
export PATH=$PATH:$SQOOP_HOME/bin
```

#### 5. 验证

```
# sqoop-version
Warning: /opt/sqoop//../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /opt/sqoop//../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
21/05/02 22:19:54 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
Sqoop 1.4.7
git commit id 2328971411f57f0cb683dfb79d19d4d19d185dd8
Compiled by maugli on Thu Dec 21 15:59:58 STD 2017
```

### Sqoop 示例

#### 1. 列出 MySQL 的数据库

```
sqoop list-databases \
--connect jdbc:mysql://localhost:3306/ \
--username root \
--password 123456
```

#### 2. 列出 MySQL 的数据库中的表

```
sqoop list-tables \
--connect jdbc:mysql://localhost:3306/mysql \
--username root \
--password 123456
```

#### 3. 创建一张和 MySQL 中的 employee 表一样的 hive 表 hive_employee

```
sqoop create-hive-table \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
--hive-table hive_employee
```

> 报错 Could not load org.apache.hadoop.hive.conf.HiveConf 
>
> 在环境变量中设置 
>
> export HADOOP_CLASSPATH=$HADOOP_HOME/lib/*
> export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/*

> 报错 access denied ("javax.management.MBeanTrustPermission" "register")
>
> cp $HIVE_HOME/conf/hive-site.xml $SQOOP_HOME/conf/
>
> vim $JAVA_HOME/jre/lib/security/java.policy 
>
> 添加
>
> grant {
>
> ​	permission javax.management.MBeanTrustPermission "register";
>
> }

指定 hive 的数据库

```
sqoop create-hive-table \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
--hive-database test \
--hive-table hive_employee
```

#### 4. 导入 MySQL 表到 HDFS

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
-m 1
```

默认 HDFS 路径在 /user/root/ 下，不指定 -m 参数默认是 4 。

--target-dir 可以指定目录，--fields-terminated-by 可以指定分隔符。

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
--target-dir /user/root/employee1 \
--fields-terminated-by '\t' \
-m 1
```

--where 可以指定查询条件

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--where "id=1" \
--table employee \
--target-dir /user/root/employee2 \
-m 1
```

--query 可以指定查询 sql

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--target-dir /user/root/employee3 \
--query 'select id,name from employee WHERE $CONDITIONS and name = "John"' \
--split-by id \
--fields-terminated-by '\t' \
-m 1
```

$CONDITIONS 是必须的，没有条件也得有。

导入 MySQL 表到 Hive

```
sqoop import \
--connect jdbc:mysql://hadoop1:3306/test \
--username root \
--password 123456 \
--table employee \
--hive-import \
--target-dir /user/root/employee4 \
-m 1
```

--lines-terminated-by指定行分隔符

--fields-terminated-by和列分隔符

--hive-overwrite 指定覆盖导入

--create-hive-table 指定自动创建 hive 表，hive 中不能有重名表存在

--hive-table 指定表名

--delete-target-dir 指定删除中间结果数据目录

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
--fields-terminated-by "\t" \
--lines-terminated-by "\n" \
--hive-import \
--hive-overwrite \
--create-hive-table \
--delete-target-dir \
--hive-database default \
--hive-table employee
```

--last-value 导入 id > 5 的记录，--incremental append 追加

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test \
--username root \
--password 123456 \
--table employee \
--target-dir /user/root/employee5 \
--incremental append \
--check-column id \
--last-value 5 \
-m 1
```

#### 5. 导出 HDFS 到 MySQL 

```
sqoop export \
--connect jdbc:mysql://hadoop1:3306/sqoopdb  \
--username root \
--password 123456 \
--table sqoop_employee \
--export-dir /user/root/employee \
--fields-terminated-by ','
```

MySQL 中的数据库和表要自己创建。

导入 MySQL 数据到 HBase

```
sqoop import \
--connect jdbc:mysql://hadoop1:3306/test \
--username root \
--password 123456 \
--table employee \
--columns "id,name" \
--column-family "info" \
--hbase-row-key "id" \
--hbase-table "employee"
```

--hbase-create-table 支持 HBase1.0.1 之前的版本的自动创建 HBase 表的功能

#### 6. 导出 HBase 数据到 MySQL

1. 将 Hbase 数据，转成 HDFS 文件，然后再由 sqoop 导入。
2. 直接使用 HBase 的 Java API 读取表数据，直接向 MySQL 导入，不需要使用 sqoop 。