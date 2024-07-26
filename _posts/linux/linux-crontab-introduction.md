---
title: Linux 定时任务 crontab 命令详解
date: 2016-09-13 16:03:32
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

安装
```
yum install crontabs
```

查看当前用户的定时任务
```
crontab -l
```
启动定时任务
```
5 3 * * * /u01/backup.sh
```

crontab 时间格式
![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/crontab.png)

举例：
```
每分钟执行一次            
*  *  *  *  *

每隔一小时执行一次        
00  *  *  *  *
or
* */1 * * *  (/表示频率)

每小时的 15 和 45 分各执行一次
15,45 * * * * （,表示并列）

在每天上午 8 - 11 时中间每小时 15 ，45分各执行一次
15,45 8-11 * * * command （-表示范围）

每个星期一的上午 8 点到 11 点的第 3 和第 15 分钟执行
3,15 8-11 * * 1 command

每隔两天的上午 8 点到 11 点的第 3 和第 15 分钟执行
3,15 8-11 */2 * * command
```
