---
title: Building Eureka Server with Peer Awareness
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2020-03-22 09:22:03
thumbnail:
---

构建一个 Eureka Server Standalone 非常简单，可以参考 [Service Registration and Discovery](../../../../2018/06/16/service-registration-and-discovery/) 。
Eureka Server 也支持部署多个节点，步骤和单节点类似，只是配置文件略有修改。如果我们有 3 个对等的节点，配置文件是这样的：
eureka01
```
server:
  port: 8761
spring:
  application:
    name: eureka01
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://eureka01:8761/eureka/,http://eureka02:8762/eureka/,http://eureka03:8763/eureka/
  server:
    enable-self-preservation: false
```
eureka02
```
server:
  port: 8762
spring:
  application:
    name: eureka02
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://eureka01:8761/eureka/,http://eureka02:8762/eureka/,http://eureka03:8763/eureka/
```
eureka03
```
server:
  port: 8763
spring:
  application:
    name: eureka03
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://eureka01:8761/eureka/,http://eureka02:8762/eureka/,http://eureka03:8763/eureka/
```

Eureka Server 之间是对等的，所以可以将自己作为 Client 注册到其他的 Eureka Server 上，defaultZone 可以不用区分，启动的时候会过滤掉自己。