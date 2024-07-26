---
title: Architecture Design
tags:
  - Interview
category:
  - Architecture
author: bsyonline
lede: 没有摘要
date: 2099-08-11 22:21:42
thumbnail:
---





## Part I 架构设计

### 架构演进

架构是解决问题的过程和方案。架构不是一成不变的，一个问题可以用不同的架构来解决，但是肯定有一种架构是最合适的。所以架构没有好坏，只有是否合适。如果我们脱离了实际问题讨论架构，那将是空中楼阁。架构是经验的总结和提炼，经过不断的抽象，逐渐形成以下几种架构。


#### Monolith 架构 

Monolith 就是单体应用程序，即单个应用程序包含所有的功能，一个应用就是一个进程。
<img src="https://s2.ax1x.com/2020/02/27/3dNjRs.png" alt="3dNjRs.png" border="0" style="width:550px"/>
单体架构是我们最熟悉的一种架构，它的优点很明显，易于开发，测试，部署和扩展。同时它的缺点也很明显，就是所有功能耦合在一起，修改任何一个功能，都需要对所有功能进行测试，不利于做持续集成和发布。所以如果我们的场景不需要持续集成持续发布，那么单体架构是非常不错的选择。

#### 分层架构

在单体应用中，往往我们为了开发和维护方便，我们会将软件划分成多个层次，比如常见的 3 层架构：表现层，业务层和数据访问层。
<img src="https://s2.ax1x.com/2020/02/27/3dUioF.png" alt="3dUioF.png" border="0" style="width:550px"/>
每一层都有清晰的角色和分工，层与层之间通过接口通信，如果多人协同开发，不同的人员负责不同的层，每一层都是相对独立的，可以单独进行测试和部署。这对于大型的公司或是多人协同开发是非常合适。这种水平分层架构解决了功能耦合的问题，但是增加了调试和定位问题的难度，同时也对部署的要求更高。

#### SOA 架构

SOA 即面向服务的架构，它和分层架构的思想类似。水平分层是从功能层面解耦，而 SOA 则是从业务层面进行解耦。
<img src="https://s2.ax1x.com/2020/02/27/3dUYQI.png" alt="3dUYQI.png" border="0" style="width:500px"/>
优点是业务比较独立，服务之间通过接口通信，内部调整不会影响其他服务。但是我们同时也会看到对业务的划分比水平层次的划分更难。

#### 微服务架构

