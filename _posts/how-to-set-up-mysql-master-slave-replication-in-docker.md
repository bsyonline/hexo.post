---
title: How to Set Up MySQL Master Slave Replication in Docker
date: 2019-12-03 13:39:22
tags:
  - Docker
  - MySQL
category:
  - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



使用 docker 可以快速的搭建 mysql 主从复制环境，便于我们快速开发。接下来就让我们看看如何操作。

#### 1. 准备 my.cnf 文件

准备两份 my.cnf 文件，这样做的好处是我们可以在启动 docker 镜像的时候挂载而不用登录到 docker 镜像内部去修改 my.cnf 文件，而且镜像删除配置信息也不会丢失。
master:

```
[mysqld]
server_id = 1
log-bin= mysql-bin
read-only=0

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

slave:

```
[mysqld]
server_id = 2
log-bin= mysql-bin
read-only=1

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

#### 2. 创建镜像

```
docker run --name mysql-master -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -v D:/Dev/mysql/master/data:/var/lib/mysql -v D:/Dev/mysql/master/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

```
docker run --name mysql-slave -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -v D:/Dev/mysql/slave/data:/var/lib/mysql -v D:/Dev/mysql/slave/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

如果本地没有镜像，docker 会先下载。

#### 3. 在 master 上创建主从复制用户

登录到 mysql-master 

```
docker exec -it mysql-master /bin/bash
```

登录到 mysql

```
mysql -uroot -p123456
```

创建用户 slave 用来同步数据

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456';
```

查看 master 信息

```
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 684
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```

File 和 Position 的值需要记录下来，在配置 salve 的时候会用到。

#### 4. 在 slave 上配置同步信息

登录到 mysql-slave 

```
docker exec -it mysql-slave /bin/bash
```

登录 mysql

```
mysql -uroot -p123456
```

设置同步配置

```
change master to master_host='172.17.0.3',master_user='slave',master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=684,master_port=3306;
```

> 这里有几个点需要注意：
1. master_host 是 mysql-master 的 IP ，可用 docker inspect --format='{ { .NetworkSettings.IPAddress } }' mysql-master 查看。
2. master_log_file 和 master_log_pos 要和 mysql-master 的信息一致。
3. master_port docker 镜像之间通信走的是 docker 内部的网络，所以端口是 3306 而不是 3307 。




启动主从同步

```
start slave;
```

查看 slave 状态

```
mysql> show slave status\G;                                                            
*************************** 1. row ***************************                         
               Slave_IO_State: Waiting for master to send event                        
                  Master_Host: 172.17.0.3                   
                  Master_User: slave                         
                  Master_Port: 3306                         
                Connect_Retry: 60                           
              Master_Log_File: mysql-bin.000003             
          Read_Master_Log_Pos: 684                           
               Relay_Log_File: c491b3d4773f-relay-bin.000002 
                Relay_Log_Pos: 320                           
        Relay_Master_Log_File: mysql-bin.000003             
             Slave_IO_Running: Yes                           
            Slave_SQL_Running: Yes                           
              Replicate_Do_DB:                               
          Replicate_Ignore_DB: mysql,sys,information_schema,performance_schema         
           Replicate_Do_Table:                               
       Replicate_Ignore_Table:                               
      Replicate_Wild_Do_Table:                               
  Replicate_Wild_Ignore_Table:                               
                   Last_Errno: 0                             
                   Last_Error:                               
                 Skip_Counter: 0                             
          Exec_Master_Log_Pos: 684                           
              Relay_Log_Space: 534                           
              Until_Condition: None                         
               Until_Log_File:                               
                Until_Log_Pos: 0                             
           Master_SSL_Allowed: No                           
           Master_SSL_CA_File:                               
           Master_SSL_CA_Path:                               
              Master_SSL_Cert:                               
            Master_SSL_Cipher:                               
               Master_SSL_Key:                               
        Seconds_Behind_Master: 0                             
Master_SSL_Verify_Server_Cert: No                           
                Last_IO_Errno: 0                             
                Last_IO_Error:                               
               Last_SQL_Errno: 0                             
               Last_SQL_Error:                               
  Replicate_Ignore_Server_Ids:                               
             Master_Server_Id: 1                             
                  Master_UUID: fd482d37-146c-11ea-9944-0242ac110003                    
             Master_Info_File: /var/lib/mysql/master.info   
                    SQL_Delay: 0                             
          SQL_Remaining_Delay: NULL                         
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates  
           Master_Retry_Count: 86400                         
                  Master_Bind:                               
      Last_IO_Error_Timestamp:                               
     Last_SQL_Error_Timestamp:                               
               Master_SSL_Crl:                               
           Master_SSL_Crlpath:                               
           Retrieved_Gtid_Set:                               
            Executed_Gtid_Set:                               
                Auto_Position: 0                             
         Replicate_Rewrite_DB:                               
                 Channel_Name:                               
           Master_TLS_Version:                               
1 row in set (0.00 sec)                                                                
```

Slave_IO_Running 和 Slave_SQL_Running 都为 YES 则配置成功。

> 如果 Slave_IO_Running 为 connecting 可根据  Last_IO_Error 检查