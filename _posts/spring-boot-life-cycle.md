---
title: Spring Boot Life Cycle
tags:
  - Interview
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2019-11-07 18:38:52
thumbnail:
---

spring boot 启动流程
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









