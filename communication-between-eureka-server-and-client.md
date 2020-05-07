---
title: Communication between Eureka Server and Client
tags:
  - Microservices
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:06:28
thumbnail:
---





Eureka Server 和 Eureka Client 之间是通过一组 API 进行通信的，包括服务注册（registry），续约（renew），退出（cancel）和剔除（evict）。

### 注册

Eureka Client 启动之后首先会向 Eureka Server 进行注册，注册流程如下：

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180902/213336520.png" style="width:600px;" />

1. Client 通过 ```POST /eureka/v2/apps/{appID}``` 向 Server 发起注册请求；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/194055677.png)

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/193909570.png)

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/195210819.png)

2. Server 收到请求后，用双层 HashMap 保存注册信息；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/202843068.png)

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203115942.png)

3. 更新自我保护机制阈值 ```expectedNumberOfRenewsPerMin``` 和 ```numberOfRenewsPerMinThreshold``` ;

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203204566.png)

   > - `expectedNumberOfRenewsPerMin` ，期望**最大**每分钟**续租**次数。
   > - `numberOfRenewsPerMinThreshold` ，期望**最小**每分钟**续租**次数。

4. 将服务名字放到 ```recentRegisteredQueue``` 中；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203238017.png)

5. 更新实例的状态；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203428268.png)

6. 将实例信息放到 ```recentlyChangedQueue``` 中；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203518017.png)

7. 通过 ```invalidate()``` 清除 guava 缓存；

   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180902/203555642.png)

8. 向相邻节点复制信息。

### 续约

Eureka Client 注册到 Eureka Server 后，需要定期（每30s）发送心跳来维持通信。如果超时未收到心跳，服务器会认为服务不可用，从而将 Eureka Client 的注册信息从服务器删除。Renew 的流程如下：
   <img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180916/181214611.png" style="width:600px;" />
1. Eureka Client 通过定时任务向服务器发起 renew 请求
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/103505114.png)
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/103600120.png)
2. Eureka Server 收到请求后会更新该 Client 的更新时间
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/105522756.png)
   更新最小分钟数 renewsLastMin 和 最后更新时间
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/110101803.png)
3. Eureka Server 向相邻节点同步信息
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/105217080.png)

### 剔除

在 Eureka Server 上未正常推出和超时未收到心跳的 Client 注册信息会被剔除。 Evict 的流程如下：
   <img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180916/203523620.png" style="width:600px;" />
1. 初始化环境和上下文
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/111039577.png)
2. 从相邻节点同步注册信息
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/173229607.png)
3. 更新自我保护阈值
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/194857681.png)
4. 创建 evictionTask
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/174851249.png)
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/194744627.png)
5. 判断是否过期
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/195021721.png)
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/195402678.png)
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/195834558.png)
6. 计算剔除的数量并随机删除
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/200711079.png)
7. 更新缓存

### 退出

退出是 Eureka Client 主动向服务通知服务关闭的操作，流程如下：
   <img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180916/210042210.png" style="width:600px;" />

1. Eureka Client 向服务器发送 cancel 请求
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20180916/204524786.png)
2. Eureka Server 收到请求后，将实例从服务列表中删除
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/182926978.png)
   将实例信息放到 recentlyChangedQueue 中，清空 guava 缓存
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/183358439.png)
3. 向相邻节点同步信息，更新阈值
   ![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181209/183044458.png)

