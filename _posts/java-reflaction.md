---
title: Java 反射
date: 2016-08-19 15:20:47
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

Java 反射是 jdk 1.5 加入新特性。Java 反射机制可以让我们在编译期 (Compile Time) 之外的运行期 (Runtime) 检查类，接口，变量以及方法的信息。反射还可以让我们在运行期实例化对象，调用方法，通过调用 get/set 方法获取变量的值。

如果我们要将一个对象 User 实例 A 的值赋给实例 B ，通常这样
```java
public class User{
    String name;
    int age;
    //getter/setter
}

User A = new User();
A.setName("tom");
A.setAge(20);
User B = new User();
B.setName(A.getName());
B.setAge(A.getAge());

```
这样写非常简单，但是如果对象 User 的属性很多，那么写起来就比较麻烦了。于是可以利用反射来实现：

```java
Class clazz = Class.forName("a.b.c.User");
Constructor constructor = clazz.getDeclaredConstructor();
Object B = constructor.newInstance();
Field[] fields = clazz.getDeclaredFields();
for (Field field : fields) {
    String fieldName = field.getName();
    String setMethodName = "set" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
    String getMethodName = "get" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
    Method setMethod = clazz.getDeclaredMethod(setMethodName, new Class[]{field.getType()});
    Method getMethod = clazz.getDeclaredMethod(getMethodName, new Class[]{});
    setMethod.invoke(B, getMethod.invoke(A, new Object[]{}));
}
```
使用反射实现，代码多了，但是好处是灵活性提高。不过并不建议这样使用，原因有二：

* 代码不够精简
* 性能不高

所以通常我们会借助第三方类库来实现。第三方类库有很多，以 commons.BeanUtils 为例：

```java
BeanUtils.setProperty(B, field, BeanUtils.getProperty(user, field));
```
可以看到，代码精简了许多。

### 高性能反射库 ReflectASM

关于反射，除了诸多优点，还有一个被人诟病的问题就是性能。不能否认，使用反射性能的确要慢一些，但是随着 Java 的不断优化，反射的速度也在不断提升。另外还是有一些反射库能帮助提升程序性能，比如 ReflectASM 。

ReflectASM 是通过字节码方式实现反射机制的反射库，写法比较简单，而且比 Java 自己的反射包和 commons.BeanUtils 等要快很多。

```
<dependency>
    <groupId>com.esotericsoftware.reflectasm</groupId>
    <artifactId>reflectasm</artifactId>
    <version>1.09</version>
</dependency>
```

```java
MethodAccess access = MethodAccess.get(User.class);
access.invoke(B, "setName", access.invoke(A, "getName"));
```

虽然 RefectASM 速度快很多，但是在多次使用到相同的对象还是应该利用缓存才能进一步提升性能。

```java
MethodAccess access = map.get("user");
if (access == null) {
    access = MethodAccess.get(User.class);
    map.put("user", access);
}
```
反射是一项非常有用的特性，能够在编程时解决很多特殊的问题，而性能会越来越快，所以应该合理大胆的使用反射。

---
补充：关于使用 ReflectASM 的问题
使用 ReflectASM 创建类的实例
```java
ConstructorAccess access = ConstructorAccess.get(User.class);
access.newInstance();
```
但是想调用带参数的构造器时，却没有找到方法。
