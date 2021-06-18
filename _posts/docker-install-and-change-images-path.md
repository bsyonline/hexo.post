---
title: Docker 安装及修改 image 存储路径
date: 2017-07-15 22:25:09
tags:
 - Docker
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

###安装 docker
```
sudo yum install -y docker-engine-1.12.6-1.el7.centos.x86_64
sudo systemctl start docker
sudo docker run hello-world
sudo groupadd docker
sudo usermod -aG docker $USER
docker run hello-world
sudo systemctl enable docker
```
###修改 image 存储路径
```
sudo systemctl stop docker
cd /var/lib
sudo cp -rf docker docker.bak
sudo cp -rf docker /u01/
sudo rm -rf docker 
sudo ln -s /u01/docker docker
sudo systemctl start docker
```
