---
title: Hive-0.9.0 安装
date: 2015-11-12 15:49:47
tags:
 - Hive
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


#### **1. 解压缩**
```
[rolex@node2 hive]$ pwd  
/usr/hive
[rolex@node2 hive]$ tar -zxf hive-0.9.0.tar.gz
```
#### **2. 配置环境变量**
```
[rolex@node2 hive]$ sudo echo "HIVE_HOME=/usr/hive/hive-0.9.0">/etc/profile.d/hive.sh
[rolex@node2 hive]$ sudo echo 'PATH=$PATH:$HIVE_HOME/bin'>>/etc/profile.d/hive.sh
[rolex@node2 hive]$ sudo echo "export HIVE_HOME PATH">>/etc/profile.d/hive.sh
[rolex@node2 hive]$ . /etc/profile
```
#### **3. 配置文件**
```
[rolex@node2 conf]$ pwd
/usr/hive/hive-0.9.0/conf
[rolex@node2 conf]$ cp hive-default.xml.template hive-default.xml
[rolex@node2 conf]$ cp hive-env.sh.template hive-env.sh
[rolex@node2 conf]$ cp hive-log4j.properties.template hive-log4j.properties
[rolex@node2 conf]$ cp hive-exec-log4j.properties.template hive-exec-log4j.properties
[rolex@node2 conf]$ cp hive-default.xml.template hive-site.xml
```
#### **4. 启动**
```
[rolex@node2 bin]$ pwd
/usr/hive/hive-0.9.0/bin
[rolex@node2 bin]$ ./hive
Logging initialized using configuration in file:/usr/hive/hive-0.9.0/conf/hive-log4j.properties
Hive history file=/tmp/rolex/hive_job_log_rolex_201511061336_165277921.txt
hive>
```
#### **5. 测试**
```
hive> show databases;
OK
default
Time taken: 0.101 seconds
hive> CREATE TABLE x (a INT);
OK
Time taken: 0.702 seconds
hive> SELECT * FROM x;
OK
Time taken: 0.283 seconds
hive> DROP TABLE x;
OK
Time taken: 1.059 seconds
hive> exit;
```


>Tips：在配置 hadoop-2.6.2 和 hive-1.2.2 启动 hive 时遇到 2 个小问题：
1.  Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
把 ```./conf/hive-site.xml``` 中的 ${system:java.io.tmpdir} 替换成具体的路径。
2. Found class jline.Terminal, but interface was expected
jar 包版本问题，把 ```./lib/jline-2.12.jar``` 拷贝到 ```hadoop-2.6.2/share/hadoop/yarn/lib/``` ，删除原来的 ```jline-0.9.94.jar```，重启 hadoop。

