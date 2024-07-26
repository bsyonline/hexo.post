---
title: CentOS 6.5 minimal安装
date: 2015-07-26 15:53:37
tags:
 - CentOS
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

### 1. 安装eth0

**/etc/sysconfig/network-scripts/ifcfg-eth0**
```
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.64
GATEWAY=192.168.1.1
```
### 2. 创建用户和组
```shell
groupadd rolex  
useradd -g rolex rolex  
passwd rolex
```
### 3. 添加sudo权限
```shell
visudo

rolex	ALL=(ALL)	ALL
rolex	ALL=NOPASSWD:	ALL
```

### 4. 安装ftp

参考 [CentOS 6.5 minimal 安装ftp](../../../../2016/07/16/CentOS 6.5 minimal 安装ftp/) 。

### 5. 安装JDK

参考 [CentOS 6.5 安装JDK](../../../../2016/07/16/CentOS 6.5 安装JDK/) 。