微服务架构，是在 2014 年由 James Lewis 和 Martin Fowler 提出的，更多信息可以参考 [https://www.martinfowler.com/articles/microservices.html](https://www.martinfowler.com/articles/microservices.html) 。简单来说微服务架构就是水平分层架构 + SOA 架构。
<img src="https://s2.ax1x.com/2020/02/27/3dUwTS.png" alt="3dUwTS.png" border="0" style="width:550px" />
水平分层架构有业务耦合，SOA 架构有功能耦合，微服务架构在垂直方向进行业务解耦，在水平方向进行功能解耦，完全解决了耦合性的问题。其优势在于各个服务之间耦合度低，具有很好的扩展性。每个服务可以单独开发，测试和部署，单个服务调整不会影响其他服务，利于做持续集成和发布。各个服务拆分开来，不用再囿于相同的语言和环境，各个团队可以根据自身的技术选择实现方式。

但是凡事有两面，我们不能光看到微服务架构的优点，同时我们也应该清楚微服务架构的缺点。由于要解耦，服务会被拆分成大量的微服务，而服务之间需要进行通信这就导致系统的变得极为复杂。同时由于通信链路变多，微服务的性能和单体架构相比会有所损失。还有在服务被拆分之后，在分布式环境中，数据的一致性也变得难以保证，需要我们额外使用一些手段来保证数据一致性。在大量的微服务面前，也对运维提出了极高的要求，所以虽然微服务在当下十分流行，但是在架构选择的时候还要根据自身的情况进行衡量，不要盲目跟风。

#### 服务网格架构

Service Mesh 服务网格架构是由 Linkerd 提出的一种微服务架构，伴随着微服务的流行，Service Mesh 也越来越被人们关注。在微服务时代，服务治理是微服务架构的首要难题和痛点。微服务通过水平拆分功能解耦，垂直拆分业务解耦，已经极大降低了耦合性。但是微服务之间要通信，那么每个微服务中又必须包含相关的通信组件。通信和业务耦合不利于业务快速迭代，同时也不利于通信组件的升级。所以在这种情况下，Service Mesh 就将通信从服务中剥离出来，从而提高业务的迭代速度和降低通信组件的维护难度。
<img src="https://s2.ax1x.com/2020/02/27/3dUH61.png" alt="3dUH61.png" border="0" style="width:750px"/>

### 高可用

#### 什么是高可用

高可用（High Availability，HA）简单来说，就是用户在任何时间访问系统都能获得正确的结果。如果系统不具有高可用性，那么意味着在某一个时刻用户无法使用系统或服务，用户体验会不好，更有可能使公司丢掉商业机会，造成损失，所以企业对高可用系统的重要性越来越重视，同时也提出一系列衡量高可用系统的标准，比如：按停机时间计算。

> 一年 365 天，如果系统能够在 99% 的时间里提供服务，那么意味着系统一年之中停机时间不能超过 87.6 小时。
> 如果要保证 3 个 9 可用，停机时间不能超过 8.76 小时。
> 如果是 4 个 9 则停机时间不能超过 53 分钟。
> 如果是 5 个 9 则停机不能超过 6 分钟。

停机时间可以从一个方面反应出系统的可用性，但是在高峰期停机 6 分钟和在低峰期停机 6 分钟造成的影响是不一样的，所以又提出了另一种方法，通过估算停机时的请求量/一年的总请求量的比值来进行评估。不管使用何种评估方式，我们都可以对我们系统的高可用性有一个大致的了解。如果想要提高系统的可用性，那么就需要在系统的架构设计中进行改进。

#### 如何实现高可用

1. 服务冗余

5 个 9 的服务可用性是非常具有挑战的，因为我们的系统是软件，这些软件会有 bug ，而软件又是运行在计算机硬件上的，硬件也会出现故障，所以单一系统出现问题是无法避免的。为了解决单点问题，就必须将我们的服务冗余，进行集群，从而保证集群中任何节点出现故障都不影响系统整体的可用性。

2. 负载均衡

服务冗余固然可以消除单点故障的影响，但是仅仅冗余部署是不够的，必须要有一套监控机制来保证。服务部署多份，那么如何将请求分配到不同的节点，保证不会出现某个节点接受请求过大而另一些节点请求过少，或者当一个节点故障之后不会再有请求被分配到故障节点，这就需要使用负载均衡技术。

3. 幂等

当一个节点的请求处理失败，负载均衡会将请求路游到另外的节点上进行重试，为了保证系统不会因为多次请求导致数据不一致，我们还需要保证幂等性。

4. 无状态及动态扩容缩容

当请求不断增多，集群压力过大，如果有富裕的硬件资源，我们需要增加服务节点，当请求减少，我们需要减少服务节点，回收多余的硬件资源。为了能够动态的扩容和缩容，服务必须是无状态的，否则每个节点都是不一样的，那么处理起来就会非常困难。

5. 限流、降级、熔断

有时在很短的时间内我们的系统资源无法支撑海量的请求，为了保证系统可用，我们需要对系统进行限流，降级或熔断处理。**限流**就是当请求数量超过系统的处理能力时，丢弃一部分请求以保证一部分请求能够正常处理。**降级**就是当系统资源不足时，将一部分边缘服务使用默认处理或是停掉，以保证核心服务资源可用。**熔断**是当一个服务请求处理缓慢时，使用服务的默认行为，防止因为单个服务缓慢导致整个系统雪崩。

6. 服务拆分

将服务拆分，按照服务等级施行不同策略的保障措施。

7. 服务监控

从平均响应延时或异常条数来进行实时监控，需要有对日志实时采集和分析的能力。

8. 热部署或热切换

在不停机的情况下进行服务更新或状态修改，比如使用配置中心。

9. 异地容灾

不管是应用还是数据，异地部署都是有效的手段。比如，同城不同机房，异地多个机房等。

[无状态设计]()

[负载均衡]()

### 幂等设计

幂等设计是架构设计中需要考虑的重要环节。幂等设计可以分两个层面考虑：接口定义和幂等实现方案。

#### 接口定义

请求幂等是说请求一次和请求多次结果一样。其实这个说法有一点不准确，因为如果是看请求结果的话，那么在读请求在数据被修改之后，得到的结果也是不同的。所以准确的说应该是看请求是否有副作用，比如每次请求都会将数据修改成不一样的值。按照这个定义，读请求不会修改数据本身，所以读请求是幂等的。而写请求是会修改数据本身的，所以写请求不是幂等的。大多数情况下，我们接口的设计都是遵守 RESTful 规范。在 RESTful 中定义了 7 种方法：

1. OPTIONS，获取服务器信息，幂等。
2. HEAD，请求资源的头信息，幂等。
3. GET，获取资源信息，幂等。
4. POST，新建资源，非幂等。
5. PUT，全量更新，幂等。
6. DELETE，删除资源，幂等。
7. PATCH，局部更新，非幂等。

所以在设计接口的时候需要和 RESTful 的规范保持一致。

#### 幂等实现方案

在由于写请求不是天然幂等的，所以我们在架构设计的时候需要考虑幂等性。我们可以尝试通过如下几种思路来解决幂等性问题。

1. 利用数据库约束

由于请求最终都是操作数据库，所以最终都会反映到数据库的 CRUD 操作，所以我们可以利用数据库主键或唯一索引来保证唯一。

2. 去重表

单独维护一张表保存去重字段，比如订单号，在业务操作之前先插入订单号，插入订单号和业务处理在同一个事务中，同时成功提交或者回滚。这种方案是第一种方式的变形，好处是业务无关，多个业务可以共享一张表。

3. 乐观锁

在进行更新和删除操作时比较常用。可以利用 version 字段，如

```
update t1 set age=age+1, version=version+1 where id=1 and version=1
```

也可以使用业务字段，如

```
update t1 set age=age+1 where id=1 and age=19
```

>在进行更新和删除操作时，最好使用绝对值，不要使用相对值。

4. 使用全局唯一 ID

5. Token

1) 客户端首先向服务器发送请求获取 token ，服务器生成 token 存到缓存中，服务器返回 token 给客户端。
2) 客户端业务请求同时带着 token ，如果服务端存在 token 说明是第一次操作，可以进行业务处理，如果 token 不存在，说明业务已经执行过。
3) 业务执行完成删除 token 。
这种方式可以防止客户端重复提交，缺点是多了一次获取 token 的请求。

