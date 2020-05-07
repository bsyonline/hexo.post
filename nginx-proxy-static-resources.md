---
title: Nginx 代理静态网页
date: 2017-10-17 14:16:13
tags:
 - Nginx
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



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