---
title: Mybatis Result Mapping
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 17:00:53
thumbnail:
---

对于查询结果，mybatis 通过 resultType 或是 resultMap 来进行映射。简单的类型使用 resultType ，比如基本类型或是结构简单的对象类型。对于一些复杂的结构，可以通过 resultMap 来组装，比如聚合查询的结果。
```
<resultMap id="groupByColumnResultMap" type="java.util.Map">
	<result property="age" column="age"></result>
	<result property="count" column="count"></result>
</resultMap>

<select id="groupByColumn" parameterType="java.lang.String" resultMap="groupByColumnResultMap">
	select age, count(*) as count from t_user group by ${columnName}
</select>
```
关联查询结果使用 &lt;association&gt; 和 &lt;collection&gt; 组装也是这种用法。
>在进行结果映射时，如果 column 和 property 名称相同，mybatis 会自动映射，如果不同，则不会自动映射。可以使用别名同一字段名称，或是使用 resultMap 。