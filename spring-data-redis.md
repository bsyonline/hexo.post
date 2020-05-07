---
title: spring-data-redis示例说明
date: 2016-07-15 22:25:09
tags:
 - Spring
 - Redis
category: 
 - Spring
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---

###1. POM
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.6.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.6.2</version>
</dependency>
```

###2. 配置文件

####2.1 redis.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd">


    <context:property-placeholder location="classpath:config.properties"/>
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.cache.pool.maxTotal}" />
        <property name="maxIdle" value="${redis.cache.pool.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.cache.pool.maxWaitMillis}" />
        <property name="testOnBorrow" value="${redis.cache.pool.testOnBorrow}" />
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
        </property>
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"/>
        </property>
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="${redis.cache.ip}" />
        <property name="port" value="${redis.cache.port}" />
        <property name="usePool" value="${redis.cache.usePool}"/>
        <property name="poolConfig" ref="jedisPoolConfig" />
    </bean>

    <bean id="queryCacheRepository" class="com.daas.data.dao.repository.QueryCacheRepository"></bean>

</beans>
```
####2.2 properties文件

#redis配置

```
redis.cache.pool.maxTotal=1024
redis.cache.pool.maxIdle=200
redis.cache.pool.maxWaitMillis=1000
redis.cache.pool.testOnBorrow=true
redis.cache.ip=192.168.11.21
redis.cache.port=6379
redis.cache.usePool=true
```

###3. 类
####3.1 实体类

```java
/*
 * @(#)QueryCache.java	1.0 2016/3/8
 *
 */
package com.daas.data.engine.entity.redis;

import java.io.Serializable;

/**
 * Created with IntelliJ IDEA.
 * User: rolex
 * Date: 2016/3/8
 * version: 1.0
 */
public class QueryCache implements Serializable{

    String id;
    String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}
```

####3.2 查询类
```java
/*
 * @(#)QueryCacheRepository.java	1.0 2016/3/8
 *
 */
package com.daas.data.dao.repository;

import com.daas.data.engine.entity.redis.QueryCache;
import org.springframework.data.redis.core.RedisTemplate;

import javax.annotation.Resource;

/**
 * Created with IntelliJ IDEA.
 * User: rolex
 * Date: 2016/3/8
 * version: 1.0
 */
public class QueryCacheRepository {

    @Resource
    private RedisTemplate<String, QueryCache> redisTemplate;

    public void set(QueryCache qc){
        redisTemplate.opsForValue().set(qc.getId(), qc);

    }

    public QueryCache get(String id){
        return redisTemplate.opsForValue().get(id);
    }
}
```

####3.3 测试类
```java
/*
 * @(#)DataBusinessTest.java	1.0 2016/3/4
 *
 */
package com.daas.data.business;

import com.daas.data.dao.repository.QueryCacheRepository;
import com.daas.data.engine.entity.redis.QueryCache;
import org.elasticsearch.search.aggregations.InternalAggregation;
import org.hamcrest.core.IsEqual;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import java.io.IOException;

import static org.junit.Assert.assertThat;


/**
 * Created with IntelliJ IDEA.
 * User: rolex
 * Date: 2016/3/4
 * version: 1.0
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:redis-test.xml")
public class RedisServiceTest {


    @Resource
    QueryCacheRepository queryCacheRepository;

    @Before
    public void before() {
        QueryCache qc = new QueryCache();
        qc.setId("123");
        qc.setValue("test");
        queryCacheRepository.set(qc);
    }

    @Test
    public void testGet() throws IOException {
        QueryCache qc = queryCacheRepository.get("123");
        System.out.println(qc.getValue());
        assertThat(qc.getValue(), new IsEqual("test"));

    }

}
```