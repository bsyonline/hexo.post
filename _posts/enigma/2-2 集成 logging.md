---
title: Java
tags: 
category: 
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


日志的集成和 Springboot2 并无区别，还是使用 ``spring-boot-starter-logging`` 。

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-logging</artifactId>  
</dependency>
```

通常我们还会对日志进行一些定制，比如输出到什么地方，输出格式，日志文件大小，滚动等。这些都可以在 logback-spring.xml 中定义，文件放在 resources 下即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
    <contextName>logback</contextName>  
    <property name="app.name" value="enigma-backend"/>  
    <property name="log.path" value="./logs/${app.name}"/>  
    <!--输出到控制台-->  
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">  
        <layout>  
            <Pattern>%green(%d{yyyy-MM-dd' 'HH:mm:ss.SSS}) %magenta(%level) %blue(${PID:- }) %yellow(%thread) %red(%logger) - %msg ##'%exception'%n</Pattern>  
        </layout>  
    </appender>  
  
    <!--输出到文件-->  
    <appender name="infoFile" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log.path}/app.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <!-- 按天轮转 -->  
            <fileNamePattern>${log.path}/app-%d{yyyy-MM-dd}-%i.zip</fileNamePattern>  
            <!-- 单个文件最大 500M -->            
            <maxFileSize>500MB</maxFileSize>  
            <!-- 保存 30 天的历史记录，最大大小为 30GB -->
		    <maxHistory>30</maxHistory>  
            <totalSizeCap>3GB</totalSizeCap>  
        </rollingPolicy>  
        <layout>            
	        <Pattern>%d{yyyy-MM-dd' 'HH:mm:ss.SSS} %level ${PID:- } %thread %logger - %msg ##'%ex'%n</Pattern>  
        </layout>  
    </appender>  
    <appender name="errorFile" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log.path}/error.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <!-- 按天轮转 -->  
            <fileNamePattern>${log.path}/error-%d{yyyy-MM-dd}-%i.zip</fileNamePattern>  
            <!-- 单个文件最大 500M -->            
            <maxFileSize>500MB</maxFileSize>  
	        <!-- 保存 30 天的历史记录，最大大小为 30GB -->            
	        <maxHistory>30</maxHistory>  
            <totalSizeCap>3GB</totalSizeCap>  
        </rollingPolicy>  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>ERROR</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>  
        <layout>            
	        <Pattern>%d{yyyy-MM-dd' 'HH:mm:ss.SSS} %level ${PID:- } %thread %logger - %msg ##'%ex'%n</Pattern>  
        </layout>  
    </appender>  
  
    <root level="info">  
        <appender-ref ref="console"/>  
        <appender-ref ref="infoFile"/>  
        <appender-ref ref="errorFile"/>  
    </root>  
  
</configuration>
```
