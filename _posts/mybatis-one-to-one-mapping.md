---
title: Mybatis One to One Mapping
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 15:55:28
thumbnail:
---

mybatis 一对一映射通过 <association> 标签实现。
```
<mapper namespace="com.rolex.microlabs.dao.EmployeeDao">

    <resultMap id="employeeResultMap" type="com.rolex.microlabs.model.Employee">
        <id column="id" property="id"></id>
        <result column="name" property="name"></result>
        <association property="department" javaType="com.rolex.microlabs.model.Department">
            <id column="dept_id" property="deptId"></id>
            <result column="dept_name" property="deptName"></result>
        </association>

    </resultMap>

    <select id="findAll" resultMap="employeeResultMap">
        select e.id, e.name, d.dept_id, d.dept_name from t_employee e, t_dept d where e.dept_id=d.dept_id
    </select>

</mapper>
```