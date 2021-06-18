---
title: Sqlplus 执行文件
date: 2016-02-13 16:00:30
tags:
 - Oracle
category: 
 - Databases
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

使用客户端工具导入数据量大时会卡死，所以改用命令行执行。

1. 连接数据库
```
sqlplus user/password@192.168.11.15:1521/orcl
```
2. 执行文件
```
SQL> @D:\a.sql
```
