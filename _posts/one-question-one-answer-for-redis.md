---
title: One Question One Answer for Redis
tags:
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-03-17 08:59:23
thumbnail:
---



### Redis 介绍

Redis 是一个开源的 kv 内存数据库。

Redis 速度很快：

1. 单进程单线程模型，没有上下文切换。
2. 使用多路复用无阻塞 I/O 模型。
3. 内存计算。
4. 数据结构简单，查找和操作的时间复杂度都是 O(1) 。

### Redis 安装

下载

```
wget https://download.redis.io/releases/redis-3.2.8.tar.gz
tar -zxf redis-3.2.8.tar.gz
cd redis-3.2.8
```

编译安装

```
make && make test && make install
```

> make test 过程可能报错:
>
> 1. You need tcl 8.5 or newer in order to run the Redis test
>
>    需要安装 tcl 。
>
>    ```
>    yum install tcl
>    ```
>
> 2. NOREPLICAS Not enough good slaves to write 
>
>    修改当前目录文件 tests/integration/replication-2.tcl 将 after 1000 改为 after 10000 以延长等待时间。

将 utils/redis_init_script 拷贝到 /etc/init.d/ 中，重命名为 redis_6379 。

```
cp utils/redis_init_script /etc/init.d/
cd /etc/init.d/
mv redis_init_script redis_6379 
```

创建 redis 的配置文件目录

```
mkdir -p /etc/redis
```

创建 redis 的持久化目录

```
mkdir -p /var/redis/6379
```

创建 redis 的日志目录

```
mkdir -p /var/log/redis/6379
```

修改 redis.conf 

```
daemonize yes									#让redis以后台进程运行
pidfile /var/run/redis_6379.pid					#redis pid文件的位置
port 6379										#端口号
dir /var/redis/6379								#redis持久化文件的存储位置
logfile /var/log/redis/6379/redis.log			#redis日志路径
```

将 redis.conf 拷贝到 /etc/redis ，重命名为 6379.conf 。

```
cp redis.conf /etc/redis/
cd /etc/redis/
mv redis.conf 6379.conf
```

启动 redis

```
cd /etc/init.d/
redis_6379 start
```

设置为开机启动

在 redis_6379 第二行加入 

```
# chkconfig: 2345 90 10
```

> 启动等级 2345 时自动启动 redis ，启动优先级为 90 ，关闭优先级为 10 。

生效 chkconfig

```
chkconfig redis_6379 on
```

重启系统验证 redis 是否正常启动。



### Redis 的数据类型

数据类型有 8 种：

#### 1. string

简单的 KV 缓存，比如单值缓存，再比如文章的阅读量。

```
127.0.0.1:6379> incr article:view:19 
127.0.0.1:6379> incr article:view:19 
127.0.0.1:6379> incr article:view:19
127.0.0.1:6379> get article:view:19 
"3"
```

#### 2. list

list 使用场景很多，可以存储列表类型的数据，比如消息列表。

```
127.0.0.1:6379> lpush 12:msg 101
127.0.0.1:6379> lpush 12:msg 104
127.0.0.1:6379> lrange 12:msg 0 5
1) "104"
2) "101"
```

再比如查询历史。

```
127.0.0.1:6379> lpush history "select * from user where id=1"
127.0.0.1:6379> lpush history "select * from dept where dept_name='sales'"
127.0.0.1:6379> lrange history 0 5
1) "select * from dept where dept_name='sales'"
2) "select * from user where id=1"
```

可以用来分页。

```
127.0.0.1:6379> lrange list 0 9
```

可以用来实现队列： lpush + rpop 。实现栈：lpush + lpop 。

#### 3. set

应用场景需要没有重复的元素，比如将歌曲从歌曲库加到我喜欢的歌曲列表。

```
127.0.0.1:6379> smove songs my_favourite "yestoday once more"
(integer) 1
```

再比如抽奖

```
127.0.0.1:6379> srandmember lottery_draw 1	#不移除
or
127.0.0.1:6379> spop lottery_draw 1		   #移除
```

