---
title: Gulp 环境安装
date: 2016-07-20 11:13:51
tags:
 - Gulp
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

gulp是一个前端代码构建工具。安装gulp需要node.js环境，node.js环境安装参考 [node.js 环境安装](../../../../2016/07/16/node.js 环境安装/) 。
### 全局安装gulp命令行工具
```
npm install -g gulp
or
npm install --global gulp
```
### 安装开发环境
切换到项目目录安装开发环境。
```
sudo npm install -save-dev gulp
```
如果网速慢，可以使用淘宝镜像。
```
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo cnpm install
```
### 创建gulp文件
在项目根目录下创建gulpfile.js
```
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```
### 运行
安装完成执行命令启动gulp。
```
gulp
```
