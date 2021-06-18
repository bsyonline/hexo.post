---
title: Nginx 安装配置
date: 2017-04-01 15:05:51
tags:
 - Nginx
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

Nginx 是一款高性能的 HTTP 和反向代理服务器。

<!-- more -->

安装配置过程如下：

### 下载
```shell
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
wget https://nginx.org/download/nginx-1.10.3.tar.gz
```

### 安装编译工具和库
CentOS 

```shell
yum group install Development Tools
yum install openssl openssl-devel zlib zlib-devel
```
ubuntu

```shell
apt-get install openssl libssl-dev
```

安装 PCRE

```shell
tar -zxf pcre-8.35.tar.gz
cd pcre-8.35
./configure
make && make install
```

### 安装 Nginx

```shell
tar -zxf nginx-1.10.3.tar.gz
mv nginx-1.10.3 /opt/nginx/
cd /opt/nginx/nginx-1.10.3
./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/root/pcre-8.35
make && make install
```

显示版本信息则安装成功。

```shell
/usr/local/webserver/nginx/sbin/nginx -v
```


### 配置

nginx 的配置文件为 /usr/local/webserver/nginx/conf/nginx.conf

```
user root;
worker_processes 8; #设置值和CPU核心数一致
error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
pid /usr/local/webserver/nginx/nginx.pid;
#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;
events
{
  use epoll;
  worker_connections 65535;
}
http
{
  include mime.types;
  default_type application/octet-stream;
  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';
     
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;
     
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 60;
  tcp_nodelay on;
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  gzip on; 
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;
 
  #limit_zone crawler $binary_remote_addr 10m;
  #下面是server虚拟主机的配置
  #server多的时候建议使用 include /opt/nginx/nginx-1.10.3/conf/conf.d/*.conf; 每个server 单独一个配置文件
 server
  {
    listen 80;#监听端口
    server_name localhost;#域名
    index index.html index.htm index.php;
    root /usr/local/webserver/nginx/html;#站点目录
      location ~ .*\.(php|php5)?$
    {
      #fastcgi_pass unix:/tmp/php-cgi.sock;
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
    {
      expires 30d;
  # access_log off;
    }
    location ~ .*\.(js|css)?$
    {
      expires 15d;
   # access_log off;
    }
    access_log off;
  }

server {
    listen 8002 ;
    charset utf-8;
    access_log logs/host.access.log main;
    location /{
        proxy_pass http://cn.bing.com/;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP  $remote_addr;
    }
}
}
```

修改后，检查配置。
```shell
/usr/local/webserver/nginx/sbin/nginx -t
```

启动 nginx

```
/usr/local/webserver/nginx/sbin/nginx
```