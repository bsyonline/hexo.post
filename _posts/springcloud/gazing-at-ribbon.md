---
title: Gazing at Ribbon
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2020-03-23 21:58:31
thumbnail:
---

通过自动装配扫描 spring.factories 中的 LoadBalancerAutoConfiguration 用来构造 LoadBalancerInterceptor 实例。在进行 http 调用的时候进行拦截。如果 server 有多个，会根据 IRule 来选择 server 。默认使用 aroundRobin ，server 存在 ArrayList 中，有一个计数器记录 nextIndex，用 nextIndex 对 server 数量取余来获取 server 的 index 。拿到 server 的 ip 和 port后，在 RibbonLoadBalancerClient 的 reconstructURI 中将服务名替换成 ip 和 port ，构造成新的 url ，通过 LoadBalancerRequest.apply() 发起真正的 http 调用。

Ribbon 在 RibbonClientConfiguration 中进行了默认配置。有 3 个配置：
1. IRule ，负载均衡的规则，默认 ZoneAvoidanceRule 。获取所有可用的 zone ，如果当前 zone 可用，就使用当前 zone 中的 server ，不再看其他的 zone 。如果 server 有多个默认使用 aroundRobin 来选择 server 。
2. IPing ，向服务发起 Ping 操作，默认 DummyPing 什么都不做。
3. ILoadBalancer ，使用 IRule 和 IPing 配置进行 loadbalance 。默认使用 ZoneAwareLoadBalancer 。