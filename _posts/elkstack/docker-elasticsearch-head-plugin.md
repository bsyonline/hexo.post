---
title: Elasticsearch 添加 head 插件
date: 2016-07-15 22:25:09
tags:
 - Docker
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 准备工作

```
mkdir dockerfiles/elasticsearch
touch Dockerfile
wget -O elasticsearch-head-master.zip -c https://codeload.github.com/mobz/elasticsearch-head/zip/master
```
***Dockerfile***
```
FROM elasticsearch
MAINTAINER rolex

ADD elasticsearch-head-master.zip /elasticsearch-head-master.zip

RUN /usr/share/elasticsearch/bin/plugin install file:/elasticsearch-head-master.zip

RUN rm -f /elasticsearch-head-master.zip

EXPOSE 9200 9300

CMD [/usr/share/elasticsearch/bin/elasticsearch]

```
### 2. 创建docker镜像

```
sudo docker build -t elasticsearch-2.3 .
```

### 3. 启动

```
sudo docker run elasticsearch-2.3 --publish=[19200:9200,19300:9300]
```
