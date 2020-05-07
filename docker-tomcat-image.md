---
title: 创建 tomcat 镜像
date: 2016-07-15 22:25:09
tags:
 - Docker
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 准备工作

1.1 创建制作 docker 镜像的工作目录和文件

```
mkdir dorckerfiles
cd dockerfiles
touch Dockerfile run.sh
```
***Dockerfile***
```
FROM ubuntu_14.04

MAINTAINER rolex

ENV DEBIAN_FRONTEND noninteractive

RUN echo "Asia/Beijing" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata

RUN apt-get install -yq --no-install-recommends wget pwgen ca-certificates && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV CATALINA_HOME /tomcat
ENV JAVA_HOME /jdk

ADD apache-tomcat-8.0.35 /tomcat
ADD jdk1.8.0_74 /jdk

#ADD create_tomcat_admin_user.sh /create_tomcat_admin_user.sh
ADD run.sh /run.sh

RUN chmod +x /*.sh
RUN chmod +x /tomcat/bin/*.sh

EXPOSE 8080

CMD ["/run.sh"]

```
***run.sh***
```
#!/bin/bash

/user/sbin/sshd -D &
exec ${CATALINA_HOME}/bin/catalina.sh run

```

1.2 下载 tomcat 和 jdk

### 2. 制作镜像

```
sudo docker build -t tomcat8_on_ubuntu .
```

### 3. 运行镜像

```
sudo docker run -p 32700:8080 tomcat8_on_ubuntu
```