6. 分布式锁

7. MQ 消息去重

8. 状态机

状态机是有限状态自动机的简称，是现实事物运行规则抽象而成的一个数学模型。状态机有起始、终止、现态、次态（目标状态）、动作、条件 6 中元素。比如一个账号支付状态只能从未支付到支付，且只能进行一次。

9. 通过资源条件保证幂等

比如，有一个资源总量是 10 ，进行增加操作，每次增加都先判断资源总量，如果没到 10 就增加，如果到 10 就退出，这样不管执行多少次都是不会改变事件整体的幂等性。

虽然我们列出几种处理方式，但是并没有一种方式是完美的，需要根据业务和架构进行选择。



### 分布式锁

一个买票的场景：

```
public void reduce(int num) {
    Jedis jedis = jedisPool.getResource();
    lock.lock();
    Integer tickets = Integer.parseInt(jedis.get("ticket"));
    boolean buyTicket = false;
    if (tickets - num >= 0) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        tickets = tickets - num;
        jedis.set("ticket", String.valueOf(tickets));
        log.info("用户{}买到1张票,还剩{}张票", Thread.currentThread().getId(), tickets);
        buyTicket = true;
    } else {
        log.info("余票不足,用户{}没有买到票", Thread.currentThread().getId());
    }
    if (buyTicket) {
        if (Thread.currentThread().getId() % 2 == 0) {
            // vip加500积分
            log.info("用户{}是VIP,获得500积分", Thread.currentThread().getId());
        }
    }
}
```

10 个人买 5 张票，正常只有 5 个人能够买到票，执行程序结果如下：

```
用户9买到1张票,还剩-3张票
用户5买到1张票,还剩2张票
用户1买到1张票,还剩-2张票
用户7买到1张票,还剩-5张票
用户6买到1张票,还剩3张票
用户4买到1张票,还剩1张票
用户8买到1张票,还剩-4张票
用户4是VIP,获得500积分
用户6是VIP,获得500积分
用户0买到1张票, 还剩4张票
用户0是VIP,获得500积分
用户2买到1张票, 还剩-1张票
用户8是VIP,获得500积分
用户2是VIP,获得500积分
用户3买到1张票, 还剩0张票
```

10 个人都买到了票，说明出现了多线程并发安全问题。要解决多线程并发安全问题很容易想到 synchronized 关键字。在 reduce() 方法上加上 synchronized 就可以解决并发安全问题。

```
public synchronized void reduce(int num) {}
```

但是，synchronized 加在方法上性能不高，所以我们就会想到另一种 synchronized 用法，只把竞态条件的代码用 synchronized 包起来。

```
public void reduceWithSynchronized(int num) {
    Jedis jedis = jedisPool.getResource();
    boolean buyTicket = false;
    synchronized (this) {
        Integer tickets = Integer.parseInt(jedis.get("ticket"));
        if (tickets - num >= 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            tickets = tickets - num;
            jedis.set("ticket", String.valueOf(tickets));
            log.info("用户{}买到1张票,还剩{}张票", Thread.currentThread().getId(), tickets);
            buyTicket = true;
        } else {
            log.info("余票不足,用户{}没有买到票", Thread.currentThread().getId());
        }
    }
    if (buyTicket) {
        if (Thread.currentThread().getId() % 2 == 0) {
            // vip加500积分
            log.info("用户{}是VIP,获得500积分", Thread.currentThread().getId());
        }
    }
}
```

还可以使用 Lock ，效果和 synchronized 相同。

