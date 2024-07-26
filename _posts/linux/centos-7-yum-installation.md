---
title: CentOS 7 设置国内源
date: 2017-04-01 15:05:26
tags:
 - CentOS
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

网易源设置帮助 [http://mirrors.163.com/.help/centos.html](http://mirrors.163.com/.help/centos.html)。

<!-- more -->

### 备份
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

### 下载新源

```
cd /etc/yum.repos.d
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
```

### 生成缓存

```
yum clean all
yum makecache
```