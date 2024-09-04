---
title: Build image base alpine and Java21
date: 2015-07-16 15:53:32
tags: 
category:
  - Linux
thumbnail: 
author: bsyonline
lede: 没有摘要
---

Dockerfile

```Dockerfile
FROM alpine:latest

ENV LANG=zh_CN.UTF-8
RUN apk add --no-cache ttf-dejavu fontconfig tzdata 
RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN apk update && apk add --no-cache openjdk21
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone

ENV JAVA_HOME=/usr/lib/jvm/default-jvm
```

build

```sh
docker build -t registry.k8s.com/library/alpine:java-21 . 

docker push registry.k8s.com/library/alpine:java-21
```