再比如点赞

```
127.0.0.1:6379> sadd like:123 19		 #点赞
127.0.0.1:6379> srem like:123 19		 #取消点赞
127.0.0.1:6379> sismember like:123 19    #是否点过赞
127.0.0.1:6379> smembers like:123		#点赞用户列表
127.0.0.1:6379> scard like:123 		  #点赞数
```

再比如关注

```
127.0.0.1:6379> sadd tom:follow 101 102 199 
127.0.0.1:6379> sadd alice:follow 102 103 199 
127.0.0.1:6379> sinter tom:follow alice:follow 	  #tom和alice共同关注
1) "102"
2) "199"
127.0.0.1:6379> sdiff alice:follow tom:follow		#alice还关注了哪些人
1) "103"
```

#### 4. zset

无重复且有序。比如排行榜。

```
127.0.0.1:6379> zincrby music:month:1:week:1 1 "yestoday once more"
127.0.0.1:6379> zincrby music:month:1:week:1 1 "yestoday once more"
127.0.0.1:6379> zincrby music:month:1:week:1 1 "hotel california"
127.0.0.1:6379> zrevrange music:month:1:week:1 0 9 withscores 	#周排行
1) "yestoday once more"
2) "2"
3) "hotel california"
4) "1"
127.0.0.1:6379> zincrby music:month:1:week:2 1 "yestoday once more"
127.0.0.1:6379> zincrby music:month:1:week:3 1 "hotel california"
127.0.0.1:6379> zincrby music:month:1:week:4 1 "hotel california"
127.0.0.1:6379> zunionstore music:month:1 4 music:month:1:week:1 music:month:1:week:2 music:month:1:week:3 music:month:1:week:4 
127.0.0.1:6379> zrevrange music:month:1 0 9 withscores	#月排行
1) "yestoday once more"
2) "8"
3) "hotel california"
4) "5"
```

#### 5. hash

可以用来存储简单对象。比如数据库表 user 是这样的：

| id   | name  | age  |
| ---- | ----- | ---- |
| 1    | Tom   | 20   |
| 2    | Alice | 19   |

用 redis hash 可以这样表示：

```shell
127.0.0.1:6379> hmset user 1:name Tome 1:age 20
127.0.0.1:6379> hmset user 2:name Alice 2:age 19
127.0.0.1:6379> hmget user 1:name 1:age
1) "Tome"
2) "20"
```

如果表记录很多，可以进行分段，防止 big value 导致执行缓慢。 
再比如购物车，

```
# 用户 id 为 12 商品 id 为 1099
127.0.0.1:6379> hset cart:12 1099 1		 #添加商品
127.0.0.1:6379> hincrby cart:12 1099 1	  #增加数量
127.0.0.1:6379> hlen cart:12				#购物车中商品种类数量
127.0.0.1:6379> hdel cart:12 1099		   #删除商品
127.0.0.1:6379> hgetall cart:12 			#购物车中全部商品
```

#### 6. bitmap

可以用来实现 bloomfilter 。

#### 7. hyperloglog

可以用来进行大数据量统计，比如 UV 。但是不精确。

#### 8. geo

可以用来进行地理信息计算。

还有一些功能也列一下：

1. pub/sub，可以用来实现消息队列。
2. lua，利用它的原子性。
3. Pipeline，一次提交多条命令，一次返回多个结果，减少交互次数。
4. transaction，使用 multi ，exec ， discard 和 watch 实现。弱事务，可以保证全部执行，单不能同时成功，失败了不会回滚。



### Redis 的底层数据结构

#### 可变字节数组 (simple dynamic string, SDS) ，

#### 链表 (list) ，

#### 双哈希表 (dict) ，

#### 跳跃表 (zskiplist) ，

#### intset ，

#### ziplist ，

#### quicklist ，

#### zipmap 。



### Redis 持久化

#### 持久化方式

Redis 有两种持久化方式：

