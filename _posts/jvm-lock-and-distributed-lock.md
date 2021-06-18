---
title: JVM Lock and Distributed Lock
tags:
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-06-14 08:12:17
thumbnail:
---





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

##### 基于 redis 实现分布式锁

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

##### 基于 zookeeper 实现分布式锁







##### 基于 etcd 实现分布式锁







