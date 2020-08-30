---
title: Distributed Transations
tags:
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-08-03 22:32:48
thumbnail: 

---

我们通常说的事务是指本地事务，即对一个数据库进行的操作，并且这些操作满足事务的 4 个特性。事务的 4 个特性分别是：
Aotmicity（原子性），Consistency（一致性），Isolation（隔离性），Durability（持久性）。在一个事务中的所有操作要么同时成功，要么同时失败，这是原子性，一旦执行成功 commit 或是执行失败 rollback 那么这个结果必须是持久的。在事务开始和结束之间，存在着中间状态，这些中间状态是对事务之外是隔离的，换句话说，在事务结束之前，对数据进行的任何修改对事务之外都是不可见的，这就是隔离性。在事务结束之后，数据必须是完整并且一致的。

在传统的架构体系中经常使用本地事务来保证数据一致性。但是随着分布式系统的发展，出现了越来越多的跨多个数据库进行操作的情况。在这种情况下，本地事务就无法保证数据一致性，分布式事务应运而生。分布式事务是针对本地事务来说的，它的目的就是解决在分布式环境下，跨多个数据库操作的数据一致性问题。分布式事务可分为刚性分布式事务和柔性分布式事务。


#### 刚性分布式事务

刚性分布式事务是和本地事务一样，都满足 ACID 特性的，是强一致性的。提到刚性事务，最据代表性的就是X/Open 提出的一种分布式事务模型，DTP (**D**istributed **T**ransaction **P**rocessing Reference Model) 模型。