1. RDB，全量镜像方式，通过 fork 和 copy on write 方式实现。
2. AOF，增量日志方式。对每条命令都生成日志，fork 子进程以 append-on 方式写文件，当到达一定大小就会 rewrite 一个新的 AOF 文件，对旧的 AOF 文件进行压缩和merge ，完成后主进程将内存中的新日志追加到新的 AOF 文件中，继续写新的文件，把旧的文件删除。

#### RDB 和 AOF 的优缺点

RDB 的优点：

1. 可以分成多个文件存储，有 redis 控制固定时间间隔生成快照文件，适合做冷备。
2. fork 子进程来做 RDB ，对 redis 性能影响小。
3. RDB 是数据文件，数据恢复时，直接加载到内存，恢复速度快。

RDB 的缺点：

1. 可能会丢失一部分数据。
2. 如果 RDB 的间隔长，会导致 RDB 文件很大。

AOF 的优点：

1. AOF 数据丢失很少，通常最多丢失 1 秒的数据。
2. 顺序写磁盘，性能好，文件不容易损坏。
3. AOF 文件存的是 redis 命令，人可读。

AOF 的缺点：

1. 日志文件大。
2. AOF 每秒一次 fsync 刷 os cache ，性能比 RDB 低。
3. AOF 恢复比 RDB 更加复杂。

基于两种方式的优缺点，建议两种方式都开启，用 AOF 作为数据恢复的第一方案，用 RDB 来做冷备。

RDB 和 AOF 同时工作时

如果 RDB 在执行 snapshotting 那么不会执行 AOF rewrite ，如果在执行 AOF rewrite 那么就不会执行 RDB snapshotting 。

在执行 RDB snapshotting 时执行 BGREWRITEAOF 命令，在 snapshotting  结束后才会执行 AOF rewrite 。

如果 RDB 文件和 AOF 文件同时存在，redis 重启时，会优先使用 AOF 进行数据恢复。

#### 持久化配置

##### RDB 配置

默认是打开的，在 /etc/redis/6379.conf 中配置。

```
save 900 1
save 300 10
save 60 10000 #每隔60秒有10000个key变更
```

检查点可以配置多个，每到一个检查点，就会检查一下是否有指定数量的 key 变更，如果有就生成一个新的 dump.rdb 文件，覆盖旧的 dump.rdb 。

使用 redis-cli 的 SHUTDOWN 停止 redis 进程，是安全退出，退出前会生成一份完整的 rdb 快照。

##### AOF 配置

默认是关闭的，在 /etc/redis/6379.conf 中配置。

