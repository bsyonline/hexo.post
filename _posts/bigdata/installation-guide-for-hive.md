---
title: Installation Guide for Hive
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-04-14 23:09:10
thumbnail:
---



### 安装 MySQL

#### 1. 删除已有安装

```
rpm -qa | grep mysql
rpm -qa | grep mariadb
rpm -e --nodeps mariadb
whereis mysql
rm -rf /usr/lib64/mysql
```



#### 2. 解压

```
tar -zxf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.24-linux-glibc2.12-x86_64 mysql
```

#### 3. 创建数据目录

```
cd /opt/mysql
mkdir data
```

#### 4. 创建用户

```
groupadd mysql
useradd -r -g mysql mysql
chown -R mysql:mysql /opt/mysql/
```

#### 5. 新建配置

vim /etc/my.cnf

```
[client]
port = 3306
socket = /tmp/mysql.sock

[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
basedir=/opt/mysql
datadir=/opt/mysql/data
log-error=/var/log/mysqld.log
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=600
innodb_file_per_table=1
lower_case_table_names=1
character_set_server=utf8
```

#### 6. 初始化

```
bin/mysqld --initialize --user=mysql --basedir=/opt/mysql --datadir=/opt/mysql/data
```

#### 7. 开机启动yuPn4qh5wj0

vim support-files/mysql.server

```
basedir=/opt/mysql
datadir=/opt/mysql/data
```

```
cp support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
service mysqld start
```

#### 8. 登录修改密码

```
ln -s /opt/mysql/bin/mysql /usr/bin
mysql -uroot -p
```

密码查看/var/log/mysql.log

```
mysql>set password=password('123456');
```

#### 9. 添加远程访问权限

```
mysql>grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
mysql>flush privileges;
```



### 安装 Hive

#### 1. 解压

```
tar -zxf apache-hive-2.3.8-bin.tar.gz
mv apache-hive-2.3.8-bin.tar.gz hive-2.3.8
```

#### 2. 修改配置

vim conf/hive-site.xml

```xml
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://hadoop1:3306/hivedb?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8&amp;useSSL=false</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>123456</value>
  </property>
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
  </property>
</configuration>
```

#### 3. 拷贝mysql驱动到lib目录

#### 4. 配置环境变量

```
export HIVE_HOME=/opt/hive-2.3.8/
export PATH=$PATH:$HIVE_HOME/bin
```

#### 5. 初始化元数据

```
./bin/schematool -dbType mysql -initSchema
```

#### 6. 启动

