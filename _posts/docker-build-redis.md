---
title: 使用 docker + redis 学习 redis-cli
date: 2016-07-15 22:25:09
tags:
 - Redis
 - Docker
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

有时我们不想在自己机器上安装redis环境又想快速开始redis-cli实践，如果安装了docker环境，那么redis的环境将非常容易搭建。
### 1. 下载redis的docker镜像
```shell
sudo docker pull redis
```

### 2. 启动redis server
```shell
sudo docker run -d --name redisone redis redis-server
```

### 3. 启动redis cli
```shell
sudo docker run -it --link redisone redis redic-cli -h redisone -p 6379
```
