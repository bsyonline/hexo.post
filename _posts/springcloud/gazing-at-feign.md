---
title: Gazing at Feign
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2020-03-24 14:39:23
thumbnail:
---

Feign 的工作流程：
1. 启动时 ReflectiveFeign 通过 jdk 动态代理方式生成 FeignClient 的代理。
<img src="https://s1.ax1x.com/2020/03/24/8qynjP.png" alt="8qynjP.png" border="0" style="width:700px"/>
2. 调用 invoke() 时，生成 RequestTemplate 封装了 uri ，请求参数等信息。
<img src="https://s1.ax1x.com/2020/03/24/8qw3tI.png" alt="20200324150653" border="0" style="width:600px">
3. 在 executeAndDecode() 中用 RequestTemplate 构造出 Request 对象。
<img src="https://s1.ax1x.com/2020/03/24/8qwnXD.png" alt="20200324150745" border="0" style="width:700px">
4. 调用 LoadBalancerFeignClient.execute() 去发送 http 请求。
<img src="https://s1.ax1x.com/2020/03/24/8qwQ7d.png" alt="20200324151551" border="0" style="width:700px">
5. 获得 response 之后，根据 code 来进行 decode 将报文组装成对象。
<img src="https://s1.ax1x.com/2020/03/24/8qwm6O.png" alt="20200324151750" border="0" style="width:650px">