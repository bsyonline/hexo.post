---
title: MySQL 安装服务及 Windows CMD 中使用 MySQL 命令
date: 2017-11-01 15:20:53
tags:
 - MySQL
category: 
 - Databases
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



Windows 上使用 zip 版的 MySQL 配置服务方法：

#### 配置

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

#### 安装服务

使用管理员权限打开 CMD ，输入命令：

```
mysqld --install MySQL --defaults-file=D:\Dev\MySQL\mysql-5.5.58\my.ini
```

安装完成。

将 bin 目录加入环境变量即可在 CMD 中使用 MySQL 命令。