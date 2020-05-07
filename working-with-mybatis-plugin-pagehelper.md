---
title: Working with Mybatis Plugin Pagehelper
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 11:50:10
thumbnail:
---

mybatis 分页插件 PageHelper 可以非常方便的进行物理分页。
添加 spring-boot-starter 依赖。
```
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.2.13</version>
</dependency>
```
UserMapper.xml 不需要额外修改。
```
<select id="listForPage" resultType="com.rolex.microlabs.model.User">
	select * from t_user
</select>
```
在需要分页的查询前加入分页代码。
```
@Test
public void listForPage(){
	PageHelper.startPage(2, 5);
	List<User> list = userDao.listForPage();
	PageInfo page = new PageInfo(list);
	//测试PageInfo全部属性
	//PageInfo包含了非常全面的分页属性
	assertEquals(2, page.getPageNum());
	assertEquals(5, page.getPageSize());
	assertEquals(6, page.getStartRow());
	assertEquals(10, page.getEndRow());
	assertEquals(100, page.getTotal());
	assertEquals(20, page.getPages());
	assertEquals(true, page.isHasPreviousPage());
	assertEquals(true, page.isHasNextPage());
	System.out.println(page.getList());
}
```
使用 sprin boot + mybatis + PageHelper 如果有特殊需求，可以在配置文件中进行配置，比如分页合理化参数。
```
pagehelper:
  reasonable: true # 合理化参数
```