---
title: shell:autobak.sh
date: 2016-07-16 15:53:17
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```
#!/bin/sh
# backup

DATE=$(date +%F)

# 判断目录是否存在并且有执行权限
if [ ! -x "/test/backup" ]
then
        mkdir /test/backup
fi

# 判断文件是否存在
if [ ! -f "/test/backup/$1.error" ]
then
        touch "/test/backup/$1.error"
fi

/bin/tar -cf /test/backup/$1.$DATE.tar $1 > /dev/null 2>> /test/backup/$1.error
/bin/gzip /test/backup/$1.$DATE.tar

if [ $? -eq 0 ]
then
        echo "$1 backup successfully"
else
        echo "ERROR: $1 $DATE backup" >> /test/backup/$1.error
fi
```
