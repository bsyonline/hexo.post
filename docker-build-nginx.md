---
title: Docker 搭建 Nginx 环境
date: 2017-04-11 15:13:19
tags:
 - Docker
 - Nginx
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

官方提供了 Nginx 的 docker 镜像，就不用自己打镜像了。首先下载 nginx 镜像 [https://hub.docker.com/_/nginx/](https://hub.docker.com/_/nginx/) 。

使用 nginx 的镜像比较简单，但是在挂载配置时出了一些问题，一个在物理机上正常使用的配置文件，有可能在 docker 中报错，所以要注意挂载的配置文件和路径是否正确。<!--more-->

使用下面的命令查看镜像中的配置文件。

```shell
docker run --rm -it nginx:1.10.3 cat /etc/nginx/nginx.conf
```

可对照修改自己的配置文件。

```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

完整的启动命令：

```shell
docker run -p 80:80 -p 38002:38002 --name mynginx -v /home/rolex/work/conf.d/:/etc/nginx/conf.d/ -v /home/rolex/work/logs/:/etc/nginx/logs/ -it nginx:1.10.3
```