```
appendonly yes
appendfsync everysec #每秒将os cache中的数据fync到磁盘

#aof的触发策略，当前的aof文件大小超过上次rewrite之后的文件的100%，并且大于64mb，则触发新的rewrite
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

AOF 文件修复

如果在 append AOF 文件时宕机导致 AOF 文件损坏，可以用 redis-check-aof --fix 命令修复。

```
redis-check-aof --fix appendonly.aof
```



### 主从架构

#### 主从搭建

搭建 1-master-2-slave 读写分离集群。在3个节点安装 redis ，参考【redis 安装】。

在 master 上修改 /etc/redis/6379.conf ，打开口令认证。

```
requirepass 123456
```

在 slave 上修改 /etc/redis/6379.conf ，打开 slaveof

```
slaveof hadoop1 6379
slave-read-only yes		#默认打开，设置slave只读
```

slave 打开口令认证。

```
masterauth 123456
```

修改 bind ip ，主从节点都需要修改，默认是 127.0.0.1 ，只能本机访问，修改成自己的 ip 。

```
bind 192.168.67.201
```

重启验证主从集群是否正常。

#### 主从的意义

可以支持读写分离，进而支撑高并发，并且可以方便的水平扩容。

#### 主从数据同步

写数据往 master 上写，写成功 master 就返回了。然后 master 会异步的将数据发送给 slave 。

如果新加一个 slave 节点到集群，master 会触发一次 full resynchronization ，启动一个后台线程，生成一个 RDB 的镜像快照（可以在磁盘上，也可以在内存中），同时将新的写请求缓存在内存中。当 RDB 文件生成完，master 将 RDB 发给 slave ，slave 先将 RDB 保存到本地，然后加载到内存。最后 master 再将缓存的写请求发送给 salve 。如果在同步过程中网络断了，会自动重连，网络恢复之后会重发。

从 redis 2.8 之后，主从复制支持断点续传。master 会在内存中创建一个 backlog ，在 backlog 中维护 replica offset 。如果 replica offset 不存在，则进行 full resynchronization 。

slave 不会过期 key ，只会等待 master 处理过期 key 。如果 master 的 key 过期或者被 LRU 淘汰，master 会发送一个 del 命令给 salve 来清除 key 。

#### 主从同步性能优化

**fork 进程耗时问题**

fork 子进程需要拷贝父进程的空间内存页表，会耗费一定的时间。一般来说，如果父进程有1G数据，那么 fork 耗时可能在 20ms 左右，如果是 10G-30G 就会耗时几百毫秒，所以 fork 可能就会拖慢 redis 响应时长。所以建议 redis 的内存控制在 10G 以内。

**主从复制延迟问题**

主从复制可能严重延迟，这时需要建立监控和报警机制。在 info replication 中可以看到 master 和 slave 复制的 offset ，两者的差值就是复制的延迟量，可以通过脚本，定时监控 info replication 信息，如果延迟过多，就进行报警。

**主从复制风暴问题**

在进行 RDB 复制时，如果同时有很多个 slave 从 master 同步数据，一份 RDB 就会同时发送到多个 slave ，导致网络带宽被打满。为了避免这种情况，如果在 master 下挂载多个 slave 时，尽量用树形结构，不要用星型结构。

**AOF fsync 导致延迟**

一般 fsync everysec 一秒一次，redis 主线程会检查两次 fsync 的时间，如果距上次 fsync 的时间超过了 2 秒，那么就会阻塞写请求。建议使用 SSD 硬盘。

### 哨兵架构

#### 哨兵集群搭建

创建 sentinel 配置文件目录，将 sentinel.conf 拷贝到配置文件目录，重名为 5000.conf

```
mkdir -p /etc/sentinel
cp sentinel.conf /etc/sentinel/
cd /etc/sentinel
mv sentinel.conf 5000.conf
```

创建 sentinel 工作目录

```
mkdir -p /var/sentinel/5000
```

创建 sentinel 的日志目录

```
mkdir -p /var/log/sentinel/5000
```

修改 /etc/sentinel/5000.conf

```
bind 192.168.67.201 #自己机器的ip
prot 5000	#26379默认不能访问，需要配置防火墙
dir /var/sentinel/5000
logfile /var/log/sentinel/5000/sentinel.log
daemonize yes
sentinel monitor mymaster 192.168.67.201 6379 2
sentinel auth-pass mymaster "123456"
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

三个机器都要修改。

启动 sentinel 。

```
redis-sentinel /etc/sentinel/5000.conf
```

启动后会看到 sentinel 之间能够互相发现，则配置成功。

```
5015:X 16 Oct 15:57:45.739 # Sentinel ID is beae42f163a3808353cc546b45ac9c7ace383664
5015:X 16 Oct 15:57:45.739 # +monitor master mymaster 192.168.67.201 6379 quorum 2
5015:X 16 Oct 15:57:45.740 * +slave slave 192.168.67.202:6379 192.168.67.202 6379 @ mymaster 192.168.67.201 6379
5015:X 16 Oct 15:57:45.740 * +slave slave 192.168.67.203:6379 192.168.67.203 6379 @ mymaster 192.168.67.201 6379
5015:X 16 Oct 15:58:25.402 * +sentinel sentinel cbac5743cb7a1b4567e105345e58e2d65a2c6bb6 192.168.67.202 5000 @ mymaster 192.168.67.201 6379
5015:X 16 Oct 15:59:06.584 * +sentinel sentinel 95fe530cdf664055dba222bc1e19a3b2ae663ff0 192.168.67.203 5000 @ mymaster 192.168.67.201 6379
```



