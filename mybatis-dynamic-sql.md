---
title: Mybatis Dynamic SQL
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-31 12:52:13
thumbnail:
---

通常在通过组合若干条件进行数据库操作时，需要根据条件组装 sql ，可以使用 &lt;if&gt; 标签。
```
<select id="findByCondition" parameterType="com.rolex.microlabs.model.User"
            resultType="com.rolex.microlabs.model.User">
	select id, name, age, gender, skill from t_user
	<where>
		<if test="name != null">and name=#{name}</if>
		<if test="age != null">and age=#{age}</if>
		<if test="gender != null">and gender=#{gender}</if>
	</where>
</select>
```
还有一种类似 switch 功能的标签。
```
<select id="findByAnyCondition" parameterType="com.rolex.microlabs.model.User"
            resultType="com.rolex.microlabs.model.User">
	select id, name, age, gender, skill from t_user
	<where>
		<choose>
			<when test="name != null">and name=#{name}</when>
			<when test="age != null">and age=#{age}</when>
			<when test="gender != null">and gender=#{gender}</when>
		</choose>
	</where>
</select>
```
更新操作也可以使用 &lt;if&gt; 标签，可用于更新有值的字段。
```
<update id="update" parameterType="com.rolex.microlabs.model.User">
	update t_user
	<set>
		<if test="name != null">name=#{name},</if>
		<if test="age != null">age=#{age},</if>
		<if test="gender != null">gender=#{gender}</if>
	</set>
	where id=#{id}
</update>
```
批量操作时可以使用 &lt;foreach&gt; 标签。
```
<insert id="batchSave">
	insert into t_user
	(name, age, gender, skill)
	values
	<foreach collection="list" item="user" index="index" separator=",">
		(#{user.name}, #{user.age}, #{user.gender}, #{user.skill} )
	</foreach>
</insert>
<select id="batchQuery" resultType="com.rolex.microlabs.model.User">
	select name, age, gender, skill from t_user
	where id in
	<foreach collection="list" item="id" index="index" open="(" close=")" separator=",">
		#{id}
	</foreach>
</select>
<delete id="batchDelete">
	delete from t_user where id in
	<foreach collection="list" item="id" index="index" open="(" close=")" separator=",">
		#{id}
	</foreach>
</delete>
<update id="batchUpdate">
	<foreach collection="list" item="user" index="index" separator=";">
		update t_user
		<set>
			<if test="user.name != null">name=#{user.name},</if>
			<if test="user.age != null">age=#{user.age},</if>
			<if test="user.gender != null">gender=#{user.gender}</if>
		</set>
		where id=#{user.id}
	</foreach>
</update>
```




