---
title: CentOS 6.5 minimal安装ftp
date: 2015-07-20 15:53:35
tags:
 - CentOS
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 配置DNS

**/etc/resolv.conf**
```shell
nameserver 8.8.8.8
nameserver 4.4.4.4
```
###2. 安装ftp
```shell
yum install vsftpd
```
###3. 放开home路径权限
```shell
setsebool -P  ftp_home_dir  on
```
###4. 虚拟机ftp配置

系统装完，可以ping通，但是ftp不通，错误消息如下：

状态:      尝试连接“ECONNREFUSED - Connection refused by server”失败

google说可能是端口没开或占用

所以检查一下
```shell
netstat -na --ip
```
21端口果然没有，打开ftp
```shell
service vsftpd start
```
再连ftp，错误消息如下：

命令:      USER root  
响应:      530 Permission denied.

用的root用户，vsftp默认关闭root用户，所以打开root用户
修改文件 **/etc/vsftpd.ftpusers** 和 **/etc/vsftpd.user_list**


响应: 553 Could not create file.  
错误: 严重文件传输错误

执行命令
```shell
setsebool allow_ftpd_full_access 1
setsebool allow_ftpd_use_cifs 1
setsebool allow_ftpd_use_nfs 1
setsebool ftp_home_dir 1
setsebool httpd_enable_ftp_server 1
setsebool tftp_anon_write 1
service vsftpd restart
```
###5. vsftp上传文件限制

553 Could not create file.
```shell
/usr/sbin/setsebool -P ftp_home_dir 1
/usr/sbin/setsebool allow_ftpd_full_access 1
/usr/sbin/setsebool allow_ftpd_use_cifs 1
/usr/sbin/setsebool allow_ftpd_use_nfs 1
/usr/sbin/setsebool -P ftp_home_dir 1
/usr/sbin/setsebool httpd_enable_ftp_server 1
/usr/sbin/setsebool tftp_anon_write 1
/usr/sbin/service vsftpd restart
```