#### sdown 和 odown

sdown 是主观宕机，如果一个哨兵自己觉得 master 宕机了，那么就是主观宕机。

odown 是客观宕机，如果 quorum 的数量的哨兵觉得 master 宕机了，那么就是客观宕机。

如果一个哨兵 ping master ，超过了 is-master-down-after-milliseconds 指定的毫秒数，就认为是主观宕机。

如果一个哨兵在指定时间内收到了 quorum 数量的其他哨兵也认为 master 是 sdown ，那么就认为 master 是 odown 。

#### 哨兵的自动发现机制

哨兵之间相互发现是通过 pub/sub 系统实现的。每个哨兵都会向 \_\_sentinel\_\_:hello 这个 topic 里发一条消息，其他哨兵都消费这个 topic 来感知其他哨兵的存在。过程是这样的：

每隔 2 秒，每个哨兵都将自己的 host 、ip 、run id 以及对 master 的监控信息发送到自己订阅的 _\_sentinel\_\_:hello 中，同时也会监控其他哨兵的信息，并进行同步。

#### slave切换成master过程

哨兵监控集群状态，如果要进行 master 切换，首先需要 quorum 数量的哨兵认为 master odown ，然后选举出来一个哨兵做切换，同时这个哨兵需要得到 majority 数量的哨兵授权后才能正式执行切换。

进行切换时，执行切换的哨兵会从新的 master 获得一个 configuration epoch 作为 version ，切换完成，执行切换的哨兵会生成 master 的最新配置，通过 pub/sub 同步给其他哨兵，其他哨兵根据 version 大小来更新自己的 master 配置。

如果切换失败，那么其他哨兵会等待 failover-timeout 后继续执行切换，并会重新获取一个 configuration epoch ，每次切换 version 都是唯一的。

#### 选举算法

如果 slave 跟 master 断开连接已经超过了 down-after-milliseconds 的 10 倍 + master 宕机时长，那么这个 slave 不参加选举。

