---
title: Configuration Management with Spring Cloud Config
tags:
  - Microservices
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-26 10:11:01
thumbnail:
---

微服务架构中面临的一个问题就是每个服务有一个配置，随着服务的不断增加，配置文件散落各处，管理不便。Spring Cloud 提供一套中心化配置管理的方案，通过 Spring Cloud Config Server 来对服务提供配置服务。Spring Cloud Config 支持多种管理配置文件的方式，比如环境变量、本地文件、版本控制工具等，相比之下使用版本控制工具来管理更加方便。假如我们使用 github 来管理配置文件，形式如下：
```
/config
    /default
        config-client.yml
    /dev
        config-client-dev.yml
    /prod
        config-client-prod.yml
    /test
        config-client-test.yml
```
default 下的配置文件以应用名称命名，存放公共的配置信息。dev，prod 和 test 分别对应开发，生产和测试的配置文件，各自存放环境差异的配置。Spring Boot 在加载配置文件的时候会先加载 defualt ，然后在加载指定的配置文件，如果有相同的配置，default 中的配置会被覆盖。使用 git 管理配置文件只是我们的第一步，接下来我们看看如何创建 Config Server 。
### **Config Server**
通过声明式注解的方式可以很方便地创建一个 Config Server 。简单起见，我们先在本地创建一个config-repo 文件夹，然后新建三个配置文件分别为 order-service-dev.yml，order-service-test.yml，order-service-prod.yml，并对其内容加以区分。接下来就可以来配置 congif-server 了。

1.首先添加 maven 的配置

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
2.修改配置文件

```
server:
  port: 8888
spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: file:///D:\config-repo
```
3.最后给启动类加上 ```@EnableConfigServer``` 注解

```
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

启动服务访问 [http://localhost:8888/order-service/dev](http://localhost:8888/order-service/dev) 即可看到开发环境的配置信息。

### **Config Client**

接下来看看客户端如何使用 Config Server 来获取配置。

1.添加 maven 配置

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2.添加配置文件

客户端的配置文件有一个特殊的地方需要注意，Config Server 的连接信息要写在 bootstrap.yml 中，不能写在 application.yml 中。原因是 bootstrap.yml 会先于程序启动，这样就可以在程序启动之前加载配置信息。
```
spring:
  application:
    name: order-service
  cloud:
    config:
      uri:  http://localhost:8888
      name: ${spring.application.name}
      profile: dev
```
>bootstrap.yml 中的内容都是静态的，如果将 bootstrap.yml 打成镜像，就不能灵活修改了，所以 label 和 profile 最好不要写在配置中，而是通过启动参数来控制 ```java -jar config-client-1.0-SNAPSHOT.jar --spring.profiles.active=prod --spring.cloud.config.label=develop --spring.cloud.config.profile=prod``` 。

配置好以后，启动服务可以看到类似的日志。

```
c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8888
```

并且程序可以正常启动。

3.可以添加测试方法来验证。

```
@Value("${msg}")
String msg;

@GetMapping("/msg")
String getMsg() {
	return msg;
}
```

http://localhost:9001/msg 会得到 msg 的信息。

### 配置更新

我们已经通过 Config Server 管理配置文件，并配置 Config Client 获取配置，但此时如果更新了配置文件，order-service 服务是不会生效的。因为配置是在启动时候加载的，如果配置文件更新，程序是无法动态获取最新的配置的，只有重启之后再次加载配置。那么有没有办法让应用不停机就能获取最新的配置信息呢？答案当然是肯定的。Spring Cloud Config 提供了一个入口可以远程刷新配置，这个入口是隐藏的，需要借助 actuator 才能开启。

1.首先在 maven 中添加 actuator 配置

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2.在 controller 上添加 ```@RefreshScope``` 注解

```
@RefreshScope
public class OrderController {
    ...    
}
```
3.添加配置信息，打开 actuator 开关

```
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
配置完成后启动服务，更新配置文件并请求 msg 信息，msg 还是启动之前的信息。使用 ```curl -XPOST 'http://localhost:9001/actuator/refresh'``` 来刷新配置，成功后会返回 ["msg"] 信息。此时再重新请求 msg 就会发现 msg 信息已经更新了。

### 使用 Git 保存配置

我们已经通过本地的 config-repo 实现了 Config Server ，但是通常很少有使用本地文件作为 config-repo，更多的是将使用 git 来做 config-repo 。要使用 git 需要对 config-server 的配置进行修改。

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/bsyonline/config-repo # git repo 的地址
          search-paths: /** # 检索该目录下的配置文件
```

可以通过 ```/{name}/{profile}/{label}```  的方式访问。

- name ${spring.application.name}
- profile ${spring.profiles.active} ，就是环境后缀 dev ， prod 和 test
- label 分支，默认为 master