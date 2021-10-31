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

### nginx 安装

下载

```shell
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
wget https://nginx.org/download/nginx-1.10.3.tar.gz
```

安装编译工具和库

CentOS 

```shell
yum group install Development Tools
yum install -y openssl openssl-devel zlib zlib-devel pcre-devel
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

安装 Nginx

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

配置

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



### docker 安装 nginx

官方提供了 Nginx 的 docker 镜像，就不用自己打镜像了。首先下载 nginx 镜像 [https://hub.docker.com/_/nginx/](https://hub.docker.com/_/nginx/) 。

使用 nginx 的镜像比较简单，但是在挂载配置时出了一些问题，一个在物理机上正常使用的配置文件，有可能在 docker 中报错，所以要注意挂载的配置文件和路径是否正确。

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

```
docker run -p 80:80 -p 38002:38002 --name mynginx -v /home/rolex/work/conf.d/:/etc/nginx/conf.d/ -v /home/rolex/work/logs/:/etc/nginx/logs/ -it nginx:1.10.3
```



### 代理静态资源

Nginx 默认是支持静态资源的，配置如下：

```
server {
  listen      8202 ;

  charset utf-8;

  server_name 192.168.201.74;
  root /home/ops/developer.bidata.com.cn;
  index index.html;
  location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    root /home/ops/developer.bidata.com.cn;
  }
}
```

生效后，访问 192.168.201.74:8202/index.html 就会去访问 /home/ops/developer.bidata.com.cn 下的 index.html 。

### Openresty

下载 openresty 

```
cd /opt/servers
wget https://openresty.org/download/ngx_openresty-1.7.7.2.tar.gz
tar -zxf ngx_openresty-1.7.7.2.tar.gz
```

安装 luajit

```
cd /opt/servers/ngx_openresty-1.7.7.2/bundle/LuaJIT-2.1-20150120
make clean && make && make install
ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit
```

安装其他 bundle 

```
cd /opt/servers/ngx_openresty-1.7.7.2/bundle
wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz  
tar -xvf 2.3.tar.gz 

wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz  
tar -xvf v0.3.0.tar.gz 
```

安装 openresty

```
cd /opt/servers/ngx_openresty-1.7.7.2
./configure --prefix=/opt/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2
make && make install
```

查看安装的nginx版本

```
/opt/servers/nginx/sbin/nginx -V
```

启动 nginx

```
/opt/servers/nginx/sbin/nginx
```

lua helloworld

编辑 /opt/servers/nginx/conf/nginx.conf ，在 http 部分添加

```
lua_package_path "/opt/servers/lualib/?.lua;;";
lua_package_cpath "/opt/servers/lualib/?.so;;";
```

在 /opt/servers/nginx/conf 下，创建一个 lua.conf 

```
server {  
    listen       80;  
    server_name  _;
    location /lua {  
    	default_type 'text/html';  
    	content_by_lua_file conf/lua/helloworld.lua;
	} 
}
```

在 conf 下创建一个 lua 目录，将 lua 脚本都放在这个目录下。

```
ngx.say("hello world");
```

在 nginx.conf 中 http 部分引入 lua.conf 

```
http {
    ...
    include lua.conf;
    ...
}
```

验证修改是否正确

```
/opt/servers/nginx/sbin/nginx -t
```

重新加载 nginx

```
/opt/servers/nginx/sbin/nginx -s reload
```

请求 /lua 地址

```
curl -XGET hadoop1/lua
hello world
```