```
Lock lock = new ReentrantLock();

public void reduceWithLock(int num) {
    boolean buyTicket = false;
    try {
        Jedis jedis = jedisPool.getResource();
        lock.lock();
        Integer tickets = Integer.parseInt(jedis.get("ticket"));
        if (tickets - num >= 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            tickets = tickets - num;
            jedis.set("ticket", String.valueOf(tickets));
            log.info("用户{}买到1张票,还剩{}张票", Thread.currentThread().getId(), tickets);
            buyTicket = true;
        } else {
            log.info("余票不足,用户{}没有买到票", Thread.currentThread().getId());
        }

    } finally {
        lock.unlock();
    }
    if (buyTicket) {
        if (Thread.currentThread().getId() % 2 == 0) {
            // vip加500积分
            log.info("用户{}是VIP,获得500积分", Thread.currentThread().getId());
        }
    }
}
```

不管是 synchronized 还是 Lock 都是 JVM 级别的锁，单体环境下没有问题，但是如果是在分布式环境下就不能保证并发安全了。



首先把单体应用改成分布式应用。

启动 2 个 Ticket 服务，端口分别为 8081 和 8082 。通过 nginx 来做负载均衡。

```
http {
	upstream myticket{
		server 127.0.0.1:8081;
		server 127.0.0.1:8082;
	}
	server {
		listen 8888 ;
		charset utf-8;
		location /tickets{
        	proxy_pass http://myticket;
		}
	}
}
```

然后还是单体一样，模拟并发来调用 http 接口。运行结果可以看到，虽然都加了 Lock ，但是还是出现了超卖的情况，说明 JVM 级别的锁在分布式环境下是不能解决并发安全问题的。所以我们需要使用分布式锁。

#### 基于 redis 实现分布式锁

```
public void reduce(int num) {
    String lockKey = "ticket_lock";
    String lockId = UUID.randomUUID().toString(); // 1
    Jedis jedis = jedisPool.getResource();
    try {
//            Long result = 0L;
//            while (0 == result) {
//                result = jedis.setnx(lockKey, lockId);
//                jedis.expire(lockKey, 10);
//            }
        String result = "";
        while (!"OK".equals(result)) {
            result = jedis.set(lockKey, lockId, "NX", "PX", 10000); // 2
            expireTimer = new Timer();
            expireTimer.schedule(new TimerTask() { // 3
                @Override
                public void run() {
                    jedis.expire(lockKey, expire);
                }
            }, 0, period);
        }
        boolean buyTicket = false;
        Integer tickets = Integer.parseInt(jedis.get("ticket"));
        if (tickets - num >= 0) {
            tickets = tickets - num;
            jedis.set("ticket", String.valueOf(tickets));
            log.info("用户{}买到1张票,还剩{}张票", Thread.currentThread().getId(), tickets);
            buyTicket = true;
        } else {
            log.info("余票不足,用户{}没有买到票", Thread.currentThread().getId());
        }
        if (buyTicket) {
            if (Thread.currentThread().getId() % 2 == 0) {
                // vip加500积分
                log.info("用户{}是VIP,获得500积分", Thread.currentThread().getId());
            }
        }
    } finally {
//            if (lockId.equals(jedis.get(lockKey))) {
//                jedis.del(lockKey);
//            }
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(lockId)); // 4
    }
}
```

上边是基于 jedis 的 redis 分布式锁的实现示例，基本思想就是在 redis 里维护一个 key ，调用业务代码前先来设置 key ，如果设置成功就获得了锁，否则就等待重试。代码比较简单，但是有几个地方需要注意：

1. lockId 的作用

   每一把锁创建都会有一个唯一 id ，唯一 id 的作用就是防止自己的锁被别人删掉。

2. setnx

   setnx 无法指定锁的失效时间，需要使用 expire 来设置锁失效时间，但是这两个操作不是原子操作，在高并发场景下会出现死锁。所以在高版本的 redis 中使用 set 指定参数的方式来将两个操作作为一个原子操作。

3. 续租

   redis 锁续租是一个比较麻烦的问题，通常我们可以根据业务来估算处理时间，锁的失效时间应该大于业务执行的时间。但是估算难免存在误差，所以为了保证在业务执行完之前锁不会因为超时而释放掉，我们可以通过后台线程来定时设置 lockKey 的失效时间，从而达到锁续租的目的。

4. 释放锁

   在锁用完之后需要通过删除 lockKey 来释放锁，所以释放锁的代码应该放在 finally 中保证一定会被执行。在执行删除时需要做判断，保证自己只能删除自己的锁。这里同样会有非原子性的问题，解决方法是使用 lua 脚本将操作变成原子操作。

