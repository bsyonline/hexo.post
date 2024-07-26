---
title: Linux ftp 问题汇总
date: 2014-07-16 15:49:44
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

####虚拟机ftp配置

今天在 vmware 上装 contos，系统装完，可以ping通，但是ftp不通，错误消息如下：
>状态:      尝试连接“ECONNREFUSED - Connection refused by server”失败

google说可能是端口没开或占用

所以检查一下

	#netstat -na --ip

21端口果然没有，打开ftp

	# service vsftpd start

再连ftp，错误消息如下：

>命令:      USER root  
>响应:      530 Permission denied.

用的root用户，vsftp默认关闭root用户，所以打开root用户
修改文件 **`/etc/vsftpd.ftpusers`** 和 **`/etc/vsftpd.user_list`**


>响应: 553 Could not create file.  
>错误: 严重文件传输错误

执行命令

	setsebool allow_ftpd_full_access 1
	setsebool allow_ftpd_use_cifs 1
	setsebool allow_ftpd_use_nfs 1
	setsebool ftp_home_dir 1
	setsebool httpd_enable_ftp_server 1
	setsebool tftp_anon_write 1
	service vsftpd restart

####vsftp上传文件限制
>553 Could not create file.

	[root@node3 vsftpd]# /usr/sbin/setsebool -P ftp_home_dir 1
	[root@node3 vsftpd]# setsebool allow_ftpd_full_access 1
	[root@node3 vsftpd]# setsebool allow_ftpd_use_cifs 1
	[root@node3 vsftpd]# setsebool allow_ftpd_use_nfs 1
	[root@node3 vsftpd]# setsebool -P ftp_home_dir 1
	[root@node3 vsftpd]# setsebool httpd_enable_ftp_server 1
	[root@node3 vsftpd]# setsebool tftp_anon_write 1
	[root@node3 vsftpd]# service vsftpd restart
