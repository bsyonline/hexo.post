---
title: 企业信息变更监控系统
tags:
  - 
category:
  - 
author: bsyonline
lede: 没有摘要
date: 2021-05-30 18:31:35
thumbnail:
---



### 1. 架构 

分层

1. 分发层 nginx
2. 应用层 nginx
3. 分布式缓存层 redis
4. 应用服务层+本地缓存
5. 数据层 mysql

### 2. 缓存更新流程



### 3. 缓存数据库双写一致性方案



### 4. 热点数据更新方案





redis cluster 3m3s，机器配置 4c  32g 



按1汉字=2Byte计算，一个response500字=1KB

1KB/request * 10w = 100M

mysql 主从 

服务 3 节点

企业+个体量 2e 5000w的baseinfo大小60G

总baseInfo 150G



订阅数3000w 30G的基本信息



30G 都放在redis 每个master给15G，这样可以保证3000w企业的信息查询都能走缓存



PV=5000W

QPS 