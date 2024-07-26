---
title: Spring Boot in Action
tags:
  - Interview
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2020-02-25 12:07:18
thumbnail:
---

### Springboot 启动流程

1. 实例化 SpringApplication
   1.1 扫描 ApplicationContextInitializer 的配置并实例化（通过 SpringFactoriesLoader 加载 spring.factories 中的配置）
   1.2 扫描 ApplicationListener 的配置并实例化（通过 SpringFactoriesLoader 加载 spring.factories 中的配置）
2. 执行 SpringApplication.run() 方法
   2.1 创建并启动 stopwatch
   2.2 扫描 SpringApplicationRunListener 并将实例化后的 EventPublishingRunListener 加入到 RunListener 集合
   2.3 遍历 RunListener 集合，启动 runListener ，创建一个 **SpringBootStartingEvent** 加入到事件广播  
   2.4 包装命令行的参数
   2.5 配置环境变量  
   &ensp;&ensp;&ensp;&ensp;2.5.1 根据 applicationType 获取环境变量  
   &ensp;&ensp;&ensp;&ensp;2.5.2 配置自定义环境变量，比如命令行的参数
   2.6 打印 banner
   2.7 根据 applicationType 实例化 ConfigurableApplicationContext 类  
   2.8 扫描 SpringBootExceptionReporter 的配置并实例化，将 failureAnalyzer 加入到 exceptionReporter 集合  
   2.9 准备 context
   &ensp;&ensp;&ensp;&ensp;2.9.1 set 环境变量
   &ensp;&ensp;&ensp;&ensp;2.9.2 做一些后置处理  
   &ensp;&ensp;&ensp;&ensp;2.9.3 遍历 Initializer 集合执行每个 initializer 的 initialize() 方法  
   &ensp;&ensp;&ensp;&ensp;2.9.4 创建一个 **ApplicationContextInitializedEvent** 加入到事件广播  
   &ensp;&ensp;&ensp;&ensp;2.9.5 创建 beanDefinitionLoader
   &ensp;&ensp;&ensp;&ensp;2.9.6 ApplicationListener 和 Context 互相 set，然后创建一个 **ApplicationPreparedEvent** 加入到事件广播  
   2.10 refreshContext
   &ensp;&ensp;&ensp;&ensp;2.10.1 调用 super.refresh()
   &ensp;&ensp;&ensp;&ensp;2.10.1.1 prepareRefresh
   &ensp;&ensp;&ensp;&ensp;2.10.1.2 prepareBeanFactory
   &ensp;&ensp;&ensp;&ensp;2.10.1.3 postProcessBeanFactory  
   &ensp;&ensp;&ensp;&ensp;2.10.1.4 invokeBeanFactoryPostProcessors
   &ensp;&ensp;&ensp;&ensp;2.10.1.5 registerBeanPostProcessors
   &ensp;&ensp;&ensp;&ensp;2.10.1.6 initApplicationEventMulticaster
   &ensp;&ensp;&ensp;&ensp;2.10.1.7 onRefresh 创建 webserver
   &ensp;&ensp;&ensp;&ensp;2.10.1.8 registerListeners
   &ensp;&ensp;&ensp;&ensp;2.10.1.9 finishBeanFactoryInitialization
   &ensp;&ensp;&ensp;&ensp;2.10.1.10 finishRefresh  
   &ensp;&ensp;&ensp;&ensp;2.10.1.10.1 启动 webserver
   &ensp;&ensp;&ensp;&ensp;2.10.1.10.2 创建一个 **ServletWebServerInitializedEvent** 加入到事件广播  
   &ensp;&ensp;&ensp;&ensp;2.10.2 registerShutdownHook
   2.11 afterRefresh 什么都没有做
   2.12 创建一个 **ApplicationStartedEvent** 加入到事件广播  
   2.13 创建一个 **ApplicationReadyEvent** 加入到事件广播  



### Springboot java -jar 启动原理

Spring Boot 是如何通过 java -jar 方式运行的？为了搞清楚其中缘由，我们需要先构建一个 Spring Boot 工程。


通过 mvn package 将 Spring Boot 项目打包，然后解压缩，我们可以看到 jar 中的结构。这是一个 fat-jar ，其中主要有 3 部分：

1. BOOT-INF，用来存放应用的 class 文件和 Spring BOOT 依赖的 jar 。
2. META-INF，用来存放 MANIFEST.MF 文件。
3. spring-boot-loader 的 class 文件。

打开 MANIFEST.MF 文件我们会发现有两个信息：

```
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.rolex.springbootlearning.SpringBootLearningApplication
```

