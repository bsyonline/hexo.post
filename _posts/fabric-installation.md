---
title: Fabric安装
date: 2016-05-17 15:45:09
tags:
 - Fabric
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 安装依赖包

```shell
sudo rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm  
sudo yum install -y python-devel  
sudo yum install -y gcc  
sudo yum install -y wget  
sudo yum install -y python-pip  
```
### 2. 安装fabric
```shell
sudo pip install fabric
```
### 3. 验证

```shell
python -c "from fabric.api import * ; print env.version"
```
输出版本信息则安装成功。
