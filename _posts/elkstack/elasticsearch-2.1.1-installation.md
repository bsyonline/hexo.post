---
title: Elasticsearch-2.2.0 安装
date: 2016-07-14 19:20:12
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---


### 1. 下载

[https://download.elasticsearch.org/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.2.0/elasticsearch-2.2.0.tar.gz](https://download.elasticsearch.org/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.2.0/elasticsearch-2.2.0.tar.gz "elasticsearch-2.2.0")

### 2. 安装
```
tar -zxf elasticsearch-2.2.0.tar.gz
```

### 3. 配置
>vi ./config/elasticsearch.yml

	cluster.name: elsticsearch  
	node.name: chinadaas01
	network.host: 192.168.200.163
	discovery.zen.ping.unicast.hosts: ["192.168.200.163","192.168.200.164","192.168.200.165","192.168.200.166"]

### 4. 安装插件
#### 4.1 下载

>wget https://github.com/mobz/elasticsearch-head.git
#### 4.2 安装
elasticsearch提供了两种安装方式，在线安装总失败，所以选离线安装。
>./bin/plugin install file:./elasticsearch-head-master.zip

### 5. 集群搭建

其他机器执行相同配置，ip和node.name改成对应机器的ip。


### 6. 启动

>./bin/elasticsearch &

浏览器中使用 `http://192.168.200.163:9200/_plugin/head/` 可看到界面
