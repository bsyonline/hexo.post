---
title: Working with Mybatis
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-29 07:59:40
thumbnail:
---

集成 spring boot 和 Mybatis 非常简单，只需要 6 步即可完成。
1. 添加 maven 依赖
```
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.0</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

2. 按照数据库结构创建对象。
假设我们有一张用户表。
```
create table t_user
(
    id   int auto_increment primary key,
    name varchar(50) not null,
    age  int         null
);
```
对应的实体对象 com.rolex.microlabs.model.User 。
```
public class User {
    private int id;
    private String name;
    private int age;
	//getter and setter
}
```
3. 添加查询接口 com.rolex.microlabs.dao.UserDao 。
```
@Mapper // 和@MapperScan二选一
public interface UserDao {
    int save(User user);
}
```
4. 编写对应的 mapper 映射。
创建 resources/mapper/UserMapper.xml 文件，添加如下内容。
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.rolex.microlabs.dao.UserDao">
    <insert id="save" parameterType="com.rolex.microlabs.model.User">
        insert into t_user
          (name, age)
        values
          (#{name}, #{age})
    </insert>
</mapper>
```
5. 修改配置文件。
在 application.yml 中添加 mybatis 配置信息。
```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
mybatis:
  mapper-locations: classpath:mapper/*.xml  #注意：一定要对应mapper映射xml文件的所在路径
```
6. 配置 spring boot 启动类。
```
@SpringBootApplication
@MapperScan("com.rolex.microlabs.dao") // 和@Mapper二选一
public class MyBatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyBatisApplication.class, args);
    }
}
```

完成以上配置就可以使用 JUnit 进行测试了。










