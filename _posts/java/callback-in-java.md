---
title: Java 中的回调函数
date: 2016-08-08 17:23:33
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

在 Spring 和 Hibernate 中有很多回调函数的用法，感觉使用起来非常方便，那么回调函数是如何实现的呢？

首先需要定义一个回调接口，比如：
```java
public interface QueryCallback<T> {

    <T> List<T> query(Session session);

}
```
然后在定义的方法是调用接口方法。
```java
public class Query {

    public List<String> queryForList(String sql, Callback callback){
        Session session = new Session();
        return callback.query(session);
    }

}
```
调用的时候就可以使用回调了。
```java
String sql = "select * from user";
List<String> list = new QueryDB().queryForList(sql, new Callback() {
    @Override
    public List query(Session session) {
        List<String> list = session.query(sql);
        return list;
    }
});
```
