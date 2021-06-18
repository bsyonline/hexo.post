---
title: 一个简单的 Spring Boot + JMS Sample
date: 2016-10-26 15:40:04
tags:
 - Spring Boot
 - JMS
category: 
 - Spring Boot
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

学习 spring boot 集成 JMS 时写的一个小程序。

![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/jms_2.png)

<!--more-->

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
```java
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
