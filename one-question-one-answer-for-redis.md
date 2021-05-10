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

#### Q: Redis 有哪些数据类型，及应用场景。

A: 数据类型有 8 种：

1. string，简单的 KV 缓存，比如单值缓存，再比如文章的阅读量。
```
127.0.0.1:6379> incr article:view:19 
127.0.0.1:6379> incr article:view:19 
127.0.0.1:6379> incr article:view:19
127.0.0.1:6379> get article:view:19 
"3"
```

2. list，list 使用场景很多，可以存储列表类型的数据，比如消息列表。
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

3. set，应用场景需要没有重复的元素，比如将歌曲从歌曲库加到我喜欢的歌曲列表。
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
4. zset，无重复且有序。比如排行榜。
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

5. hash，可以用来存储简单对象。比如数据库表 user 是这样的：
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

6. bitmap，可以用来实现 bloomfilter 。

7. hyperloglog，可以用来进行大数据量统计，比如 UV 。但是不精确。

8. geo，可以用来进行地理信息计算。

还有一些功能也列一下：
1. pub/sub，可以用来实现消息队列。
2. lua，利用它的原子性。
3. Pipeline，一次提交多条命令，一次返回多个结果，减少交互次数。
4. transaction，使用 multi ，exec ， discard 和 watch 实现。弱事务，可以保证全部执行，单不能同时成功，失败了不会回滚。

#### Q: Redis 底层数据结构有哪些？

A: 可变字节数组 (simple dynamic string, SDS) ，链表 (list) ，双哈希表 (dict) ，跳跃表 (zskiplist) ，intset ，ziplist ，quicklist ，zipmap 。

#### Q: Redis 为什么快？

A:  

1. 单进程单线程模型，没有上下文切换。
2. 使用多路复用无阻塞 I/O 模型。
3. 内存计算。
4. 数据结构简单，查找和操作的时间复杂度都是 O(1) 。

#### Q: Redis 是怎么样持久化的？

A: Redis 有两种持久化方式：

1. RDB，全量镜像方式，通过 fork 和 copy on write 方式实现，备份时间长，恢复速度快，可以分成多个文件存储，时间间隔长。
2. AOF，增量日志方式，对每条命令都生成日志，以 append-on 方式写文件，写入速度快，数据量大，频率高。

#### Q: Redis 的高可用是怎么做的？

A: 通过哨兵模式，至少 3 个节点，如果 master 挂了，slave 会被提升为 master 。哨兵模式只能保证高可用，不能保证数据不丢。



#### Q: Redis 主从之间是如何进行数据同步的？

A: 如果新加一个 slave 节点到集群，master 会生成一个 RDB 的镜像快照，将 RDB 发给 slave ，slave 将 RDB 保存到本地，然后加载到内存。如果在数据同步期间有增量数据，等 slave 通过 RDB 恢复了再把增量的部分发给 slave 。如果网络断了，会自动重连，网络恢复之后会重发。

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

