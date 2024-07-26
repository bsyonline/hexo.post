---
title: Services Communication using Feign
tags:
  - Microservices
  - Feign
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-20 16:22:09
thumbnail:
---

在服务注册到 Eureka 后，可以通过 Eureka 获取服务地址，使用 http 可以进行通信，这和我们使用 httpclient 、jeseryclient 或 restTemplate 的方式一样。但是还有一种更为简单的方式，就是 Feign 。Feign 是 Spring Cloud 的组件之一，是一个声明式的 REST 客户端。

#### **使用声明式的 Feign 注解**
Feign 内部集成了 Ribbon ，使用起来要简单很多。假定我们已有服务 order-service 。
```
@RestController
@Slf4j
public class OrderController {
    @GetMapping("/orders/{id}")
    public Order get(@PathVariable String id) {
        return new Order("1", LocalDateTime.now().toString(), "1");
    }
}
```
这就是一个普通的服务。那么如何使用 feign 来进行通信呢？

1.在的 api-gateway 中添加 maven 配置

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2.创建一个 feign 客户端

```
@FeignClient(name="order-service")
@Service
public interface OrderService {
    @GetMapping(value = "/orders/{id}")
    Order findById(@PathVariable("id") String id);
}
```
FeignClient 的 name 要指定，请求的路径和服务端一致，请求参数也应和服务端一致。

3.在启动类添加注解 ```@EnableFeignClients```

```
@EnableFeignClients
@SpringBootApplication
public class FeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }
}
```
4.调用 feign 客户端

```
@Test
public void findById() {
    Order result = orderService.findById("1");
    Assert.assertEquals("1", result.getId());
}
```
可以看到，比我们使用 httpclient 等方式要简洁很多。

#### **Feign 的其他用法**
如果想定制 Feign 的功能，可以使用 FeignClientsConfiguration 。FeignClientsConfiguration 可以复写 ```feign.Decoder```、```feign.Encoder``` 和 ```feign.Contract``` ，并在 ```@FeignClient``` 中添加属性。
```
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```

