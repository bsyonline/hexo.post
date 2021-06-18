---
title: Ubuntu 16.04 安装 oracle instant client
date: 2016-04-19 15:53:12
tags:
 - Ubuntu
 - Oracle
category: 
 - Databases
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 安装alien

```
sudo apt-get install alien
```

### 2. 下载oracle instant client

oracle-instantclient12.1-basic-12.1.0.2.0-1.x86_64.rpm
oracle-instantclient12.1-devel-12.1.0.2.0-1.x86_64.rpm
oracle-instantclient12.1-sqlplus-12.1.0.2.0-1.x86_64.rpm

### 3. 安装
```
sudo alien -i *.rpm
```
### 4. 配置环境变量

```
echo 'export ORACLE_HOME=/usr/lib/oracle/12.1/client64' > /etc/profile.d/oracle.sh
echo 'export ORACLE_BASE=/usr/lib/oracle/12.1' >> /etc/profile.d/oracle.sh
echo 'export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH' >> /etc/profile.d/oracle.sh
echo 'export NLS_LANG=AMERICAN_AMERICA.AL32UTF8' >> /etc/profile.d/oracle.sh
echo 'export PATH=$ORACLE_HOME/bin:$PATH' >> /etc/profile.d/oracle.sh
source /etc/profile
```
### 5. 解决上下左右键乱码

```
sudo apt-get install rlwrap
echo "alias sqlplus='rlwrap sqlplus'" >> ~/.bashrc
source /etc/profile
```

### 6. 测试
```
sqlplus username/passwd@//hostname:post/sid
```
