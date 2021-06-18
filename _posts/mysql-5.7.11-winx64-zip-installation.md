---
title: MySQL-5.7.11-winx64 zip版安装
date: 2016-06-22 22:25:09
tags:
 - MySQL
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

OS : Windows 10 x64

### 解压

解压路径为 **%MYSQL_HOME%**

### 配置环境变量

add **%MYSQL_HOME%\bin** to path

### 修改my.ini

拷贝my-default.ini为my.ini
```
basedir = D:\Program Files\MySQL\mysql-5.7.11-winx64
datadir =  D:\Program Files\MySQL\mysql-5.7.11-winx64\data
port = 3306
```
### 初始化
```
mysqld --initialize --user=mysql --console
```
记录末尾的初始密码

### 安装服务
```
mysqld --install MySQL
```
### 启动
```
net start mysql
```
### 登录
```
mysql -u root -p
```
### 修改密码
```
set password for root@localhost = password('123456');
```
