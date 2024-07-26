---
title: Ubuntu14.04 + ssh 镜像
date: 2016-07-15 22:25:09
tags:
 - Docker
 - Ubuntu
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 准备工作
```
mkdir ubuntu
cd ubuntu
touch Dockerfile run.sh sources.list authorized_keys
```
***Dockerfile***
```
FROM ubuntu:14.04

MAINTAINER rolex

RUN rm /etc/apt/sources.list

ADD sources.list /etc/apt/sources.list

RUN apt-get update



RUN sudo apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh

RUN sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd

ADD authorized_keys /root/.ssh/authorized_keys
ADD run.sh /run.sh
RUN chmod 755 /run.sh

RUN groupadd dockerone
RUN useradd -g dockerone dockerone
RUN echo "dockerone:dockerone" | chpasswd

RUN echo "root:dockerone" | chpasswd

EXPOSE 22

CMD ["/run.sh"]

```
***run.sh***
```
#!/bin/bash

/usr/sbin/sshd -D

```
***sources.list***
```
deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse

```
***authorized_keys***

```
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub > authorized_keys
```
### 2. 创建镜像

```
sudo docker build -t ubuntu_14.04 .
```
### 3. 测试

```
sudo docker run -d -p 10122:22 ubuntu_14.04

ssh dockerone@localhost -p 10122
```
