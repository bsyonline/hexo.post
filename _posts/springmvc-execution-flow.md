---
title: springMVC 执行流程
tags:
  - Interview
category:
  - Spring
author: bsyonline
lede: 没有摘要
date: 2019-12-02 10:16:08
thumbnail:
---



<img src="https://s2.ax1x.com/2020/02/27/3dSjgK.png" alt="3dSjgK.png" border="0" style="width:600px" />

SpringMVC 执行流程：

1. 请求发送到 DispatcherServlet 调用 HandlerMapping 根据 url-pattern 找到对应的处理器。
2. 调用 HandlerAdepter 做类型转换。
3. 调用 preHandle 。
4. Controller 处理完成返回 ModelAndView 对象。
5. 调用 postHandle 。
6. 由 ViewReslover 解析后返回 View 。

