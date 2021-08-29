---
title: design pattern
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-06-19 09:59:47
thumbnail:
---





## Part I 设计原则

### [SOLID](../../../../2020/04/05/the-basic-principles-of-design)

### 单一职责原则（Single Resposibility Principle，SRP）

### 开闭原则（Open Closed Principle，OCP）

### 里氏替换原则（Liskov Substitution Principle，LSP）

### 接口隔离原则（Interface Segregation Principle，ISP）

### 依赖倒置原则（Dependence Inversion Principle，DIP）

### 迪米特法则（Law of Demeter，LoD）

### KISS

### YAGNI

### DRY

### Rule of Three

## Part II 设计模式

### 创建型

#### 1. 单例模式（**Singleton** Design Pattern）

单例模式是最简单的设计模式之一，用于保证系统中只有一个类的实例的场景。

单例模式的特点：

* 只有一个实例
* 只能自己创建自己的实例
* 能够对外提供自己创建的实例

**程序示例**

1. 懒汉式

在 Gof 的设计模式中介绍单例模式使用的说明代码叫做懒汉式，java 版代码如下：

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

这是最简单的代码实现，但是这个代码存在问题，即在多线程的程序中，不能保证一定是单例的。原因也很简单，代码被编译成字节码被执行，有可能在刚进入 if 判断后时间分片就用完了，另一个线程也进入 if 判断，从而创建多个实例。一种简单的处理方法是将方法同步。

```java
synchronized public static Singleton getInstance() {
}
```



2. 饿汉式

另外还有一种比较常见的叫做饿汉式，这种方式更符合 java 的风格。

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

3. 静态内部类

通常情况下，直接使用恶汉式是最安逸的，况且内存越来越不是瓶颈了。但是如果还对装载类时初始化耿耿于怀，可以使用这种方式。

