---
title: Dubbo in Action
tags:
  - Microservices
  - Dubbo
category:
  - Dubbo
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:06:40
thumbnail:
---



### Dubbo 简介



#### 关闭服务检查

默认会检查服务提供者状态，如果服务提供者没有启动，消费者在启动时会报错。这就需要服务提供者先于消费者启动，但是有时候不清楚服务的调用顺序，又或者服务之间存在循环调用，那么就需要关闭服务检查。

关闭服务检查的几种写法：

yml

```
dubbo:
  consumer:
    check: false
```

xml

```
<dubbo:consumer check="false" />
```

properties

```
dubbo.consumer.check=false
```

#### 版本

灰度发布时会将一部分服务升级成新版本，在调用的时可以通过 version 来控制调用哪个版本的服务。

服务提供者

```java
@DubboService(version = "1.0.0")
public class DemoServiceImpl implements DemoService {}
```

服务消费者

```java
@DubboReference(version = "2.0.0")
private DemoService demoService;
```

#### 服务分组

当一个接口有多种实现时，可用使用 group 分组。

```java
@DubboService(group = "chinese", version = "1.0.0")
public class ChineseGreetingServiceImpl implements GreetingService {
}
```

```
@DubboService(group = "english", version = "1.0.0")
public class EnglishGreetingServiceImpl implements GreetingService {
}
```

```java
@DubboReference(version = "1.0.0", group = "english")
private GreetingService englishGreetingService;
englishGreetingService.greeting();

@DubboReference(version = "1.0.0", group = "chinese")
private GreetingService chineseGreetingService;
chineseGreetingService.greeting();
```

group 还支持聚合。

返回所有组。

```java
@DubboReference(version = "1.0.0", group = "*", merger = "true")
```

返回指定组。

```
@DubboReference(version = "1.0.0", group = "chinese,english", merger = "true")
```

> 注意，接口返回需要为集合类型。

#### 多协议

dubbo 支持 9 种服务暴露协议。

1. dubbo

   Dubbo 的缺省协议。采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。不适合传送大数据量的服务，比如传文件，传视频等。

   默认使用Hessian 二进制序列化。

2. rmi

   RMI 协议采用 JDK 标准的 `java.rmi.*` 实现，采用阻塞式短连接和 JDK 标准序列化方式。适合传入传出参数数据包大小混合，消费者与提供者个数差不多。可传文件。

3. hessian

   采用 Http 通讯，可以和原生 Hessian 服务互操作。

4. webservice

   基于 WebService 的远程调用协议，可以和原生 WebService 服务互操作。

5. http

   可用表单或URL传入参数，适用于需同时给应用程序和浏览器 JS 使用的场景。

6. rest

   基于标准的 Java REST API——JAX-RS 2.0（Java API for RESTful Web Services的简写）实现的 REST 调用支持。

7. thrift

8. redis

9. memcached

10. grpc

#### 负载均衡

1. random

   默认。

2. roundrobin

   按照权重依此进行调度。

   provider.xml

   ```
   <dubbo:service interface="com.rolex.tips.api.DemoService" ref="demoService" weight="1"/>
   ```

3. leastactive

   最少活跃度，被调用的次数越少，优先级越高。

4. consistenthash

   相同参数的请求被路由到相同提供者。默认对第一个参数进行hash。

   ```
   
   ```

   

负载均衡策略可以在 consumer 和 provider 设置。如果两边都设置了，comsumer 优先级更高。

可以对服务设置也可以对方法设置，方法优先级高于服务。

```
<dubbo:service interface="helloService" loadbalance="roundrobin">
　　<dubbo:method name="hello" loadbalance="leastactive"/>
</dubbo:service>
```



```
dubbo:
  consumer:
    loadbalance: roundrobin
```



```
@DubboReference(version = "1.0.0", loadbalance = "roundrobin")
private DemoService demoService;
```

#### 容错策略

1. Failover

   调用失败会自动尝试调用其他 provider 。

   ```java
   @DubboReference(version = "1.0.0", cluster = "failover")
   private DemoService demoService;
   ```

2. Failfast

   失败立即报错。常用于非幂等请求。

3. Failsafe

   出现异常则或略本次调用。常用与不重要的服务。

4. Failback

   调用失败后，记录失败请求，定时重试。常用于实时性要求不高的服务。

5. Forking

   并行调用多个 provider ，只要一个成功就调用结束并返回。

6. Broadcast

   广播调用所有 provider ，有一个报错则本次调用失败。

#### 服务降级

Dubbo 服务降级是通过 mock 实现的，有两种方式：

mock value

```java
@DubboReference(version = "1.0.0", mock = "return 11")
private DemoService demoService;
```

适合返回简单value的场景，但是没有返回值的接口不会有任何返回。

mock class

在接口相同包名下创建mock类，名字以 Mock 结尾。

```java
public class DemoServiceMock implements DemoService {
    @Override
    public String sayHello(String name) {
        return "mock hello";
    }
}
```

打开 mock 开关

```yml
dubbo:
  consumer:
    mock: true
```

#### 限流

Dubbo 的限流可以分为两类：直接限流和间接限流。

直接限流

