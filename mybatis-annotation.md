---
title: Mybatis Annotation
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-30 21:24:52
thumbnail:
---

除了使用 mapper 文件之外，还可以使用注解来写 sql 。
```
@Mapper // 和@MapperScan二选一
public interface UserDao {

    @Insert("insert into t_user (name, age, gender, skill) values (#{name}, #{age}, #{gender}, #{skill})")
    int save(User user);

    @Select("select id, name, age, gender, skill from t_user")
    List<User> findAll();

}
```
使用注解就可以不用 xml 的相关配置了。