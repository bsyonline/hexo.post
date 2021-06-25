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



关闭服务检查

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

版本

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

服务分组

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

多协议

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

负载均衡

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

