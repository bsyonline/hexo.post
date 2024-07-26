---
title: Routing and Filtering with Zuul
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-17 18:24:06
thumbnail:
---

在微服务架构中系统为客户端应用程序提供一个独特的入口 —— API 网关是一种非常常见的微服务模式。API 网关实现代理、路由、流量控制，同时为用户提供了统一的界面，而屏蔽了微服务内部的细节。
使用 API 网关的优点：
* 集中处理逻辑，比如鉴权、速率限定、监控等；
* 提高了安全性，对外只暴露一个入口；
* 提高了扩展性，重构微服务不会影响客户行为。

同时，使用 API 网关也会带来以下缺点：
* 面临单点问题；
* 增加了额外的路由。

### **Spring Cloud Zuul**
Spring Cloud Zuul 是 Netflix 开源的基于 Java 的 API 网关，通过声明式注解可以方便的实现路由、过滤等功能。
#### 实现路由
首先我们对 service1 服务进行一些改造，为它添加一个访问入口。
```
@RestController(value = "/api")
public class Service1Controller {

    @GetMapping(value = "/hello")
    public String hello(String name) {
        return "hello, " + name;
    }
}
```
访问 [http://localhost:9900/api/hello?name=tom](http://localhost:9900/api/hello?name=tom) 可以看到返回结果。通常不会直接将服务入口暴露出来，而是在服务前加一层网关来进行代理，比如 nginx，zuul 也可以实现类似功能。

1.创建一个工程，加入 maven 配置

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

2.在启动类添加注解```@EnableZuulProxy```

```
@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

3.添加配置文件

```
spring:
  application:
    name: api-gateway
server:
  port: 9000
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
zuul:
  routes:
    hello: # 访问的 uri
      url: http://localhost:9900/api/hello
  prefix: /api
```

完成以上配置，访问 [http://localhost:9000/api/hello?name=bob](http://localhost:9000/api/hello?name=bob) 会跳转到 service1 服务。

zuul 的路由规则可以指定 url 也可以通过 path 来匹配，比如:

```
zuul:
  routes:
    service1:
      path: /hello
      serviceId: service1
```

这个规则会匹配 service1 服务的 "/hello" 接口。

还可以支持通配符，比如： 

```
zuul:
  routes:
    service4:
      path: /hello/*/world
      serviceId: service1
```

如果 service1 服务有2个接口分别是 "/hello/{id}/world" 和 "/{id}/world" ，这个规则会将请求路由到 "/{id}/world" 这个接口。"\*" 号表示匹配一级，如果有多级可以用 "**" 。

#### 实现负载均衡
在上一个例子中， zuul 已经实现了负载均衡的功能，只不过我们只有一个实例，为了显示直观，我们重新创建一个 service2 并加入一些返回信息。
```
@RestController
public class Service2Controller {

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping(value = "/hello")
    public String hello(HttpServletRequest request) {
        return "host=" + request.getRemoteHost() + ", port=" + request.getServerPort();
    }
}
```
更新 api-gateway 的配置文件：
```
zuul:
  routes:
    service2:
      path: /service2/**
      serviceId: service2
  prefix: /api
```
启动服务使用不同端口启动两个 service2 实例，然后访问 [http://localhost:9000/api/service2/hello](http://localhost:9000/api/service2/hello) ，刷新请求会看到返回信息在来回切换，这实际上就是 zuul 实现了负载均衡，对 service2 的两个实例轮询访问。

#### 过滤器
过滤器是 Spring Cloud Zuul 的核心组件之一，对应请求的生命周期，zuul 定义了 4 种过滤器类型：
* pre 在请求被路由之前调用；
* route 将请求路由到微服务；
* post 在路由到微服务以后执行；
* error 发生错误时执行。

除了以上 4 种，zuul 还支持自定义过滤器。
##### Pre Filter

我们先来看看 pre filter 。我们编写一个简单的 pre filter 来打印访问日志信息。

```
@Component
@Slf4j
public class PreFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("send %s request to %s", request.getMethod(), request.getRequestURL().toString()));
        return null;
    }
}
```
在 api-gateway 项目中加入上面的代码，然后重启，每次发送请求都可以看到打印的日志信息。
zuul 内置了很多过滤器，如果要去掉一个过滤器，可以通过配置文件灵活配置。
```
zuul:
  PreFilter:
    pre:
      disable: true
```
这样我们刚才加入的 PreFilter 就不起作用了，可以重启服务试试看。同理，对于其他的过滤器，我们也可以使用 ```zuul.{filterName}.{filterType}.disable``` 的方式来进行配置。
##### Route Filter
route filter 的作用就是把请求路由到其他服务。我们在本文的开始通过配置，实现了路由功能，现在我们通过自己写一个过滤器来实现路由功能。

```
@Component
@Slf4j
public class RouteFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "route";
    }

    @Override
    public int filterOrder() {
        return 10;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletResponse response = ctx.getResponse();
        String url = "http://cn.bing.com";
        try {
            log.info(String.format("redirect to %s", url));
            response.sendRedirect(url);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
##### Post Filter
Post Filter 用来处理服务对响应后要进行的操作，比如添加 http header 。

```
@Component
@Slf4j
public class PostFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "post";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        log.info("post");
        RequestContext ctx = RequestContext.getCurrentContext();
        List<Pair<String, String>> headers = ctx.getZuulResponseHeaders();
        headers.add(1, new Pair<String, String>("X-RateLimit-Remaining", "30"));
        return null;
    }
}
```
##### Error Filter
在过滤器中出现的任何错误都会进入 error filter 处理，所以在 error filter 中统一处理异常比在每一处代码都使用 try-catch 要简单很多。

```
@Component
@Slf4j
public class ErrorFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "error";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        Throwable throwable = ctx.getThrowable();
        log.info(String.format("take exception %s", throwable.getCause().getMessage()));
        return null;
    }
}
```
这样在出错时就可以得到一个 ```/error``` 的映射，比如可以对映射进行简单的处理：
```
@RestController
public class ErrorHandlerController implements ErrorController {
    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping("/error")
    public String error() {
        return "{\"code\": 500, \"msg\":\"internal server error\"}";
    }
}
```
这样错误就不会直接暴露给用户了。