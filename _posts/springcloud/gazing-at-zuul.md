---
title: Gazing at Zuul
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2020-03-24 22:20:58
thumbnail:
---

ZuulServlet 在 service 中顺序执行 preRoute() ，route() ，postRoute() 。具体的 route 动作是通过聚合 ZuulRunner ，在 ZuulRunner 中使用 FilterProcessor 来执行。
首先执行的是 pre 类型的 filter 。默认的 preFilter 有 5 个：
1. ServletDetectionFilter ，第一个 preFilter ，order 是 -3 ，用来判断 isDispatcherServletRequest 。
2. Servlet30WrapperFilter ，第二个 preFilter ，order 是 -2 ，用来包装 request 请求。
3. FormBodyWrapperFilter ，第三个 preFilter ，order 是 -1 ，用来对请求进行 FormBodyRequestWrapper 包装，以便后续的 filter 能够拿到流的信息。
4. DebugFilter ，第四个 preFilter ，order 是 1 ，用于 debug 开启时设置 debug 开关，打印 debug 信息。开启 bebug 之后，debug 信息会放在 X-Zuul-Debug-Header 中。
5. PreDecorationFilter ，第五个 preFilter ，order 是 5 ，用来做一些预处理，比如 set requestURI ，比如添加一些 request header 信息。


然后会执行 route 类型的 filter 。默认的 routeFilter 有 3 个：
1. RibbonRoutingFilter ，第一个 routeFilter ，order 是 10 ，针对通过 serviceId 进行路由的请求。
2. SimpleHostRoutingFilter ，第二个 routeFilter ，order 是 100 ，针对通过 url 进行路由的请求。
3. SendForwardFilter ，第三个 routeFilter ，order 是 500 ，用来处理 request.getRequestDispatcher() 的请求。

最后执行 post 类型的 filter 。默认的 postFilter 有 1 个：
1. SendResponseFilter ，第一个 postFilter ，order 是 1000 ，当头信息不为空，或有响应流信息，或有响应体的时候对 response 进行处理，将数据发回客户端。

正常情况下不会执行 error 类型的 filter 。

>filter 的 order 表示 filter 的执行顺序，越小越早执行。需要注意，如果顺序错了，可能导致结果有问题，比如我们如果要在 preFilter 中重写 url ，应该放在 PreDecorationFilter 之后，因为 PreDecorationFilter 中会 set request 信息，如果我们在此之前 rewrite 了 url ，在这一步又会被覆盖成原始的。