我们知道，如果我们需要打一个可执行的 jar ，那么 Main-Class 就是这个 jar 的入口。但是我们工程实际的程序入口是 Start-Class 对应的类，所以我们可以大胆猜测 Spring Boot 是通过 JarLauncher 来引导执行应用程序的入口类的。
接下来我们就要 debug 一下 JarLauncher 的执行过程。正常构建 Spring Boot 工程不需要加 spring-boot-loader 依赖，Spring Boot 会通过插件自动加入。由于我们需要通过 JarLauncher 启动 debug ，所以加入 spring-boot-loader 依赖。

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-loader</artifactId>
	<scope>provided</scope>
</dependency>
```

然后配置运行配置。
<img src="https://s2.ax1x.com/2020/02/25/3JcSvn.png" alt="3JcSvn.png" border="0" />
最后在 JarLauncher 中打上断点就可以启动调试了。

1. 首先获取文件路径，加载 fat-jar 中所有的 jar 的 URL path 。
   <img src="https://s2.ax1x.com/2020/02/25/3JR4dP.png" alt="20200225103627" border="0">
2. 然后加载 Manifest 信息，读取 Start-Class 。
   <img src="https://s2.ax1x.com/2020/02/25/3JR5If.png" alt="20200225102947" border="0">
3. 最后通过反射 invoke 应用程序的 main 方法。
   <img src="https://s2.ax1x.com/2020/02/25/3JRhZt.png" alt="20200225103200" border="0">

通过以上 3 步就实现了通过 java -jar 方式启动 Spring Boot 应用程序。



### Springboot 自动配置

spring boot 的自动配置主要包含两步：

1. 扫描 spring.factories 文件获得依赖
2. 通过条件注解对依赖进行过滤并进行加载。

先看第一步。

@EnableAutoConfiguration 通过 @Import 引入了 AutoConfigurationImportSelector 。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```
我们需要重点关注一下 getAutoConfigurationEntry() 方法。
<img src="https://s2.ax1x.com/2020/02/25/3tCwt0.png" alt="3tCwt0.png" border="0" />
这个方法通过 SpringFactoriesLoader 加载 META-INF/spring.factories 中的配置，在 spring-boot-autoconfigure 中 auto configuration 配置了 100+ 个类的全名，通过逐个 jar 包扫描，获取所有的依赖。

接下来是第二步。

通过排除，过滤最终剩下需要自动配置的 29 个类。有了类的全名，在 doCreateBean() 方法中通过 beanFactory 创建类的实例。
自动配置除了上边的加载过程，还有一个重要的环节就是条件注解。我们看到在 spring.factories 中配置的类都是 xxxAutoConfiguration 这样的配置类，这些类有一个共同的特点就是都会带一些条件注解，以  JdbcTemplateAutoConfiguration 为例。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class }) //当类路径下有指定类的条件下
@ConditionalOnSingleCandidate(DataSource.class)		   //当指定Bean在容器中只有一个，或者虽然有多个但是指定首选Bean	
@AutoConfigureAfter(DataSourceAutoConfiguration.class)      //自动配置必须在指定类之后进行
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
}
```
可以看到，有很多 @ConditionalOnXXX 的注解，这就是条件注解。条件注解的作用就是给 bean 的创建加上一些限制条件，以保证在自动实例化和注入时正确执行。 
综上所述，Spring Boot 的自动配置就是通过 @EnableAutoConfiguration + 条件注解实现的。

>常见的一些条件注解：
@ConditionalOnBean：当容器里有指定Bean的条件下
@ConditionalOnClass：当类路径下有指定类的条件下
@ConditionalOnExpression：基于SpEL表达式作为判断条件
@ConditionalOnJava：基于JVM版本作为判断条件
@ConditionalOnMissingBean：当容器里没有指定Bean的情况下
@ConditionalOnMissingClass：当类路径下没有指定类的条件下
@ConditionalOnProperty：指定的属性是否有指定的值
@ConditionalOnResource：类路径是否有指定的值
@ConditionalOnSingleCandidate：当指定Bean在容器中只有一个，或者虽然有多个但是指定首选Bean



### Springboot Starters 

Spring Boot Starters 是什么？实际上就是一组依赖描述合集。以 spring-boot-starter-web 为例，工程只有一个 pom 文件，其中就是几个依赖。

```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-json</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-tomcat</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-validation</artifactId>
		<exclusions>
			<exclusion>
				<groupId>org.apache.tomcat.embed</groupId>
				<artifactId>tomcat-embed-el</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
	</dependency>
</dependencies>
```

Spring Boot Starts 的作用就是**减少手动来管理 pom 依赖，提高 pom 的可管理性，减少项目的整体配置的时间**。 



### Spring boot 集成 JMS

![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/jms_2.png)

**1. 加入 maven 依赖**
pom.xml

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jms</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.activemq</groupId>
	<artifactId>activemq-core</artifactId>
	<version>5.7.0</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>com.google.code.gson</groupId>
	<artifactId>gson</artifactId>
	<version>2.3.1</version>
</dependency>
```

