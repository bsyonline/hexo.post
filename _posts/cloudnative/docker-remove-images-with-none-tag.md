---
title: 'Docker : 删除所有为 none 的镜像'
date: 2016-10-12 13:29:34
tags:
 - Docker
category: 
 - Docker
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

在使用 Docker 时，如果创建镜像出错，会产生 REPOSITORY 和 TAG 为空的镜像，要删除这些镜像，可以使用
```shell
sudo docker rmi image_id
```
如果这样的镜像有很多，删除就有点麻烦。我们可以利用一个小技巧来批量删除：
```shell
sudo docker images | grep none | awk '{print $3}' | xargs sudo docker rmi -f
```
