---
title: Building Eureka Server with Region and Zone
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2020-03-22 19:01:34
thumbnail:
---

Eureka Region 和 Zone 的架构参考 [Eureka Region and Zone](../../../../2018/08/29/eureka-region-and-zone/) 。

按照 Region 和 Zone 的架构来构建，有一个 Region us-east-1 ，us-east-1 下有 3 个 Zone ：us-east-1c ，us-east-1d ，us-east-1e 。各个 Zone 下有一个 Eureka Server 和 Application Service ，另外在 us-east-1c 下还有一个 Application Client 。Application Client 和 Application Service 都会注册到 Eureka Server 上，Application Client 会调用 Application Service 。当服务正常时，Application Client 会优先调用相同 us-east-1c 下的 Application Service ，当 us-east-1c 下的 Application Service 下线，Application Client 会调用 us-east-1d 和 us-east-1e 中的 Application Service 。
配置文件比之前多了 Region 和 Zone 的信息。
eureka-server-01
```
server:
  port: 8761
spring:
  application:
    name: eureka-server-01
eureka:
  instance:
    hostname: ${spring.application.name}
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就失效
    metadata-map:
      zone: us-east-1c #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1c,us-east-1d,us-east-1e #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 1000 #剔除时间间隔
```
eureka-server-02
```
server:
  port: 8762
spring:
  application:
    name: eureka-server-02
eureka:
  instance:
    hostname: ${spring.application.name}
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就失效
    metadata-map:
      zone: us-east-1d #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1d,us-east-1c,us-east-1e #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 1000 #剔除时间间隔
```
eureka-server-03
```
server:
  port: 8763
spring:
  application:
    name: eureka-server-03
eureka:
  instance:
    hostname: ${spring.application.name}
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就失效
    metadata-map:
      zone: us-east-1e #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1e,us-east-1c,us-east-1d #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 1000 #剔除时间间隔
```
application-service-01
```
server:
  port: 10003
spring:
  application:
    name: application-service
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就剔除服务
    metadata-map:
      zone: us-east-1c #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1c,us-east-1d,us-east-1e #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
```
application-service-02
```
server:
  port: 10004
spring:
  application:
    name: application-service
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就剔除服务
    metadata-map:
      zone: us-east-1d #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1d,us-east-1c,us-east-1e #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
```
application-service-03
```
server:
  port: 10005
spring:
  application:
    name: application-service
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就剔除服务
    metadata-map:
      zone: us-east-1e #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1e,us-east-1c,us-east-1d #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
```
application-client-01
```
server:
  port: 8081
spring:
  application:
    name: application-client
eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    lease-renewal-interval-in-seconds: 1 #心跳间隔
    lease-expiration-duration-in-seconds: 3 #3秒没收到心跳就剔除服务
    metadata-map:
      zone: us-east-1c #服务属于哪个zone
  client:
    region: us-east-1
    availability-zones:
      us-east-1: us-east-1c,us-east-1d,us-east-1e #将自己的zone写在前边
    serviceUrl:
      us-east-1c: http://eureka-server-01:8761/eureka/
      us-east-1d: http://eureka-server-02:8762/eureka/
      us-east-1e: http://eureka-server-03:8763/eureka/
    prefer-same-zone-eureka: true #优先使用相同zone的服务
```

通过 metadata-map.zone 标识服务属于哪个 Zone ，```prefer-same-zone-eureka: true``` 会优先调用相同 Zone 的服务。availability-zones 的 Zone 列表应优先将自己 Zone 写在前边，因为 Eureka 在查找服务时是按顺序找的。