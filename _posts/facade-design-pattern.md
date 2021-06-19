---
title: Facade Design Pattern
tags:
  - Interview
category:
  - Design Pattern
author: bsyonline
lede: 没有摘要
date: 2019-03-12 22:11:08
thumbnail:
---



门面模式（Facade Pattern）是一种常用的封装模式。门面模式为子系统提供了统一的接口，隐藏了子系统的复杂性，是系统易于使用。

![1552403113798](https://raw.githubusercontent.com/bsyonline/pic/master/20190312/1552403341010.png)

以 slf4j 为例，slf4j(Simple Logging Facade for Java)是一个服务于各种日志框架（如log4j，logback，slf4j-impl 等）的日志门面，它并不包含日志的具体实现。slf4j 提供了统一的接口，在使用多种日志框架时只需要使用 slf4j 定义的接口，而不需要关心具体的日志框架使用，大大降低了使用日志框架的复杂度。

我们如果在项目中只加入 slf4j ，日志是不生效的。比如我们使用日志简单打印一条日志：

```
public class Slf4jTest {

    static Logger logger = LoggerFactory.getLogger(Slf4jTest.class);
    public static void main(String[] args) {
        logger.info("today is {}, the air temperature is {} at {}.", "Warmer Day", "18 degrees", "10:00");
    }
    
}
```

在 pom.xml 中加入依赖

```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
```

执行会得到警告

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

由此可见只使用 slf4j 日志是不生效的。查看源码我们发现 sfl4j 会去加载 org/slf4j/impl/StaticLoggerBinder.class ，而我们没有加入任何 StaticLoggerBinder 的实现，所以自然也就不会打印出日志。

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20190312/1552747908647.png"/>

如果我们想使用日志，那么加入任何一种日志框架即可，当然也包括自己实现的日志框架。

为了能够打印日志，我们可以自己加一个 StaticLoggerBinder 的简单实现。

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20190312/1552748321761.png"  style="width:350px" >

首先是具体的日志实现 MyLogger ，我们只简单重写 info 方法。

```
public class MyLogger implements Logger {
	...
	@Override
    public void info(String format, Object... arguments) {
        format = format.replace("{}", "%s");
        System.out.println(String.format(format, arguments));
    }
    ...
}
```

接着创建一个工厂。

```
public class MyLoggerFactory implements ILoggerFactory {
    @Override
    public Logger getLogger(String name) {
        return new MyLogger();
    }
}
```

最后使用 StaticLoggerBinder 来告诉日志门面使用那个 Logger 。

```
public class StaticLoggerBinder implements LoggerFactoryBinder {

    static StaticLoggerBinder SINGLETON = new StaticLoggerBinder();
    ILoggerFactory loggerFactory = new MyLoggerFactory();
    @Override
    public ILoggerFactory getLoggerFactory() {
        return loggerFactory;
    }

    @Override
    public String getLoggerFactoryClassStr() {
        return loggerFactory.getClass().getName();
    }

    public static StaticLoggerBinder getSingleton() {
        return SINGLETON;
    }
}
```

这样就可以正常输出日志了。使用了门面，我们可以自由的更换我们想要的日志实现，而对编程来说，只要 slf4j 的日志接口不变，代码完全不用修改，这提高了系统的灵活性，降低了复杂度。