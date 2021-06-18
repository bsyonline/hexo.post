---
title: Mybatis Parameter Mapping
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 16:10:56
thumbnail:
---

通常我们在查询的时候会需要传递参数，参数的类型可以通过 parameterType 指定。
```
<select id="findByAge" parameterType="java.lang.Integer" resultType="com.rolex.microlabs.model.User">
	select * from t_user where age=#{age}
</select>
```
非简单类型的参数也是一样的。
```
<select id="findByAnyCondition" parameterType="com.rolex.microlabs.model.User"
            resultType="com.rolex.microlabs.model.User">
        select id, name, age, gender, skill from t_user
</select>	
```
parameterType 如果不指定，mybatis 会自动推断参数的类型。
在参数赋值时，如果查询接口的参数名字和 sql 中使用的占位符名称一样，会自动完成赋值。如果不一样，可以通过 @Param 指定。
```
List<User> findByNameAndAge(@Param("name") String arg1, @Param("age") Integer arg2);
```
大多数情况下，参数赋值都是使用 #{} 方式，这种方式类似 jdbc 中 PrepareStatement 的 ? 占位符。还有一种占位符是 ${} ，它只会进行字符串替换，不会进行预处理，只有在一些特殊场景才会用到。例如通过参数控制对哪一列进行 group by 。
```
<select id="groupByColumn" parameterType="java.lang.String" resultMap="groupByColumnResultMap">
	select age, count(*) as count from t_user group by ${columnName}
</select>
```
使用 ${} 占位符，mybatis 无法通过名字进行绑定，需要通过 @Param 来指定。
```
List<Map<Integer,Integer>> groupByColumn(@Param("columnName") String columnName);
```
