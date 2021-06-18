---
title: Learning Guide for Hive
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-05 23:32:05
thumbnail:
---



### 简介



### 安装

[Installation Guide for Hive](../../../../2021/04/14/installation-guide-for-hive/)

### 常用命令

列数据库

```
show databases;
```

创建数据库

```
create database test;
```

切换数据库

```
use test;
```

查看当前使用的数据库

```
select current_database();
```

创建内部表

```
create table student (id int, name string) row format delimited fields terminated by ',';
```

创建外部表

```
create external table student (id int, name string) row format delimited fields terminated by ',';
```

创建分区表

```
create table student2 (id int, name string) partitioned by (city string comment "partitioned field") row format delimited fields terminated by ",";
```

创建多个分区的表

```
create table student3 (id int, name string) partitioned by (city string comment "partitioned field", dt string) row format delimited fields terminated by ",";
```

利用查询结果创建表

```
create table dpt_count as select department, count(*) as total from student group by department;
```

加载 hdfs 数据

```
load data inpath "/user/root/employee2" into table student;
```

加载本地数据

```
load data local inpath "/home/root/employee.txt" into table student;
```

加载分区数据

```
load data local inpath "/home/root/student.txt" into table student2 partition(city="beijing");
```

```
alter table student2 add partition (city="shanghai");
load data local inpath "/home/root/student.txt" into table student2 partition(city="shanghai");
```

```
load data local inpath "/home/root/student.txt" into table student3 partition(city="beijing", dt='2012-12-12'); 
```

增加分区

```
alter table student3 add partition(city="beijing", dt='2012-12-14') partition (city="beijing" , dt='2012-12-13');
```

查看分区

```
show partitions student3;
```

查看表结构

```
desc student;
```

查看表结构（详细）

```
desc formatted student;
```

添加字段

```
alter table student add columns (city string, dt string);
```

修改分区的数据目录

```
alter table student3 partition(city="beijing") set location "hdfs://hadoop02:9000/stu_beijing";
```



### 函数

#### 打包

```
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>2.3.8</version>
</dependency>
```

```
public class ToLowerCase extends UDF {
    public Text evaluate(Text input) {
        if (input == null)
            return null;
        return new Text(input.toString().toLowerCase());
    }
}
```

打包 hive-1.0-SNAPSHOT.jar 。

#### 临时函数

##### 加载

上传 jar 到服务器，并在 hive 中加载。

```
hive> add jar /home/data/hive-1.0-SNAPSHOT.jar;
```

##### 创建函数

临时函数退出失效。

```
hive> create temporary function tolowercase as 'com.rolex.hadoop.hive.udf.ToLowerCase';
```

##### 使用

```
hive> select tolowercase('ABC');
```

#### 永久函数

##### 加载

上传 jar 到 HDFS

##### 创建函数

```
create function tolowercase as 'com.rolex.hadoop.hive.udf.ToLowerCase' using jar 'hdfs://hadoop-cluster/user/jar/hive-1.0-SNAPSHOT.jar';
```

##### 使用

```

```

