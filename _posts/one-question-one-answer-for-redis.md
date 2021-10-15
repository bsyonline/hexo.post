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

速度快

1. 单进程单线程模型，没有上下文切换。
2. 使用多路复用无阻塞 I/O 模型。
3. 内存计算。
4. 数据结构简单，查找和操作的时间复杂度都是 O(1) 。

### Redis 安装



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

如果在 append AOF 文件时宕机导致 AOF 文件损坏，可以用 redis-check-oaf --fix 命令修复。



### 主从架构

#### 主从的意义

可以支持读写分离，进而支撑高并发，并且可以方便的水平扩容。

#### 主从数据同步

写数据往 master 上写，写成功 master 就返回了。然后 master 会异步的将数据发送给 slave 。

如果新加一个 slave 节点到集群，master 会触发一次 full resynchronization ，启动一个后台线程，生成一个 RDB 的镜像快照（可以在磁盘上，也可以在内存中），同时将新的写请求缓存在内存中。当 RDB 文件生成完，master 将 RDB 发给 slave ，slave 先将 RDB 保存到本地，然后加载到内存。最后 master 再将缓存的写请求发送给 salve 。如果在同步过程中网络断了，会自动重连，网络恢复之后会重发。

从 redis 2.8 之后，主从复制支持断点续传。master 会在内存中创建一个 backlog ，在 backlog 中维护 replica offset 。如果 replica offset 不存在，则进行 full resynchronization 。

slave 不会过期 key ，只会等待 master 处理过期 key 。如果 master 的 key 过期或者被 LRU 淘汰，master 会发送一个 del 命令给 salve 来清除 key 。

#### 主从搭建

### 哨兵架构

#### 数据丢失场景及解决方案



#### 选举算法



#### 哨兵集群搭建



### redis cluster



#### Q: Redis 的高可用是怎么做的？

A: 通过哨兵模式，至少 3 个节点，如果 master 挂了，slave 会被提升为 master 。哨兵模式只能保证高可用，不能保证数据不丢。





#### Q: 哨兵模式 master 挂了如何进行选主？

A: 先看优先级，优先级相同，看复制偏移量（数据完整性），偏移量相同，取 id 最小的。

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