**2. 在 Server 端的主类中加入 @EnableJms 注解，并注册两个队列**

```java
import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.annotation.EnableJms;

import javax.jms.Queue;

@SpringBootApplication
@EnableJms
public class Main {

	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
	}

	@Bean(name = "requestQueue")
	public Queue requestQueue() {
		return new ActiveMQQueue("Request.Queue");
	}

	@Bean(name = "responseQueue")
	public Queue responseQueue() {
		return new ActiveMQQueue("Response.Queue");
	}
}
```

**3. 创建查询请求消息的消费者和查询响应消息的生产者**
生产者

```java
import javax.annotation.Resource;
import javax.jms.Queue;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

@Component
public class Producer {

	@Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

	@Resource(name = "responseQueue")
	private Queue responseQueue;

	public void send(String msg) {
		this.jmsMessagingTemplate.convertAndSend(this.responseQueue, msg);
	}

	public void send(Response msg) {
 		this.jmsMessagingTemplate.convertAndSend(this.responseQueue, msg);
	}

}
```

消费者

```java
import com.google.gson.Gson;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer {

	@Autowired
	Producer producer;

	@JmsListener(destination = "Request.Queue")
    public void receiveQueue(String text) {
		System.out.println(text);
		Gson gson = new Gson();
	 	Request request = gson.fromJson(text, Request.class);
		System.out.println("do query");
		producer.send(new Gson().toJson(new Response(request.getId(),"ok")));
	}

	@JmsListener(destination = "Request.Queue")
    public void receiveQueue(Request obj) {
		System.out.println(obj.toString());
		System.out.println("do query");
		producer.send(new Gson().toJson(new Response(obj.getId(),"ok")));
	}

}
```

实体类

```java
import java.io.Serializable;

public class Request implements Serializable {

    private static final long serialVersionUID = -797586847427389162L;
	private final String id;

	public Request(String id) {
		this.id = id;
	}

	public String getId() {
		return id;
	}
}

import java.io.Serializable;

public class Response implements Serializable {

  private static final long serialVersionUID = -797586847427389162L;
	private final String id;
	private final String result;

	public Response(String id, String result) {
		this.id = id;
		this.result = result;
	}

	public String getId() {
		return id;
	}


	public String getResult() {
		return result;
	}
}
```

客户端和服务端类似，把生产者和消费者对应的队列交换即可。

```java
import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.annotation.EnableJms;

import javax.jms.Queue;

@SpringBootApplication
@EnableJms
public class Main {

	@Bean(name = "requestQueue")
	public Queue requestQueue() {
		return new ActiveMQQueue("Request.Queue");
	}

	@Bean(name = "responseQueue")
	public Queue responseQueue() {
		return new ActiveMQQueue("Response.Queue");
	}

	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
	}
}
```

生产者

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import javax.jms.Queue;

@Component
public class Producer {

	@Autowired
    private JmsTemplate jmsTemplate;

	@Resource(name = "requestQueue")
	private Queue requestQueue;

	public void send(String msg) {
		this.jmsTemplate.convertAndSend(this.requestQueue, msg);
	}

	public void send(Request request) {
 		this.jmsTemplate.convertAndSend(this.requestQueue, request);
	}

}
```

消费者

```
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer {

	@JmsListener(destination = "Response.Queue")
    public void receiveQueue(String text) {
		System.out.println("receive json response");
		System.out.println(text);
	}

	@JmsListener(destination = "Response.Queue")
    public void receiveQueue(Request obj) {
		System.out.println("receive response");
		System.out.println(obj.toString());
	}

}
```

### Springboot cache （redis）

1. pom.xml

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. cachemanager

```
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.core.RedisTemplate;

import java.lang.reflect.Method;

@Configuration
@EnableCaching
public class RedisCache {
    @Bean
    public CacheManager cacheManager(
            @SuppressWarnings("rawtypes") RedisTemplate redisTemplate) {
        return new RedisCacheManager(redisTemplate);
    }

    /**
     * 自定义key的生成策略
     * @return
     */
    @Bean
    public KeyGenerator myKeyGenerator(){
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }
}
```

3. 方法上添加注解

```
@Cacheable(value = "entInfo", keyGenerator = "myKeyGenerator")
public EntMultipleInfo findEntInfo(String entName) {
    looger.info("no cache");
}
```

>在调用方法前会先去查询是否有缓存。
>对象需要能够序列化

4. application.properties

```
spring.redis.database= 3
spring.redis.host=192.168.11.21
spring.redis.port=6379
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
```