---
title: Eureka Data Structure
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-08-29 17:06:40
thumbnail:
---



在 Eureka Server 中，使用了双层 ConcurrentHashMap 来存储 Eureka Client 的注册信息。

```java
private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
            = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
```

第一层 Map 的 Key 是 Eureka Client 的 ``spring.application.name`` ，Value 是 ConcurrentHashMap 。第二层 Map 的 Key 是 Client 的 instanceId ，Value 是一个 Lease 对象，对象存放 InstanceInfo 和 Client 的注册时间，剔除时间，续约时间，持续时长等信息。InstanceInfo 包含的信息如图：

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180902/112829711.png" style="width:240px;" />

详情可参考 ``com.netflix.eureka.registry.AbstractInstanceRegistry`` 中的 ```register()``` 方法。

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20180902/185301822.png" style="width:600px;" />