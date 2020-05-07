---
title: CentOS 6.5 安装JDK
date: 2015-07-16 15:53:32
tags:
 - CentOS
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

### 1. 下载
#### 1.1 安装wget
```shell
yum install -y wget
```
#### 1.2 下载JDK
```shell
wget "http://download.oracle.com/otn-pub/java/jdk/8u20-b26/jdk-8u20-linux-x64.tar.gz"
```
### 2. 解压
```shell
tar -zxf jdk-8u20-linux-x64.tar.gz
```
### 3. 配置环境变量
```shell
echo "JAVA_HOME=/usr/java/jdk1.8.0_20" > /etc/profile.d/java.sh  
echo '\$JAVA_HOME/bin:$PATH' >> /etc/profile.d/java.sh  
echo 'export JAVA_HOME PATH' >> /etc/profile.d/java.sh
```
