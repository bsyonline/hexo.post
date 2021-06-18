---
title: One Question One Answer for ES
date: 2021-03-03 09:14:59
tags:
---



#### Q: 为什么要用 ES？

A: 优点：

​     缺点：



#### Q: ES、Solr、Lucene 的比较 

A: Lucene 是一个开源搜索引擎，Solr 基于 Lucene ，提供了 Http 接口的搜索引擎。ES 是基于 Lucene 的分布式搜索引擎。

#### Q: ES 的架构？

A: 

1. 



#### Q: ES 的读写原理？

A: 

1. 写数据

   1）ES 客户端随机连到一个节点（协调节点），协调节点将数据路由到对应索引的 primary shard 所在的机器上。

   2）数据首先写到 os cache 中，然后写到 translog 中， 每隔 1 秒将 os cache 中的数据 refresh 到 segment file 中。

   3）每隔 5 秒执行一次 commit ，将 translog 中的数据持久化到磁盘。

   4）当 segment file 数据到达阈值，会执行 merge 操作，将多个 segment file 合并成一个新的 segment file 。

2. 删除数据
   1）将删除的数据放在 .del 文件中，在 merge 的时候会将 .del 中的数据从 segment file 中删掉。

3. 查询

   1）客户端随机连到一个节点（协调节点），协调节点找到对应的 shard 。 

   2）然后在 primary shard 和 replica shrad 中随机找一个进行查询，将查询结果返回给协调节点。

   3）协调节点再将数据返回给客户端。

4. 检索

   1）客户端随机连到一个节点（协调节点），协调节点向每个 shard 发送检索请求。

   2）shard 将检索结果返回给协调节点，协调节点对数据进行合并得到最终结果。

   3）协调节点按照 id 去对应的 shard 上查询完整的 document 数据，再返回给客户端。


#### Q: ES 如何优化？ 

A: 

1. os cache
ES 查询会先从 os cache 中查，如果没有再查文件，将结果放到 os cache 中，下次查询直接从 os cache 中查询。所以查询的数据是否在 os cache 中会极大影响查询性能。以某项目为例，企业数量 5000 万，个体数量 1.3 亿，总数据量 2T 左右。生产 ES 集群5台，每台机器 192G ，ES 进程分配 32G ，os cache 分 160G 。ES 集群 os cache 总大小 800G ，按总数据量来看，可以覆盖 50% 左右的数据走内存。 

2. 索引设计
ES 对于关联字段，在写 ES 的时候就处理好，避免在查询时再进行复杂的连接查询。

3. 数据预热，冷热分离


4. 分页




#### Q: ES 在生产环境的部署情况？ 

A:  一般不会设置消息时效，如果真的失效了，只能自己写一个程序从源头开始一条一条手动补数据。



