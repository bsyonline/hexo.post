---
title: 枚举单例读取配置文件
date: 2016-08-02 23:29:14
tags:
 - Design Patterns
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

枚举实现单例在《Effective in java》中提到过，好处是简洁，同时不会有序列化和反序列化的问题。
<!--more-->
下面一个 enum 单例的实际示例，用来读取配置文件。

**config.properties**
```java
host=htt://localhost
port=8080
```

**AppContext.java**
```java
import java.util.ResourceBundle;

public enum  AppContext {

    INSTANCE;

    private volatile static ResourceBundle rb = ResourceBundle.getBundle("config");

    public String getValue(String key){
        return rb.getString(key);
    }
}
``

**client**
​```java
public class Client {

    public static void main(String[] args) {
        String host = AppContext.INSTANCE.getValue("host");
    }
}
```
