---
title: spring boot cache （redis）使用
date: 2016-05-26 16:00:24
tags:
 - Spring Boot
category: 
 - Spring Boot
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. pom.xml
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
### 2. cachemanager

```
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.core.RedisTemplate;

import java.lang.reflect.Method;

@Configuration
@EnableCaching
public class RedisCache {
    @Bean
    public CacheManager cacheManager(
            @SuppressWarnings("rawtypes") RedisTemplate redisTemplate) {
        return new RedisCacheManager(redisTemplate);
    }

    /**
     * 自定义key的生成策略
     * @return
     */
    @Bean
    public KeyGenerator myKeyGenerator(){
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }
}
```

### 3. 方法上添加注解
```
@Cacheable(value = "entInfo", keyGenerator = "myKeyGenerator")
public EntMultipleInfo findEntInfo(String entName) {
    looger.info("no cache");
}
```
>在调用方法前会先去查询是否有缓存。
>对象需要能够序列化

### 4. application.properties
```
spring.redis.database= 3
spring.redis.host=192.168.11.21
spring.redis.port=6379
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
```