```java
public class Singleton {

    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

使用静态内部类的好处是不会在装载 Singleton 的时候就实例化，而是在调用 getInstance() 方法时才创建实例。

4. 枚举

在 jdk1.5 以后加入了枚举，也可以使用枚举来实现单例。

```java
public enum Singleton {
    INSTANCE;
    public void whateverMethod(){

    }
}
```

使用 enum 方式创建单例的好处是简单，简单到让人看不懂。怎么使用可参考一个例子 [枚举单例读取配置文件](../../../../2016/08/02/枚举单例读取配置文件/) 。



5. 双重校验锁

网上很多单例的文章中都提到了双重校验锁的方式。这种方式出现的目的是对懒汉式进行优化。

```java
public class Singleton {

    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

在《java 与模式》一书中针对这种方式也做了分析，这种方式并不能真正达到效果，所以还是安心使用饿汉吧。

2016-08-08 补充：
网上一些资料中提到，自 JDK 1.5 以后，修复了 volatile 关键字的 bug ，所以，JDK 1.5 以后双重校验锁可以正常工作了。关于 volatile 关键字更多知识点，参考 [Java 关键字 volatile](../../../../2016/08/08/Java 关键字 volatile/)

**类图**

如果要画出常用设计模式的类图，单例无疑是最简单的了。

![单例模式](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/singleton.png)



工厂模式

#### 2. 工厂方法模式（**Factory Method** Design Pattern）

#### 3. 抽象工厂模式（**Abstract Factory** Design Pattern）

#### 4. 建造者模式（**Builder** Design Pattern）

#### 5. 原型模式（**Prototype** Design Pattern）

### 结构型

#### 6. 代理模式（**Adapter** Design Pattern）

代理模式属于结构模式。代理模式的作用是可以在调用 target 时扩展 target 的功能。

#### 静态代理

有一个接口 Image 及它的实现。

```java
public interface Image {
    void display();
}

public class JPEGImage implements Image {
    String imageFilePath;

    public JPEGImage(String imageFilePath) {
        this.imageFilePath = imageFilePath;
    }

    @Override
    public void display() {
        System.out.println("display image" + imageFilePath);
    }
}
```

我们可以在通过代理 JPEGImage 在调用 display() 的时候扩展一些功能。

```java
public class ImageProxy implements Image {

    private JPEGImage target;

    public ImageProxy(String imageFilePath) {
        this.target = new JPEGImage(imageFilePath);
    }

    @Override
    public void display() {
        System.out.println("proxy start");
        target.display();
        System.out.println("proxy end");
    }
}

public class ImageViewer {
    public static void main(String[] args) {
		ImageProxy imageProxy1 = new ImageProxy("sample/photo1.jpeg");
		imageProxy1.display();
    }
}
```

<img src="https://s1.ax1x.com/2020/03/14/8QEoid.png" alt="static proxy" border="0" style="width:200px">
静态代理的缺点是，一个对象对应有一个代理对象，如果对象很多，对应和代理对象也就有很多。

**动态代理**

动态代理分为 jdk 动态代理和 cglib 代理。

**jdk 动态代理**

jdk 动态代理利用 Java 反射通过 InvocationHandler 实现。

```java
public class ImageProxy {

    Image target;

    public ImageProxy(Image target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("proxy start");
                Object obj = method.invoke(target, args);
                System.out.println("proxy end");
                return obj;
            }
        });
    }
}

public class ImageViewer {
    public static void main(String[] args) {
        Image jpegImage1 = new PNGImage("sample/photo1.png");
		Image imageProxy1 = (Image) new ImageProxy(jpegImage1).getProxyInstance();
		imageProxy1.display();
    }
}
```

<img src="https://s1.ax1x.com/2020/03/14/8QE4de.png" alt="dynamic proxy" border="0" style="width:300px">
jdk 动态代理特点：需要实现接口，通过 Java 反射实现。

**cglib 代理**

加入 cglib 依赖。

```
<dependency>
	<groupId>cglib</groupId>
	<artifactId>cglib</artifactId>
	<version>3.1</version>
</dependency>
```

```java
public class ImageProxy implements MethodInterceptor {

    GIFImage target;

    public ImageProxy(GIFImage target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return enhancer.create(new Class[]{String.class}, new Object[]{target.imageFilePath});
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("proxy start");
        Object obj = methodProxy.invokeSuper(o, args);
        System.out.println("proxy end");
        return obj;
    }
}
```

<img src="https://s1.ax1x.com/2020/03/14/8Q3RyQ.png" alt="8Q3RyQ.png" border="0" style="width:500px"/>
cglib 特点：不需要接口，利用继承，被代理的类和方法不能是 final ，利用字节码技术 asm 实现。

#### 7. 桥接模式（**Bridge** Design Pattern）



#### 8. 装饰器模式（**Decorator** Design Pattern）



#### 9. 适配器模式（**Adapter** Design Pattern）



#### 10. 门面模式（**Facade** Design Pattern）

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

#### 11. 组合模式（**Composite** Design Pattern）



#### 12. 享元模式（**Flyweight** Design Pattern）



### 行为型

#### 13. 观察者模式（**Observer** Design Pattern）



#### 14. 模板方法模式（**Template Method** Design Pattern）



#### 15. 策略模式（**Strategy** Design Pattern）



#### 16. 职责链模式（Chain of Responsibility）



#### 17. 状态模式（**State** Design Pattern）



#### 18. 迭代器模式（**Iterator** Design Pattern）



#### 19. 访问者模式（**Visitor** Design Pattern）



#### 20. 备忘录模式（**Memento** Design Pattern）



#### 21. 命令模式（**Command** Design Pattern）

**定义**

命令模式将请求（命令）封装为一个对象，这样可以使用不同的请求依赖注入到其他对象，并且能够支持请求（命令）的排队执行、记录日志、撤销等（附加控制）功能。

**优缺点**

#### 22. 中介模式（**Mediator** Design Pattern）



#### 23. 解释器模式（**Interpreter** Design Pattern）



#### 24. 回调模式