以上就是 redis 锁的实现，注意上边 3 点，基本就可以满足一些场景的锁的需求，虽然我们考虑了很多异常的场景，也针对这些场景进行预防，但是并没有解决一个根本问题，就是 redis 本身是没有一致性的概念的，并不是一个 CP 模型，而锁本身是一个 CP 的需求，所以从根本上来说两者是冲突的。但是这并不妨碍我们在一些一致性要求不高的场景下使用一个简单 redis 实现的分布式锁。

#### 基于 zookeeper 实现分布式锁







#### 基于 etcd 实现分布式锁



### 分布式事务

#### 本地事务

##### 事务 ACID 特性

我们通常说的事务是指本地事务，即对一个数据库进行的操作，并且这些操作满足事务的 4 个特性。事务的 4 个特性分别是：
Aotmicity（原子性），Consistency（一致性），Isolation（隔离性），Durability（持久性）。在一个事务中的所有操作要么同时成功，要么同时失败，这是原子性，一旦执行成功 commit 或是执行失败 rollback 那么这个结果必须是持久的。在事务开始和结束之间，存在着中间状态，这些中间状态是对事务之外是隔离的，换句话说，在事务结束之前，对数据进行的任何修改对事务之外都是不可见的，这就是隔离性。在事务结束之后，数据必须是完整并且一致的。

##### 隔离级别

1. RU，Read UnCommitted

   级别最低，会出现脏读、不可重复读和幻读的问题。

   脏读：t2 事务读到了 t1 事务没提交的数据。

   不可重复读：t1 事务查询一条记录，t2 事务修改了这条记录，t1 事务再次读取的结果和第一次不一致。

   幻读：t1 事务查询了数据，t2 事务插入了一条新数据，t1 事务再次查询记录数和第一次不一致。

2. RC，Read Committed

   能够避免脏读。

3. RR，Repeatable Read

   能够避免不可重复读和脏读。

4. Serializable

   能够避免不可重复读、脏读、幻读。

##### MySQL 如何实现事务

[mvcc](../../../../2020/03/19/mvcc)

##### 事务的传播机制

Spring 在 TransactionDefinition 接口中定义了七个事务传播行为：

- **propagation_required**：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- **propagation_supports**：支持当前事务，如果没有当前事务，就以非事务方法执行。
- **propagation_mandatory**：使用当前事务，如果没有当前事务，就抛出异常。
- **propagation_required_new**：新建事务，如果当前存在事务，把当前事务挂起。
- **propagation_not_supported**：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **propagation_never**：以非事务方式执行操作，如果当前事务存在则抛出异常。
- **propagation_nested**：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作。

#### 分布式事务

在传统的架构体系中经常使用本地事务来保证数据一致性。但是随着分布式系统的发展，出现了越来越多的跨多个数据库进行操作的情况。在这种情况下，本地事务就无法保证数据一致性，分布式事务应运而生。分布式事务是针对本地事务来说的，它的目的就是解决在分布式环境下，跨多个数据库操作的数据一致性问题。分布式事务可分为刚性分布式事务和柔性分布式事务。


##### 刚性分布式事务

刚性分布式事务是和本地事务一样，都满足 ACID 特性的，是强一致性的。提到刚性事务，最据代表性的就是X/Open 提出的一种分布式事务模型，DTP (**D**istributed **T**ransaction **P**rocessing Reference Model) 模型。