![http://www.ibm.com/developerworks/websphere/library/techarticles/0407_woolf/images/DTPModel.gif](http://www.ibm.com/developerworks/websphere/library/techarticles/0407_woolf/images/DTPModel.gif)

DTP 模型有 3 个组成部分：

​	**AP**（Application Program）：即应用程序，定义事务边界

​    **RM**（Resource Manager）：即 RDBMS ，管理计算机共享资源

​	**TM**（Transation Manager）：负责全局事务，分配事务唯一id，监控事务的执行进度，负责事务的提交，回滚，失败恢复

TM 和 AP、TM 和 RM 之间通信遵守 XA 规范 (XA Specification) ，AP 和 RM 之间通过 Native API 通信。

#### 两阶段提交

2PC（two phase commit）是基于 DTP 模型的一种实现。
<img src="https://s2.ax1x.com/2020/03/11/8AVSxA.png" alt="8AVSxA.png" border="0" style="width:400px"/>
2PC 过程如下：

​	第一阶段，AP 发起事务 commit ，TM 发起 prepare 投票，当 RM 都同意后，进行第二阶段。

​    第二阶段，TM 最终执行 commit 。如果 commit 过程出现异常，则根据 recover 进行补偿。

刚性分布式事务的优点是强一致性，但是缺点也很明显，由于要保证数据一致性，那么 TM 要挨个询问 RM 收集投票结果，如果 RM 数量很多，那么在所有 RM 投票完成之前，RM 的资源都是被锁定的，所以会导致全局的资源锁定，处理的性能及其低下。

#### 柔性分布式事务

为了提高可用性，出现了柔性分布式事务。柔性分布式事务和刚性分布式事务不同，其理论基础是 BASE 理论。BASE 是 **B**asically **A**vailable（基本可用），**S**oft state（软状态）和 **E**ventually consistent（最终一致性）的缩写，是由 CAP 定理演化而来。

​	**基本可用**是指在特殊情况下，系统可以在功能和性能上保证基本或部分可用，比如系统响应时间从 10ms 降低到 500ms 或者只保证核心服务可用，非核心服务暂时不可用。

​	**软状态**是指在多个服务之间数据存在中间状态，多个服务之间暂时地数据不一致不会影响整体可用。

​	**最终一致性**是指数据经过短暂的不一致，最终能够达到一致状态。

通常柔性分布式事务有以下几种实现：

##### **TCC**

TCC 是 2PC 的一种变形，是 **T**ry-**C**onfirm-**C**ancel 的缩写。TCC 的执行过程和 2PC 类似，首先 try 阶段尝试执行业务，完成资源检查，预留资源。第二阶段，不用进行业务检查，直接进行 confirm ，执行成功，事务结束，如果 confirm 失败，则进行重试。如果在某一方的 try 失败了，则进行 cancel 来释放 try 阶段预留的资源。

我们通过一个简单的例子来感受一下。假设一个下订单-减库存-支付的场景，各业务方需要针对 TCC 进行改造，比如预留库存，在 try 阶段不能真正扣减库存，所以在数据库里需要增加一个预留库存的字段，再比如支付模块也需要增加字段来预存支付金额。如果 try 阶段预留资源都成功了，那么再将预留字段更新到实际扣减库存字段或扣减金额字段，并清空预留资源字段。一般经过 try 阶段的检查， confirm 基本上都能成功。如果 comfirm  不成功，可能是网络抖动，进行重试即可。如果支付模块在 try 阶段预减金额失败了，那么就进行 cancel ，库存模块按照 try 阶段预留的资源进行释放。通过举例，我们可以看出，TCC 很灵活，但是缺点是和业务耦合性高，因为 try-confirm-cancel 3 个阶段都交由业务方来实现。

##### **Saga**

Saga 是另一种著名分布式事务模型。1987 年论文 sagas 讲述了一种长事务的处理方案，即 saga 模型。Saga 模型的主要思想就是把一个分布式事务拆分成多个本地事务，每个 saga 子事务 ```T1,T2,...Tn``` 都有对应的补偿模块 ```C1,C2,...Cn-1``` 。当所有子事务都执行成功，那么它的执行顺序是 ```T1,T2,...Tn``` 。如果 Tj 执行失败了，那么它的执行顺序是 ```T1,T2,...Tj,Cj-1,...C2,C1,(0<j<n)``` 。
由于 saga 模型没有 try 阶段，所以当执行失败，需要通过补偿模块进行恢复。Saga 定义了两种恢复方式：向前恢复和向后恢复。向前恢复就是认为事务一定会执行成功而进行重试。向后恢复就是执行回滚+补偿，实际当中使用较多的是向后恢复。
我们还是以下单-减库存-支付这一场景来进行说明。首先我们需要定义两张事务表，事务状态表和事务调用表，这两张表和业务 DB 是独立的。

事务状态表
<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr></table>
事务调用表
<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>method</td><td>param_type</td><td>param_value</td></tr></table>
saga 执行过程如下：

1.AP 发起事务后 TM 生成 txId， 向事务状态表存一条记录，状态是执行中 。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>0</td><td>0</td><td>1514736000</td></tr></table>
2.AP 按顺序调用下单-减库存-支付操作，每调用一个操作之前向事务调用表插一条记录。

<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>recover_method</td><td>param_type</td><td>param_value</td></tr><tr><td>1</td><td>1</td><td>/recover_order</td><td>kv</td><td>orderid=12345</td></tr><tr><td>1</td><td>2</td><td>/recover_stock</td><td>kv</td><td>stock=1</td></tr><tr><td>1</td><td>3</td><td>/recover_payment</td><td>kv</td><td>pay=1000</td></tr></table>
这些信息如何获取到呢，又是怎么写到表里的呢？ 假如我们有个方法 ```reduceStock()``` ，我们在调用方法的时候可以利用 AOP 或者动态代理，就可以获取到方法的参数类型和参数值。recoverMethod 的信息可以通过注解写在被调用的方法上，这样通过反射就可以获取到 recoverMethod 的信息了。得到这些信息之后，按位置插到事务调用表中就可以了。

3.如果下单-减库存-支付都执行成功，TM 将事务状态表中的记录更新成成功，事务结束。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>1</td><td>0</td><td>1514737000</td></tr></table>
4.如果有一步执行失败，则将事务状态更新成失败，同时立刻通知客户端执行失败。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>recover_step</td><td>timestamp</td></tr><tr><td>1</td><td>2</td><td>0</td><td>1514737000</td></tr></table>
补偿是异步的，所以不用等补偿执行完成再通知客户端。
5.TM 定时扫描事务状态表，如果有失败状态的记录，就按照事务调用表中对应的除最后一步调用记录之外的其他记录调补偿接口进行补偿。

<table class="table table-bordered" style="width: 100px"><tr><td>tx_id</td><td>action_id</td><td>recover_method</td><td>param_type</td><td>param_value</td></tr><tr><td>1</td><td>1</td><td>/recover_order</td><td>kv</td><td>orderid=12345</td></tr><tr><td>1</td><td>2</td><td>/recover_stock</td><td>kv</td><td>stock=1</td></tr><tr><td>1</td><td>3</td><td>/recover_payment</td><td>kv</td><td>pay=1000</td></tr></table>
如果表中有 3 条记录，说明前两条是执行成功的，第 3 条执行失败了。那么只需要执行前两步的补偿，第 3 步是不需要补偿的。那么接下来就按照事务调用表中记录的补偿接口的信息进行补偿。

1. 首先将事务状态置为 3 。
2. 第二步调用 ```/recover_stock?stock=1``` 补偿库存，然后将 recover_step 改为 2，表示第二步补偿完成。
3. 第三步调用 ```/recover_order?orderid=12345&state=2``` 补偿订单，成功后将 recover_step 置为 1。

补偿接口应该设计成幂等的，这样可以保证多次重试也不会产生垃圾数据。

6.补偿完成后将事务状态表的状态更新成补偿完成。

<table class="table table-bordered" style="width: 60px"><tr><td>tx_id</td><td>state</td><td>timestamp</td></tr><tr><td>1</td><td>4</td><td>1514739000</td></tr></table>
如果补偿接口出现问题，怎么办呢，我们需不需要再给补偿接口加一个分布式事务呢？一般情况下，经过测试并且有重试机制，补偿是可以成功。我们完全没有必要再加一个分布式事务来保证补偿，因为我们一旦给补偿加上分布式事务，那我们是不是也要对保证补偿的逻辑再加一个分布式事务来保证一致性呢，这样就无穷无尽了。所以简单的做法就是一旦真的补偿接口出错了，那么记录错误日志，告警，然后人工处理就好了。

##### **异步消息**

Saga 是一种同步串行的方式，接下来我们介绍异步消息的分布式事务实现。说到异步消息，自然少不了消息中间件。通过 MQ 进行消息传递，就需要有一套机制来保证消息可靠。
第一种方式是通过异步消息方式来实现：

1.先发送一个 prepare 消息给 MQ Server 。 
2.MQ Server 收到消息返回一个 ack 。
3.执行本地事务。
4.本地事务成功向 MQ Server 发送一个 commit 消息。本地事务失败则向 MQ Server 发送一个 cancel 消息。
5.如果长时间没有收到 prepare 消息的确认，MQ Server 则需要向 Client 申请回查。
6.Client 收到回查申请后，调用本地服务的回查接口查看本地事务是否成功，如果成功，发送 commit 消息，如果本地消息失败，则发送 cancel 消息。

以上就是异步消息的操作步骤，这种方式其实也是 2PC 的变形。这种方式实现起来对 MQ 的要求较高，并且需要业务方提供回查接口，对业务入侵较大。

另一种方式是通过本地消息表来实现：
1.执行本地事务同时将消息先写到本地消息表中，由于执行本地操作和写消息在同一个事务中，所以可以保证同时成功或失败。
2.将消息从表中读出来，写到 MQ 。
3.MQ 收到消息，返回 confirm 。
4.收到 confirm 后删除本地消息。

这种方式对业务没有入侵并且实现简单，但是其中有一些细节需要注意。如果从消息表读出消息的服务部署了多个，那么都从消息表去读，就会产生大量的重复消息，所以可以使用分布式锁进行控制，获得锁的服务才能从消息表读，这样就可以避免重复消息。由于消息可能会产生重复，所以在消费端需要处理幂等。


通过上边的讲解，我们对分布式事务的几种实现方式有了简单的认识。在实际使用中，我们其实应该避免出现分布式事务，尽量让核心步骤先执行，不重要的步骤异步执行。如果实在无法避免，那么可以通过柔性分布式事务来处理，同步场景下使用 Saga ，异步场景下使用本地消息表。