(down-after-milliseconds * 10）+ milliseconds_since_master_is_in_SDOWN_state

先看优先级，

如果优先级相同，看复制偏移量（数据完整性），offset 越大，优先级越高，

如果 offset 相同，选 run id 最小的。

#### 数据丢失场景及解决方案

**异步复制导致数据丢失**

redis master 向 slave 复制数据是异步的，如果写到 master 上的数据还没有复制到 slave 上，此时 master 宕机，哨兵将 slave 提成 master ，那么在原 master 上的这部分数据就丢失了。

**脑裂导致的数据丢失**

如果 master 和 slave 出现网络分区，哨兵将 slave 提成 master ，此时集群就有 2 个 master 就出现了脑裂。如果 client 连的还是原来的 master ，会继续向原 master 写数据，当分区恢复，原 master 变成 slave ，这部分数据就丢失了。

**解决方案**

```
min-slaves-to-write 1
min-slaves-max-lag 10
```

至少有 1 个 salve 的复制不能超过 10 秒。一旦slave复制数据和ack的延时太长，就认为master宕机会损失较多数据，那么就拒绝写请求，以此来将数据丢失控制在可控范围。

master 如果拒绝写请求，client 做降级，写本地磁盘缓存数据，等恢复后在继续处理。











### Redis Cluster

redis cluster 相当于 redis replication + 哨兵。要求最少 3 台 master 3台 slave 。生产环境下建议在 6 台机器上部署 redis cluster 。

#### 缓存算法

**hash 算法**

对节点数取模，使数据分布在不同的节点上。

弊端就是当一个节点宕机，会导致所有数据重新进行 hash 取模，相当于整个缓存数据不可用。

**一致性hash 算法**

将节点构成一个环，根据数据的 hash 值分布在环上位置，顺时针找下一个最近的节点进行数据存储。

优点就是当节点失效，只有一个节点上的数据失效。

弊端就是热点数据的可能存储不均导致某个节点负载过大。

**虚拟节点算法**

在一致性 hash 的基础上虚拟出多个虚拟节点。

**hash slot 算法**

集群节点均分 16384 个 hash solt ，根据 hash 对 16384 取模，找到对应的 hash solt ，如果一个节点宕机，那么只影响这个节点上的 hash solt 上的数据，redis 会自动将这个节点上的 hash solt 迁移到其他节点上。hash solt 的数量没有改变，所以不会影响其他 hash solt 上的数据。hash solt 的数量很多，也可以达到负载均衡的效果。

#### 集群节点通信

redis cluster 节点之间使用 gossip 协议进行通信。

gossip 协议包含多种消息：

meet：发送 meet 给新加入的节点，然后新节点开始与其他节点进行通信。

ping：每个节点都会频繁的发送 ping 消息来交换元数据。

pong：meet 和 ping 的响应信息。

fail：某个节点判定另一个节点故障，就发送 fail 给其他节点。

redis cluster 在各节点都维护一份元数据信息，每个节点都有一个专门用于节点间通信的端口（比如自己的服务端口是6379，那么通信端口就是 16379）。每个节点都会往另外几个节点发送 ping 消息，每个节点每秒会执行 10 次 ping ，每次会先择 5 个最久没有通信的节点。如果发现某个节点的通信延迟超过阈值（cluster_node_timeout/2），也会立刻发送 ping ，避免延迟过长。每次 ping 都会带上自己节点的信息和 1/10 个其他节点（最少 3 个）的信息发送出去，其他节点收到 ping 之后返回 pong 。通过这种方式来交换节点的故障信息，节点的增删信息，hash slot 信息等。

### docker 安装 redis

有时我们不想在自己机器上安装redis环境又想快速开始redis-cli实践，如果安装了docker环境，那么redis的环境将非常容易搭建。

1. 下载redis的docker镜像

   ```
   sudo docker pull redis
   ```

   

2. 启动redis server

   ```
   sudo docker run -d --name redisone redis redis-server
   ```

   

3. 启动redis cli

   ```
   sudo docker run -it --link redisone redis redic-cli -h redisone -p 6379
   ```

   

### spring-data-redis 的使用



添加依赖

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.6.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.6.2</version>
</dependency>
```

配置文件

redis.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd">


    <context:property-placeholder location="classpath:config.properties"/>
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.cache.pool.maxTotal}" />
        <property name="maxIdle" value="${redis.cache.pool.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.cache.pool.maxWaitMillis}" />
        <property name="testOnBorrow" value="${redis.cache.pool.testOnBorrow}" />
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
        </property>
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"/>
        </property>
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="${redis.cache.ip}" />
        <property name="port" value="${redis.cache.port}" />
        <property name="usePool" value="${redis.cache.usePool}"/>
        <property name="poolConfig" ref="jedisPoolConfig" />
    </bean>

    <bean id="queryCacheRepository" class="com.daas.data.dao.repository.QueryCacheRepository"></bean>

</beans>
```

properties文件

```
redis.cache.pool.maxTotal=1024
redis.cache.pool.maxIdle=200
redis.cache.pool.maxWaitMillis=1000
redis.cache.pool.testOnBorrow=true
redis.cache.ip=192.168.11.21
redis.cache.port=6379
redis.cache.usePool=true
```



```
public class QueryCache implements Serializable{

    String id;
    String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}

public class QueryCacheRepository {

    @Resource
    private RedisTemplate<String, QueryCache> redisTemplate;

    public void set(QueryCache qc){
        redisTemplate.opsForValue().set(qc.getId(), qc);

    }

    public QueryCache get(String id){
        return redisTemplate.opsForValue().get(id);
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:redis-test.xml")
public class RedisServiceTest {


    @Resource
    QueryCacheRepository queryCacheRepository;

    @Before
    public void before() {
        QueryCache qc = new QueryCache();
        qc.setId("123");
        qc.setValue("test");
        queryCacheRepository.set(qc);
    }

    @Test
    public void testGet() throws IOException {
        QueryCache qc = queryCacheRepository.get("123");
        System.out.println(qc.getValue());
        assertThat(qc.getValue(), new IsEqual("test"));

    }

}
```









#### Q: Redis 的高可用是怎么做的？

A: 通过哨兵模式，至少 3 个节点，如果 master 挂了，slave 会被提升为 master 。哨兵模式只能保证高可用，不能保证数据不丢。





#### Q: 哨兵模式 master 挂了如何进行选主？

A: 

#### Q: Redis key 的过期的 key 是怎么删除的？

A: 如果 redis 中的 key 过期了，就会返回 nil 。但是这个 key 并没有被立刻删除。Redis 的失效的 key 的删除策略有 2 种：

1. 被动删除(passive)：当再次访问这个 key 时发现它已经过期，则删除。
2. 主动删除(active)：有些 key 失效之后可能再也不会被访问，所以无法被动删除，只能主动删除。Redis 会从过期的集合中随机挑一些 key 删除，如果失效 key 大于 25% ，主动删除会一直进行。
3. 如果还删不掉，就只能等 LRU 了。

#### Q: 内存淘汰策略是怎样的？

A: noeviction，allkeys-LRU，volatile-LRU，allkeys-Random，volatile-Random，volatile-ttl 。

#### Q: 主从模式，哨兵模式，集群模式的区别。

A: 主从是备份关系，如果主挂了，从可以接替主继续工作，但是切换需要人工。主从可以支持高并发。

哨兵+主从是在主挂了之后由程序自动将从提成主，保证高可用，哨兵最少 3 台。

集群模式是将数据自动分配到 hash slot 中，多台机器同时工作，提高并发，集群最少 6 台，3 主 3 从。集群模式等效于多个主从+哨兵。

#### Q: 如何解决缓存+数据库双写一致性问题？

A: 有 3 种比较成熟的模式：

**Cache Aside Pattern**



**Read/Write Through Pattern**



**Write behind Pattern**



####  Q: 缓存穿透、缓存雪崩是什么？如何解决？

A: 

**缓存穿透**

缓存穿透是指大量请求访问一个缓存中不存在的 key ，导致缓存未命中进而访问数据库。

解决方案：对不存在的 key 缓存一个默认值。

**缓存雪崩**

缓存雪崩是指在同一时刻，缓存中的 key 失效，导致缓存未命中进而访问数据库。

解决方案：对 key 的失效时间使用随机数。

**缓存击穿**

缓存击穿是指热点 key 失效后，大量请求热点 key 的请求缓存未命中，进而访问数据库。

解决方案：热点 key 不设置失效时间，如果有失效时间，在重建缓存的时候加锁保证只有一个请求进行缓存重建，其他的请求还是查询缓存。如果有多个热点 key ，缓存重建时，使用分段锁。

如何做好 redis 高可用

#### ZSet跳表是如何设计与实现的

#### ZSet实现压缩列表和跳表如何选择

#### 6.0 多线程模型比单线程做了哪些优化

如何让redis集群支撑几十万QPS+4个9高可用+海量数据？企业级集群结构

支撑高性能，高并发，及安全保护。nginx+lua + redis + ehcache 三级缓存架构

高并发场景数据库+缓存双写一致性方案

如何解决大value更新效率低的问题？缓存维度化拆分方案

如何提升缓存命中率？双层nginx部署架构，以及lua脚本实现一致性hash流量分发策略。

如何解决缓存重建是的分布式并发重建冲突问题？基于zk分布式锁的缓存并发重建方案

如何解决缓存冷启动mysql被打死问题？基于storm试试统计热数据的分布式快速缓存预热方案

如何解决热点缓存导致单机负载瞬间超高？基于storm的实时热点发现，实时热点缓存负载均衡降级

基于hystrix的高可用缓存服务，资源隔离+限流+熔断+超时控制

基于hystrix的容错，多级降级，手动降级，生产环境参数优化，可视化运维与监控

如何解决婚车雪崩，事前+事中+事后

如何解决缓存穿透问题，

如何解决缓存失效问题