通过对连接数量的限制达到限流的目的。

1. executes 限流

   提供者端。服务并发数量。

   ```java
   @DubboService(version = "1.0.0", executes = 10)
   public class DemoServiceImpl implements DemoService {
   }
   ```

   

2. accepts 限流

   提供者端。对协议连接数量的限制。

   ```
   <dubbo:protocol name="dubbo" port="20890" accepts="10"/>
   ```

   

3. actives 限流

   提供者端。

   长连接表示当前长连接最多可以处理的请求个数。与长连接的数量无关。

   短链接表示当前服务可以同时处理的短链接的数量。

   ```java
   @DubboService(version = "1.0.0", actives = 10)
   public class DemoServiceImpl implements DemoService {
   }
   ```

   消费者端。

   长连接表示当前消费者发出的长连接种最多可以提交的请求个数。与长连接的数量无关。

   短连接表示当前消费者可以提交的短连接数量。

   ```java
   @DubboReference(version = "1.0.0", actives = 10)
   private DemoService demoService;
   ```

   

4. connections 限流

   限制连接的个数。

   消费者端

   ```java
   @DubboReference(version = "1.0.0", connections = 10)
   private DemoService demoService;
   ```

   提供者端

   ```java
   @DubboService(version = "1.0.0", connections = 10)
   public class DemoServiceImpl implements DemoService {
   }
   ```

   

间接限流

1. 延迟调用

   不调用不建立连接。消费者端。只能用 dubbo 协议。

   ```java
   @DubboReference(version = "1.0.0", lazy = true)
   private DemoService demoService;
   ```

   

2. 粘连连接

   客户端尽量使用同一个提供者发起调用，除非该提供者宕机。可以控制请求流向。消费者端。只能用 dubbo 协议。

   ```java
   @DubboReference(version = "1.0.0", sticky = true)
   private DemoService demoService;
   ```

   

3. 负载均衡

   使用 leastactive 策略，可以控制请求流向。消费者端和提供者端。

#### 声明式缓存

缓存在消费者端。根据 LRU 缓存最近 1000 个。

#### 多注册中心

```
dubbo:
  # 多注册中心
  registries:
    beijing:
      register: false #默认true, false:表示服务不注册到注册中心
      protocol: zookeeper
      address: zookeeper://127.0.0.1:2181
      port: 2181
    shanghai:
      register: false #默认true, false:表示服务不注册到注册中心
      protocol: zookeeper
      address: zookeeper://127.0.0.1:2181
      port: 2181
```

服务注册

提供者端

```
@DubboService(version = "1.0.0", registry = {"beijing, shanghai"})
public class DemoServiceImpl implements DemoService {
}
```

消费者端

```
@DubboReference(version = "1.0.0", registry = {"beijing", "shanghai"})
private DemoService demoService;
```

#### 单功能注册中心

1. 仅订阅

   可以发现服务，不能注册服务。开发联调时常用。

   ```java
   @DubboService(version = "1.0.0", register = false)
   public class DemoServiceImpl implements DemoService {
       
   }
   ```

2. 仅注册

   可以被发现，不能下载注册列表。

#### 服务暴露延迟

```java
@DubboService(version = "1.0.0", delay = 500)
public class DemoServiceImpl implements DemoService {
}
```

正数：单位毫秒，提供之对象创建完毕后的指定时间后再发布服务。

0：当前提供者创建完成后，立刻向注册中心暴露服务。

-1：spring 容器初始化完毕后再向注册中心暴露服务。

#### 消费者异步调用

Future 是 Dubbo 2.7 之前的异步调用实现方式，对异步结果的获取采用阻塞和轮询的方式。

```
String fooResult1 = demoService.foo();
System.out.println("fooResult1=" + fooResult1);
Future<String> fooFuture = RpcContext.getContext().getFuture();

String barResult1 = demoService.bar();
System.out.println("barResult1=" + barResult1);
Future<String> barFuture = RpcContext.getContext().getFuture();

String fooResult2 = fooFuture.get(); //阻塞
System.out.println("fooResult2=" + fooResult2);

String barResult2 = barFuture.get(); //阻塞
System.out.println("barResult2=" + barResult2);
```

CompletableFuture 是 Dubbo 2.7 之后的异步调用实现方式。

```java
long start = System.currentTimeMillis();
CompletableFuture<String> fooCompletableFuture = demoService.fooCompletableFuture();
CompletableFuture<String> barCompletableFuture = demoService.barCompletableFuture();
fooCompletableFuture.whenComplete(new BiConsumer<String, Throwable>() {
    @Override
    public void accept(String s, Throwable throwable) {
        if (throwable != null) {
            throwable.printStackTrace();
        } else {
            System.out.println("fooResult=" + s);
        }
    }
});

barCompletableFuture.whenComplete((result, throwable) -> {
    if (throwable != null) {
        throwable.printStackTrace();
    } else {
        System.out.println("barResult=" + result);
    }
});

// 不阻塞
System.out.println("执行完成，耗时 " + ((System.currentTimeMillis() - start)) + " ms");
```

#### 属性配置优先级

方法级最高，服务级次之，全局最低。消费者端优先，提供者端次之。