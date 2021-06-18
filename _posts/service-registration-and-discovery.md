---
title: Service Registration and Discovery
tags:
  - Microservices
  - Spring Cloud
  - Eureka
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-16 08:09:49
thumbnail:
---

服务发现和注册是微服务架构的关键步骤，它是一种使微服务能够在不知道确切位置（通常是URL）的情况下使用其他微服务的机制。在简单的环境中服务可以通过静态配置记录每个服务的地址、端口和 URL 等信息，但是在微服务架构中，由于服务的动态增减和持续集成的因素，静态的配置信息将不再适用，所以就需要能够动态记录各个服务的连接信息，这就是服务注册和发现产生的原因。
服务注册和发现通常需要以下几步：
1. 服务提供者向注册中心提交自己的配置信息；
2. 服务使用者向注册中心询问服务提供者的配置信息；
3. 注册中心返回最新的服务提供者配置信息；
4. 服务使用者与服务提供者通信。

下图展示了两个服务通信的简单流程：
![](https://vaadin.com/documents/226808/13548263/client-side-load-balancer-flow.png/df1c6235-311e-4715-a232-4ecfca470b95?t=1512740698000)

服务注册和发现的程序原理比较简单，我们可以假想通过维护一个 key-value 的集合来提供注册发现服务会涉及哪些问题：
1. 假如以 redis 作为注册中心，服务将配置信息发送到 redis ，通过 add 将信息保存起来；
2. 服务在注销时从 redis 中把注册信息删除；
3. 正常情况下，1 和 2 就可以提供服务，但是这并不完善。如果服务的配置信息变更了没有及时向注册中心提交信息或是服务停止了而没有删除注册中心的信息，那么在别的服务在请求注册中心的时候就会得到错误信息，导致服务通信失败。这是就需要“心跳服务”，每隔一段时间，发送一次心跳，如果注册中心在一段时间内没有收到心跳，则认为服务死亡，从而删除该服务的注册信息；
4. 注册中心维护着所有服务的访问信息，服务之间要通信，首先要访问注册中心，那么注册中心就会成为瓶颈，所以如果采用主从模式在每个客户端维护一份副本，那么服务就不用每次都去访问注册中心了。这样虽然解决了注册中心的瓶颈问题，但是需要主从之间进行同步；
5. 作为主节点，如果注册服务关闭，那么其他副本也就无法同步，所以主节点还要解决单点问题，比如使用分布式机器来提供服务。

综上看来，实现一个服务注册和发现的程序还是比较复杂的。不过，有很多优秀的框架已经实现了这些功能，比如 Eureka。
#### **Eureka**
Eureka 是 netflix 开源的产品，通过和 Spring Cloud 集成，通过 Java 注解声明式的注册和调用服务变得非常简单，以下以 Greenwich.SR1 为例说明。

1.加入 maven 配置

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

2.在 springboot 启动类上添加注解 ```@EnableEurekaServer```

```
@EnableEurekaServer
@SpringBootApplication
public class RegisterCenterApplication {
	public static void main(String[] args) {
		SpringApplication.run(RegisterCenterApplication.class, args);
	}
}
```
3.修改配置文件

```
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false #不对外提供服务调用
    fetchRegistry: false #不拉取注册信息
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```
简单三步即可实现一个注册发现服务。启动服务访问 [http://localhost:8761/](http://localhost:8761/) 即可看到界面。
此时还没有任何服务注册到 Eureka ，我们接下来就来看看如何将服务注册到 Eureka 。

我们再创建另一个应用 eureka-client 注册到 eureka server 上。
1.添加 maven 配置

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
2.在启动类上添加注解 ```@EnableDiscoveryClient```，从 Edgware.RELEASE 版本之后这个注解可以省略。

```
//@EnableDiscoveryClient
@SpringBootApplication
public class Service1Application {
    public static void main(String[] args) {
        SpringApplication.run(Service1Application.class, args);
    }
}
```
3.添加配置文件。

```
spring:
  application:
    name: service1
server:
  port: 9900
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```
完成以上三步，启动服务，即可在 Eureka 中看到注册的服务 eureka-client 。