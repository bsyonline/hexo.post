---
title: 一个shell脚本示例
date: 2016-07-16 16:00:19
tags:
 - Shell
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

一个通过shell执行java程序的示例。

```shell
#!/usr/bin/env bash

#``倒引号中是需要执行命令
#找到文件名字，/opt/demo/current/bin/start.sh
SCRIPT_PATH=`readlink -f "$0"`
#可用于定位路径，不用写绝对路径，增加可移植性
cd `dirname ${SCRIPT_PATH}`
#定位到/opt/demo/current/
cd ..
#创建目录
mkdir -p logs

PID_FILE=logs/web.pid
PID=""
#如果文件存在，则读文件
if [ -f $PID_FILE ]
    then
    PID=`cat $PID_FILE`
fi

#如果pid不为空，使用ps查进程
if [ ! "$PID" = "" ]
    then
    #查到任何信息都丢弃，如果查到信息返回0，输出提示消息，没查到返回1
    if ( /bin/ps -p $PID >/dev/null 2>&1 )
        then
        echo "already started, pid = $PID"
        exit 1
    fi
fi

CP="etc"
#将target/classes加到classpath
if [ -d target/classes ]
    then
    CP=$CP:target/classes
fi

#将lib目录下的所有jar包加到classpath
for jar in `/bin/ls -1 lib/*.jar`
do
    CP=$CP:$jar
done

#执行java命令,$@表示所有的参数列表
java -cp $CP com.daas.cfg.Main $@ &

#记录最后运行的后台Process的PID
PID=$!
#记录最后运行的命令的结束代码（返回值）
CODE=$?
#0 表示成功
if [ "$CODE" = "0" ]
    then
    #将pid写到文件
    echo $PID >$PID_FILE
    echo "started, PID=$PID"
fi
```
