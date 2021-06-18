---
title: Windows 安装 compass and scss
date: 2016-07-15 22:25:09
tags:
 - Compass
 - Scss
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
### 1. 安装ruby

compass 是 ruby 的插件，需要先安装 ruby 环境。
1. 下载ruby

[http://rubyinstaller.org/](http://rubyinstaller.org/)

2. 添加环境变量
   安装完成后添加环境变量
```
RUBY_HOME=D:\Ruby23-x64
path=$path;%RUBY_HOME%\bin
```
执行ruby -v 显示版本号则安装成功。

### 2. 添加证书

1. 下载证书

从[http://curl.haxx.se/ca/cacert.pem](http://curl.haxx.se/ca/cacert.pem)下载证书，保存到ruby安装目录。

2. 添加环境变量
```
SSL_CERT_FILE=D:\Ruby23-x64\cacert.pem
```
### 3. 修改源
```
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem sources -l
```
### 4. 安装compass
```
gem install compass
```
sass -v显示版本号则安装成功。正常情况compass和scss都会安装。
