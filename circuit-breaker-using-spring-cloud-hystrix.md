---
title: Circuit Breaker using Spring Cloud Hystrix
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-30 19:45:41
thumbnail:
---



断路器（Circuit Breaker）的是微服务系统中一个非常重要且关键的概念。为什么说它重要呢？是因为在微服务的系统中，服务之间是相互调用的，如果其中一个服务挂掉了，其他的服务请求还在不断的请求这个服务，如果没有特殊的处理，那么通常要等请求超时请求才能返回，每个请求就是一个线程，而在这期间线程不会被释放，那么很快调用端就会因为线程阻塞而消耗大量资源，一旦调用端资源耗尽，那么就会引起连锁反应。所以，为了保证服务之间尽可能少的相互影响，就提出了断路器的概念。

Hystrix 是 Netflix 开源的延迟和容错库，它提供了断路器的功能。本节我们来看看如何使用 Hystrix 来达到熔断的效果。

首先添加依赖。

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

第二步添加 fallback 方法，并给需要熔断的方法加上 @HystrixCommand 注解。

```
@HystrixCommand(fallbackMethod = "defaultOrder")
@GetMapping("/orders/{id}")
public Order findById(@PathVariable("id") String id) {
	return restTemplate.getForObject("http://order-service/orders/" + id, Order.class);
}

public Order defaultOrder(String id, Throwable e) {
	log.error("fall back findById", e);
	return new Order("0", "", "0");
}
```

第三步在启动类上添加 @EnableCircuitBreaker 注解。

在添加断路器之前，我们通过 order-client 调用 order-service 服务的 getById 方法大概需要 10+ 毫秒，如果我们停掉 order-service 服务，响应会 1000+ 毫秒才能返回，可见等待请求超时再返回会使线程阻塞很长时间，在有大量请求访问的时候会导致服务端资源耗尽。当我们启用断路器之后，如果 order-service 服务不可用，会直接返回 fallback 方法的返回值。

断路器的状态可以通过 `/actuator/health` 查看。

```
management:
  endpoint:
    health:
      show-details: always
```

在配置文件中加入 actuator 配置后可以查看 hystrix 的信息。

```
{
	"status": "UP",
	"details": {
		"diskSpace": {...},
		"refreshScope": {...},
		"discoveryComposite": {...},
		"hystrix": {
			"status": "CIRCUIT_OPEN"
		}
	}
}
```

hystrix 的断路器并不是始终打开的，当 `circuitBreaker.requestVolumeThreshold` （调用次数，默认是 20）在 `metrics.rollingStats.timeInMilliseconds` （默认 10 秒）定义的时间内故障率超过 `circuitBreaker.errorThresholdPercentage` （默认 50）时，则断路器会打开。

多请求几次 order-client 后可以看到断路器的状态变化。

```
{
	"status": "UP",
	"details": {
		"diskSpace": {...},
		"refreshScope": {...},
		"discoveryComposite": {...},
		"hystrix": {
			"status": "CIRCUIT_OPEN",
			"details": {
				"openCircuitBreakers": ["OrderClient::findById"]
			}
		}
	}
}
```

如果我们修改 circuitBreaker 的参数

```
@HystrixCommand(fallbackMethod = "defaultOrder", commandProperties = {
            @HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_ERROR_THRESHOLD_PERCENTAGE, value = "100"),
            @HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_REQUEST_VOLUME_THRESHOLD, value = "1")
})
```

重启之后调用一次 order-client 就可以看到 CIRCUIT_OPEN 的状态了。

#### Feign 使用 Hystrix

作为 Netflix 的组件 Feign 也整合了 Hystrix 。本节我们来看看如何使用 Feign 的 Hystrix 。

首先加入依赖，创建一个 feign-client ，可参考 [Services Communication using Feign](../../../../2018/06/20/services-communication-using-feign/) ，然后创建一个 fallback 实现。

```
@Component
public class OrderServiceFallback implements OrderService {
    @Override
    public Order findById(String id) {
        return new Order("0", "", "0");
    }
}
```

然后在 feign-client 中指定 fallback 。

```
@FeignClient(name = "order-service", fallback = OrderServiceFallback.class)
```

如果需要捕获异常可以这样写。

```
@Component
@Slf4j
public class OrderServiceFallbackFactory implements FallbackFactory<OrderClient> {
    private static final Order order = new Order("0", "", "0");
    @Override
    public OrderClient create(Throwable throwable) {
        return new OrderClient() {
            @Override
            public Order findById(String id) {
                log.error("fallback findById", throwable);
                return order;
            }
        };
    }
}
```

在 feign-client 中指定 fallbackFactory 。

```
@FeignClient(name = "order-service", fallbackFactory = OrderServiceFallbackFactory.class)
```

最后在配置文件中启用 feign hystrix 配置。

```
feign:
  hystrix:
    enabled: true
```

默认是 false 。

