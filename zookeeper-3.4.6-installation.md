---
title: Zookeeper-3.4.6 安装
date: 2015-07-14 16:00:38
tags:
 - Zookeeper
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

zookeeper 集群安装可以参考[http://zookeeper.apache.org/doc/r3.4.6/zookeeperAdmin.html#sc_systemReq](http://zookeeper.apache.org/doc/r3.4.6/zookeeperAdmin.html#sc_systemReq)。

### 1. 环境变量

zookeeper 是 Java 语言编写的，所以首先需要安装 Java 运行环境。

```
sudo echo "ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.6">/etc/profile.d/zookeeper.sh
sudo echo 'PATH=\$PATH :$ZOOKEEPER_HOME/bin'>>/etc/profile.d/zookeeper.sh
. /etc/profile
```
### 2. 配置文件

下载 zookeeper 。

```
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```

解压

```
tar -zxf zookeeper-3.4.6.tar.gz
```

修改配置文件。

```
cp /usr/zookeeper/zookeeper-3.4.6/conf/zoo_sample.cfd /usr/zookeeper/zookeeper-3.4.6/conf/zoo.cfd
```
添加集群配置信息。

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=192.168.0.201:2888:3888
server.2=192.168.0.202:2888:3888
server.3=192.168.0.203:2888:3888
```

> 集群配置的格式为 ```server.id=host:port:port``` ，host 是节点的主机地址，第一个 port 是 follow 服务器和 leader 服务器的通信端口，第二个 port 是 leader 选举使用的通信端口。

添加机器id

在 dataDir 下创建 myid 文件，文件中只有一行，记录机器的编号。编号必须是 1 - 255 之间的整数，必须唯一。

其他机器节点做相同操作，并拷贝配置文件到其他节点。

### 3. 启动

```
cd zookeeper-3.4.6/bin
./zkServer.sh start
```

> 停止 zookeeper 命令是 ```zkServer.sh stop```

测试一下

```
[root@node1 conf]# telnet 192.168.0.201 2181
Trying 192.168.0.201...
Connected to 192.168.0.201.
Escape character is '^]'.
stat
Zookeeper version: 3.4.6-1569965, built on 02/20/2014 09:09 GMT
Clients:
 /192.168.0.201:46827[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 4
Connection closed by foreign host.
```

看到类似信息则集群启动。

> 集群配置了 3 个节点，启动 1 台时集群不可用，启动 2 台时就可以看到类似上边的信息。

