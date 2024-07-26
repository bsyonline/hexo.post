---
title: HTTP 上传文件原理
date: 2015-08-30 15:49:49
tags:
 - Interview
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


1. 通过 request.getInputStream() 来得到上传的整个 post 实体的流
2. 用 request.getHeader("Content-Type") 来取得实体内容的分界字符串
3. 然后根据 http 协议，分析取得的上传的实体流
4. 把文件部分给筛出来在服务器端保存到磁盘文件中
