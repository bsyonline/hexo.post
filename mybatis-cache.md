---
title: Mybatis Cache
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 11:50:31
thumbnail:
---

mybatis 有一级缓存和二级缓存，通过 mybatis 官方文档我们知道默认情况下，mybatis 的一级缓存是默认开启的。如果我们在 springboot 中集成 mybatis ，我们在同一方法中使用两次查询，会发现 mybatis 会执行两次查询。
```
@Test
public void testCache(){
	List<User> users = userDao.findAll();
	System.out.println(users);
	List<User> users1 = userDao.findAll();
	System.out.println(users1);
}
``` 
打印 sql 。
```
2020-02-01 11:57:17.958 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==>  Preparing: select id, name, age, gender, skill from t_user 
2020-02-01 11:57:18.012 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==> Parameters: 
2020-02-01 11:57:18.049 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : <==      Total: 4
[User(id=3, name=alice, age=29, gender=Female, skill=Java), User(id=4, name=jim, age=29, gender=Male, skill=CPP), User(id=5, name=alice, age=19, gender=Female, skill=Java), User(id=6, name=jim, age=20, gender=Male, skill=CPP)]
2020-02-01 11:57:18.052 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==>  Preparing: select id, name, age, gender, skill from t_user 
2020-02-01 11:57:18.053 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==> Parameters: 
2020-02-01 11:57:18.056 DEBUG 2680 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : <==      Total: 4
[User(id=3, name=alice, age=29, gender=Female, skill=Java), User(id=4, name=jim, age=29, gender=Male, skill=CPP), User(id=5, name=alice, age=19, gender=Female, skill=Java), User(id=6, name=jim, age=20, gender=Male, skill=CPP)]
```
很显然一级缓存失效了。
我们来分析一下缓存失效的原因。我们在使用 springboot 集成 mybatis 时，从始至终都没有发现 mybatis 的一个重要的对象 SqlSession 。原因是因为 spring 在集成 mybatis 时，将 SqlSession 进行了封装，而在使用时通过代理对象创建。所以我们没有办法直接操作 SqlSession 。**正是因为 SqlSession 完全由 spring 容器管理，所以 spring 在 SqlSessionTemplate.SqlSessionInterceptor.invoke() 方法中每次执行完 invoke() 方法后，都在 finally 中将 SqlSession 关闭了，所以一级缓存就失效了。**

不过我们还可以通过配置使用二级缓存。在 UserMapper.xml 中加入 <cache/> 配置，我们会发现第二次查询使用了缓存。
```
2020-02-01 12:35:35.413 DEBUG 20624 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==>  Preparing: select id, name, age, gender, skill from t_user 
2020-02-01 12:35:35.463 DEBUG 20624 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : ==> Parameters: 
2020-02-01 12:35:35.517 DEBUG 20624 --- [           main] com.rolex.microlabs.dao.UserDao.findAll  : <==      Total: 4
[User(id=3, name=alice, age=29, gender=Female, skill=Java), User(id=4, name=jim, age=29, gender=Male, skill=CPP), User(id=5, name=alice, age=19, gender=Female, skill=Java), User(id=6, name=jim, age=20, gender=Male, skill=CPP)]
2020-02-01 12:35:35.536 DEBUG 20624 --- [           main] com.rolex.microlabs.dao.UserDao          : Cache Hit Ratio [com.rolex.microlabs.dao.UserDao]: 0.5
[User(id=3, name=alice, age=29, gender=Female, skill=Java), User(id=4, name=jim, age=29, gender=Male, skill=CPP), User(id=5, name=alice, age=19, gender=Female, skill=Java), User(id=6, name=jim, age=20, gender=Male, skill=CPP)]
```
>缓存对象还需要实现序列化接口。

mybatis 的二级缓存是 namespace 级别的缓存，同时在进行增删改操作之后会自动更新缓存。但是由于有 namespace 的限制，在一个 namespaceA 中修改了另一个 namespaceB 中的对象，那 namespaB 中的缓存是不会自动更新的，这就导致数据不一致，在多表操作时应该注意。
>除非能够确定对象不会被别的 namespace 修改，否则不要使用二级缓存。通常情况下，应禁用 mybatis 的缓存，可以使用 redis 这样的外部缓存。