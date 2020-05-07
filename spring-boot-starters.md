---
title: Spring Boot Starters
tags:
  - Interview
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2020-02-25 19:27:54
thumbnail:
---

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