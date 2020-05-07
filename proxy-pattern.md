---
title: Proxy Pattern
tags:
  - Interview
category:
  - Design Pattern
author: bsyonline
lede: 没有摘要
date: 2020-03-14 13:11:19
thumbnail:
---


代理模式属于结构模式。代理模式的作用是可以在调用 target 时扩展 target 的功能。

#### 静态代理
有一个接口 Image 及它的实现。
```
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
```
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

#### 动态代理
动态代理分为 jdk 动态代理和 cglib 代理。
##### jdk 动态代理
jdk 动态代理利用 Java 反射通过 InvocationHandler 实现。
```
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
##### cglib 代理
加入 cglib 依赖。
```
<dependency>
	<groupId>cglib</groupId>
	<artifactId>cglib</artifactId>
	<version>3.1</version>
</dependency>
```
```
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