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



简介



安装



函数

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

打包 hive-1.0-SNAPSHOT.jar

上传到服务器

加载

```
hive> add jar /home/data/hive-1.0-SNAPSHOT.jar;
```

创建函数

```
hive> create temporary function tolowercase as 'com.rolex.hadoop.hive.udf.ToLowerCase';
```

使用

```
hive> select tolowercase('ABC');
```

再次进入 Hive ，临时函数无法使用。

创建永久函数

1. 上传 jar 到 HDFS

2. 创建函数

   ```
   create function tolowercase as 'com.rolex.hadoop.hive.udf.ToLowerCase' using jar 'hdfs://hadoop-cluster/user/jar/hive-1.0-SNAPSHOT.jar';
   ```

   

```

```

