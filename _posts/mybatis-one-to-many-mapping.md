---
title: Mybatis One to Many Mapping
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-02-01 15:56:00
thumbnail:
---

mybatis 一对多映射通过 <collection> 标签来实现。
```
<mapper namespace="com.rolex.microlabs.dao.DepartmentDao">

    <resultMap id="deptResultMap" type="com.rolex.microlabs.model.Department">
        <id column="dept_id" property="deptId"></id>
        <result column="dept_name" property="deptName"></result>
        <collection property="employees" ofType="com.rolex.microlabs.model.Employee">
            <id column="id" property="id"></id>
            <result column="name" property="name"></result>
        </collection>

    </resultMap>

    <select id="findAll" resultMap="deptResultMap">
        select e.id, e.name, d.dept_id, d.dept_name from t_employee e, t_dept d where e.dept_id=d.dept_id
    </select>

</mapper>
```