![http://www.ibm.com/developerworks/websphere/library/techarticles/0407_woolf/images/DTPModel.gif](http://www.ibm.com/developerworks/websphere/library/techarticles/0407_woolf/images/DTPModel.gif)

DTP 模型有 3 个组成部分：

​	**AP**（Application Program）：即应用程序，定义事务边界

​    **RM**（Resource Manager）：即 RDBMS ，管理计算机共享资源

​	**TM**（Transation Manager）：负责全局事务，分配事务唯一id，监控事务的执行进度，负责事务的提交，回滚，失败恢复

TM 和 AP、TM 和 RM 之间通信遵守 XA 规范 (XA Specification) ，AP 和 RM 之间通过 Native API 通信。

##### 两阶段提交

2PC（two phase commit）是基于 DTP 模型的一种实现。
<img src="https://s2.ax1x.com/2020/03/11/8AVSxA.png" alt="8AVSxA.png" border="0" style="width:400px"/>
2PC 过程如下：

​	第一阶段，AP 发起事务 commit ，TM 发起 prepare 投票，当 RM 都同意后，进行第二阶段。

​    第二阶段，TM 最终执行 commit 。如果 commit 过程出现异常，则根据 recover 进行补偿。

刚性分布式事务的优点是强一致性，但是缺点也很明显：

1. **阻塞问题**：由于要保证数据一致性，那么 TM 要挨个询问 RM 收集投票结果，如果 RM 数量很多，那么在所有 RM 投票完成之前，RM 的资源都是被锁定的，所以会导致全局的资源锁定，处理的性能及其低下。
2. **TM 单点问题**：如果 TM 宕机，则无法协调 RM 进行commit 或 abort，导致资源长期锁定造成阻塞。
3. **数据一致性问题**：在 commit 阶段，如果发生网络分区，会导致一部分 RM 收到 commit ，一部分 RM 无法收到 commit ，导致数据不一致。
4. **事务状态不确定问题**：在 TM 发出 commit 消息之后，TM 和 RM 同时宕机，无法确定事物状态。
5. **容错性差**：在 prepare 过程中如果发生网络抖动，则会影响整个事务执行不成功。

##### 三阶段提交

2PC 最大的问题是阻塞，为了解决 2PC 的阻塞问题，引入了 3PC（Three phase commit）。3PC 在 prepare 前增加了 CanCommit 。

1. **CanCommit 阶段**，TM 会向 RM 发送 CanCommit 消息，询问 RM 是否可以提交事务，如果 RM 可以提交，返回 yes ，否则返回 no 。这个阶段不会真正执行事务操作。

2. **PreCommit 阶段**，如果所有的 TM 都返回 yes ，则  TM 向 RM 发送 PreCommit 请求。RM 在收到 PreCommit 请求后开始执行事务操作，并返回 ack 。如果有 TM 在 CanCommit 阶段返回 no ，则 TM 向 RM 发送 abort 请求中断事务。

3. **DoCommit 阶段**，TM 向 RM 发送 DoCommit 请求，RM 提交事务，完成后向 TM 发送 ack 。如果 TM 没有收到 ack ，则 TM 会向 RM 发送 abort 请求，终止事务。如果 RM 在超时时间内没有收到 TM 的消息，则会提交事务。

3PC 主要针对 2PC 的阻塞问题提出的，并不能解决数据一致性问题。

##### 柔性分布式事务

为了提高可用性，出现了柔性分布式事务。柔性分布式事务和刚性分布式事务不同，其理论基础是 BASE 理论。BASE 是 **B**asically **A**vailable（基本可用），**S**oft state（软状态）和 **E**ventually consistent（最终一致性）的缩写，是由 CAP 定理演化而来。

​	**基本可用**是指在特殊情况下，系统可以在功能和性能上保证基本或部分可用，比如系统响应时间从 10ms 降低到 500ms 或者只保证核心服务可用，非核心服务暂时不可用。

​	**软状态**是指在多个服务之间数据存在中间状态，多个服务之间暂时地数据不一致不会影响整体可用。

​	**最终一致性**是指数据经过短暂的不一致，最终能够达到一致状态。

通常柔性分布式事务有以下几种实现：

###### **TCC**

TCC 是 2PC 的一种变形，是 **T**ry-**C**onfirm-**C**ancel 的缩写。TCC 的执行过程和 2PC 类似，首先 try 阶段尝试执行业务，完成资源检查，预留资源。第二阶段，不用进行业务检查，直接进行 confirm ，执行成功，事务结束，如果 confirm 失败，则进行重试。如果在某一方的 try 失败了，则进行 cancel 来释放 try 阶段预留的资源。

我们通过一个简单的例子来感受一下。假设一个下订单-减库存-支付的场景，各业务方需要针对 TCC 进行改造，比如预留库存，在 try 阶段不能真正扣减库存，所以在数据库里需要增加一个预留库存的字段，再比如支付模块也需要增加字段来预存支付金额。如果 try 阶段预留资源都成功了，那么再将预留字段更新到实际扣减库存字段或扣减金额字段，并清空预留资源字段。一般经过 try 阶段的检查， confirm 基本上都能成功。如果 comfirm  不成功，可能是网络抖动，进行重试即可。如果支付模块在 try 阶段预减金额失败了，那么就进行 cancel ，库存模块按照 try 阶段预留的资源进行释放。通过举例，我们可以看出，TCC 很灵活，但是缺点是和业务耦合性高，因为 try-confirm-cancel 3 个阶段都交由业务方来实现。

###### **Saga**

Saga 是另一种著名分布式事务模型。1987 年论文 sagas 讲述了一种长事务的处理方案，即 saga 模型。Saga 模型的主要思想就是把一个分布式事务拆分成多个本地事务，每个 saga 子事务 ```T1,T2,...Tn``` 都有对应的补偿模块 ```C1,C2,...Cn-1``` 。当所有子事务都执行成功，那么它的执行顺序是 ```T1,T2,...Tn``` 。如果 Tj 执行失败了，那么它的执行顺序是 ```T1,T2,...Tj,Cj-1,...C2,C1,(0<j<n)``` 。
由于 saga 模型没有 try 阶段，所以当执行失败，需要通过补偿模块进行恢复。Saga 定义了两种恢复方式：向前恢复和向后恢复。向前恢复就是认为事务一定会执行成功而进行重试。向后恢复就是执行回滚+补偿，实际当中使用较多的是向后恢复。
我们还是以下单-减库存-支付这一场景来进行说明。首先我们需要定义两张事务表，事务状态表和事务调用表，这两张表和业务 DB 是独立的。

事务状态表

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr></table>
事务调用表

<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>method</td><td>param_type</td><td>param_value</td></tr></table>
saga 执行过程如下：

1.AP 发起事务后 TM 生成 txId， 向事务状态表存一条记录，状态是执行中 。

<table class="table table-bordered" style="width: 380px"><tr><td>tx_id</td><td style="width: 75px">state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>执行中</td><td>0</td><td>1514736000</td></tr></table>
2.AP 按顺序调用下单-减库存-支付操作，每调用一个操作之前向事务调用表插一条记录。

<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>recover_method</td><td>param_type</td><td>param_value</td></tr><tr><td>1</td><td>1</td><td>/recover_order</td><td>kv</td><td>orderid=12345</td></tr><tr><td>1</td><td>2</td><td>/recover_stock</td><td>kv</td><td>stock=1</td></tr><tr><td>1</td><td>3</td><td>/recover_payment</td><td>kv</td><td>pay=1000</td></tr></table>
这些信息如何获取到呢，又是怎么写到表里的呢？ 假如我们有个方法 reduceStock() ，我们在调用方法的时候可以利用 AOP 或者动态代理，就可以获取到方法的参数类型和参数值。recoverMethod 的信息可以通过注解写在被调用的方法上，这样通过反射就可以获取到 recoverMethod 的信息了。得到这些信息之后，按位置插到事务调用表中就可以了。

3.如果下单-减库存-支付都执行成功，TM 将事务状态表中的记录更新成成功，事务结束。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>成功</td><td>0</td><td>1514737000</td></tr></table>
4.如果有一步执行失败，则将事务状态更新成失败，同时立刻通知客户端执行失败。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>失败</td><td>2</td><td>1514737000</td></tr></table>
补偿是异步的，所以不用等补偿执行完成再通知客户端。

5.TM 定时扫描事务状态表，如果有失败状态的记录，就按照事务调用表中对应的除最后一步调用记录之外的其他记录调补偿接口进行补偿。

<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>recover_method</td><td>param_type</td><td>param_value</td></tr><tr><td>1</td><td>1</td><td>/recover_order</td><td>kv</td><td>orderid=12345</td></tr><tr><td>1</td><td>2</td><td>/recover_stock</td><td>kv</td><td>stock=1</td></tr><tr><td>1</td><td>3</td><td>/recover_payment</td><td>kv</td><td>pay=1000</td></tr></table>
如果表中有 3 条记录，说明前两条是执行成功的，第 3 条执行失败了。那么只需要执行前两步的补偿，第 3 步是不需要补偿的。那么接下来就按照事务调用表中记录的补偿接口的信息进行补偿。

1. 首先将事务状态置为 3 (补偿中) 。
2. 第二步调用 ```/recover_stock?stock=1``` 补偿库存，然后将 recover_step 改为 1，表示第二步补偿完成。
3. 第三步调用 ```/recover_order?orderid=12345&state=2``` 补偿订单，成功后将 recover_step 置为 2。

补偿接口应该设计成幂等的，这样可以保证多次重试也不会产生垃圾数据。

6.补偿完成后将事务状态表的状态更新成补偿完成。

<table class="table table-bordered" style="width: 360px"><tr><td>tx_id</td><td style="width: 90px">state</td><td>timestamp</td></tr><tr><td>1</td><td>补偿完成</td><td>1514739000</td></tr></table>
如果补偿接口出现问题，怎么办呢，我们需不需要再给补偿接口加一个分布式事务呢？一般情况下，经过测试并且有重试机制，补偿是可以成功。我们完全没有必要再加一个分布式事务来保证补偿，因为我们一旦给补偿加上分布式事务，那我们是不是也要对保证补偿的逻辑再加一个分布式事务来保证一致性呢，这样就无穷无尽了。所以简单的做法就是一旦真的补偿接口出错了，那么记录错误日志，告警，然后人工处理就好了。

###### **异步消息**

Saga 是一种同步串行的方式，接下来我们介绍异步消息的分布式事务实现。说到异步消息，自然少不了消息中间件。通过 MQ 进行消息传递，就需要有一套机制来保证消息可靠。
第一种方式是通过异步消息方式来实现：

1. 先发送一个 prepare 消息给 MQ Server 。 
2. MQ Server 收到消息返回一个 ack 。
3. 执行本地事务。
4. 本地事务成功向 MQ Server 发送一个 commit 消息。本地事务失败则向 MQ Server 发送一个 cancel 消息。
5. 如果长时间没有收到 prepare 消息的确认，MQ Server 则需要向 Client 申请回查。
6. Client 收到回查申请后，调用本地服务的回查接口查看本地事务是否成功，如果成功，发送 commit 消息，如果本地消息失败，则发送 cancel 消息。

以上就是异步消息的操作步骤，这种方式其实也是 2PC 的变形。这种方式实现起来对 MQ 的要求较高，并且需要业务方提供回查接口，对业务入侵较大。

另一种方式是通过本地消息表来实现：

1. 执行本地事务同时将消息先写到本地消息表中，由于执行本地操作和写消息在同一个事务中，所以可以保证同时成功或失败。
2. 将消息从表中读出来，写到 MQ 。
3. MQ 收到消息，返回 confirm 。
4. 收到 confirm 后删除本地消息。

这种方式对业务没有入侵并且实现简单，但是其中有一些细节需要注意。如果从消息表读出消息的服务部署了多个，那么都从消息表去读，就会产生大量的重复消息，所以可以使用分布式锁进行控制，获得锁的服务才能从消息表读，这样就可以避免重复消息。由于消息可能会产生重复，所以在消费端需要处理幂等。


通过上边的讲解，我们对分布式事务的几种实现方式有了简单的认识。在实际使用中，我们其实应该避免出现分布式事务，尽量让核心步骤先执行，不重要的步骤异步执行。如果实在无法避免，那么可以通过柔性分布式事务来处理，同步场景下使用 Saga ，异步场景下使用本地消息表。

**总结**

分布式事务不论是原理还是实现都比较复杂，且性能不高，一般除了直接和钱打交道的系统，没有必要上分布式事务。如果非要用分布式事务，可使用柔性事务或最终一致性方案。



### RPC 的实现

关于 RPC ，最近在知乎上看了一段文字，描述的比较透彻 [https://www.zhihu.com/question/25536695/answer/221638079](https://www.zhihu.com/question/25536695/answer/221638079) 。

本地过程调用 RPC 就是要像调用本地的函数一样去调远程函数。在研究 RPC 前，我们先看看本地调用是怎么调的。假设我们要调用函数 Multiply 来计算 lvalue * rvalue 的结果:

```
1. int Multiply(int l, int r) {
2.   int y = l * r;
3.   return y;
4.}
5. 
6. int lvalue = 10;
7. int rvalue = 20;
8. int l_times_r = Multiply(lvalue, rvalue);
```

那么在第 8 行时，我们实际上执行了以下操作：将 lvalue 和 rvalue 的值压栈进入 Multiply 函数，取出栈中的值10 和 20，将其赋予 l 和 r 执行第 2 行代码，计算 l * r ，并将结果存在 y 将 y 的值压栈，然后从 Multiply 返回第 8 行，从栈中取出返回值 200 ，并赋值给 l_times_r 以上 5 步就是执行本地调用的过程。

在远程调用时，我们需要执行的函数体是在远程的机器上的，也就是说，Multiply 是在另一个进程中执行的。这就带来了几个新问题：

* Call ID映射
  我们怎么告诉远程机器我们要调用 Multiply，而不是 Add 或者 FooBar 呢？在本地调用中，函数体是直接通过函数指针来指定的，我们调用 Multiply，编译器就自动帮我们调用它相应的函数指针。但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。所以，在RPC中，所有的函数都必须有自己的一个 ID 。这个 ID 在所有进程中都是唯一确定的。Call ID 映射可以直接使用函数字符串，也可以使用整数 ID 。客户端在做远程过程调用时，必须附上这个 ID 。然后我们还需要在客户端和服务端分别维护一个 ```{函数 <--> Call ID}``` 的对应表。映射表一般就是一个哈希表。两者的表不一定需要完全相同，但相同的函数对应的 Call ID 必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的 Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。
* 序列化和反序列化
  客户端怎么把参数值传给远程的函数呢？在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用 C++ ，客户端用 Java 或者 Python ）。这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。序列化反序列化可以自己写，也可以使用 Protobuf 或者 FlatBuffers 之类的。
* 网络传输
  远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把 Call ID 和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分 RPC 框架都使用 TCP 协议，但其实 UDP 也可以，而 gRPC 干脆就用了 HTTP2 。当然也可以自己写 socket 或者用 asio，ZeroMQ，Netty 之类。

所以，要实现一个 RPC 框架，其实只需要把以上三点实现了就基本完成了。

### Database

[MySQL in Action](../../../../2099/05/05/mysql-in-action/)

### MQ

[MQ](../../../../2021/09/03/mq/)

### Cache

[Cache](../../../../2021/09/17/cache/)

### ES

[Elastic Learning Path](../../../../2018/05/03/elastic-learning-path/)

[One Question One Answer for ES](../../../../2021/03/03/one-question-one-answer-for-es/)

