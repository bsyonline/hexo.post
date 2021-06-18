---
title: Client Side Loadbalancing with Spring Cloud Ribbon
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-30 19:47:19
thumbnail:
---

Spring Cloud Ribbon 是 Netflix 开源的组件之一，主要功能是提供客户端的负载均衡。 Spring Cloud 和 Ribbon 的集成使得实现负载均衡变得非常容易。我们先实现一个简单的例子，由一个客户端在两个服务器之间实现负载均衡。

首先，创建一个 ribbon-server 

```
@GetMapping("/ribbon")
public String ribbon(HttpServletRequest request) {
	log.info("uri {} called", "/ribbon");
	return "host=" + request.getRemoteHost() + ", port=" + 	request.getServerPort();
}
```

mvn package 打包后，使用 java -jar xxx.jar --server.port=9999 和 java -jar xxx.jar --server.port=9988 启动两个服务。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```
创建客户端程序，添加依赖。

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

通过访问客户端可以发现请求轮流发送给服务端。

```
@Autowired
LoadBalancerClient loadBalancerClient;
@Autowired
RestTemplate restTemplate;

@GetMapping("/hello")
public String hello() {
	log.info("uri {} called", "/hello");
	ServiceInstance serviceInstance = loadBalancerClient.choose("ribbon-server");
    String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/ribbon";
    log.info("url : {}", url);
    return restTemplate.getForObject(url, String.class);
}
```
如果使用声明式注解，上边的代码也可以这样写：

```
@Autowired
RestTemplate restTemplate;

@LoadBalanced
@Bean
public RestTemplate restTemplate() {
	return new RestTemplate();
}

@GetMapping("/ribbon-client")
public String hello() {
	return restTemplate.getForObject("http://ribbon-server/ribbon", String.class);
}
```
使用 ```@RibbonClient``` 注解可以指定 ribbon 的自定义配置来替换默认配置。
```
@RibbonClient(name = "ribbon-server", configuration = RibbonConfiguration.class)
public class RibbonClient1Controller {

}
```
默认 ribbon 使用的是轮询策略，我们可以将其改成自定义策略，比如每访问 10 次 A 服务就访问一次 B 服务：

```
@Configuration
public class RibbonConfiguration {
    @Bean
    public IRule ribbonRule() {
        return new MyRibbonRule();
    }
}
```

自定义规则类可以继承 AbstractLoadBalancerRule 或是 实现 IRule 接口。

```
@Slf4j
public class MyRibbonRule extends AbstractLoadBalancerRule {
    private AtomicInteger nextServerCyclicCounter;
    private AtomicInteger times;

    public MyRibbonRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
        times = new AtomicInteger(0);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }
        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();
            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }
            int nextServerIndex = incrementTimes();
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }
            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }
            // Next.
            server = null;
        }
        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    private int incrementTimes() {
        for (; ; ) {
            int current = nextServerCyclicCounter.get();
            int next = times.addAndGet(1);
            if (next < 10) {
                log.info("times = {}", times);
                return current;
            } else {
                times.set(0);
                log.info("times set 0 and current = {}", current);
                return current + 1;
            }
        }
    }

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }
}
```
ribbon 除了 IRule 之外，还提供 IPing 接口用来通过 ping 服务器的 url 来判断服务是否可用。

#### ribbon 的懒加载问题

由于 ribbon 是懒加载的，所以在服务启动的时候并不会初始化 ribbon 的相关类，而是在第一次调用的时候才会初始化，这就导致第一次请求的时间会更长，这有可能就会因为超过了超时时间导致失败，所以为了解决这个问题，我们可以取消懒加载配置，让 ribbon 在启动的时候就初始化。

在 RibbonAutoConfiguration 类中有这样一段代码，意思就是会按照 `ribbon.eager-load.enabled` 属性来决定是否初始化。

```
@Bean
@ConditionalOnProperty("ribbon.eager-load.enabled")
public RibbonApplicationContextInitializer ribbonApplicationContextInitializer() {
	return new RibbonApplicationContextInitializer(springClientFactory(),
				ribbonEagerLoadProperties.getClients());
}
```

所以我们可以通过修改这个值来让 ribbon 启动时初始化。

```
ribbon:
  eager-load:
    enabled: true
```

