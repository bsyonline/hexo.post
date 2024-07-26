---
title: Elasticsearch-1.4.1 集群环境搭建
date: 2016-07-14 19:20:12
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---

CentOS 6.4，机器 node3（10.20.20.203）,node4（10.20.20.204）

### 1. 安装jdk

已安装跳过。

### 2. 下载

[https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.4.1.tar.gz](https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.4.1.tar.gz "elasticsearch-1.4.1")

### 3. 解压
```
tar -zxf elasticsearch-1.4.1.tar.gz
```
### 4. 配置

分别修改2台机器配置

node3
```
cluster.name: elasticsearch
node.name: node3
network.host: 10.20.20.203  
discovery.zen.ping.unicast.hosts: ["10.20.20.203"]
```
node4
```
cluster.name: elasticsearch
node.name: node4
network.host: 10.20.20.204  
discovery.zen.ping.unicast.hosts: ["10.20.20.204"]
```
### 5. 安装插件
#### 5.1 下载
```
wget https://github.com/mobz/elasticsearch-head.git
```
#### 5.2 安装

将插件elasticsearch-head-master.zip拷贝到plugin,解压缩
```
unzip elasticsearch-head-master.zip
mv elasticsearch-head-master head
```
2台机器执行相同操作

### 6. 启动
```
./bin/elasticsearch &
```
