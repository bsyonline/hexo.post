---
title: Mapping Enum Type
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-30 17:14:06
thumbnail:
---

我们可以使用自定义 typeHandler 来处理 enum 类型的映射。在实体 User 中有两个 enum 类型的属性。
```
public class User {

    private int id;
    private String name;
    private int age;
    private Gender gender;
    private Skill skill;
	// getter and setter
}
```
数据库中分别为 tinyint 类型和 varchar ，我们可以通过 typeHandler 来进行映射。
1. 创建 enum 类型的 typeHandler。
有两种方式来创建自定义的 typeHandler ，实现 TypeHandler<T> 和继承 BaseTypeHandler<T> 。
```
@MappedTypes({Gender.class})
public class GenderHandler implements TypeHandler<Gender> {
    @Override
    public void setParameter(PreparedStatement preparedStatement, int i, Gender gender, JdbcType jdbcType) throws SQLException {
        preparedStatement.setInt(i, gender.getValue());
    }

    @Override
    public Gender getResult(ResultSet resultSet, String s) throws SQLException {
        return Gender.nameOf(resultSet.getInt(s));
    }

    @Override
    public Gender getResult(ResultSet resultSet, int i) throws SQLException {
        return Gender.nameOf(resultSet.getInt(i));
    }

    @Override
    public Gender getResult(CallableStatement callableStatement, int i) throws SQLException {
        return Gender.nameOf(callableStatement.getInt(i));
    }
}
```
```
@MappedTypes({Skill.class})
public class SkillHandler extends BaseTypeHandler<Skill> {

    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Skill skill, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i, skill.getValue());
    }

    @Override
    public Skill getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return Skill.nameOf(resultSet.getString(s));
    }

    @Override
    public Skill getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return Skill.nameOf(resultSet.getString(i));
    }

    @Override
    public Skill getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return Skill.nameOf(callableStatement.getString(i));
    }
}
```
自定义 typeHandler 上要加上 @MappedTypes 。
2. 在配置文件中加入 typeHandler 的配置。
```
mybatis:
  type-handlers-package: com.rolex.microlabs.handler
```
这样就可以在 insert 和 select 的时候完成 enum 类型的映射了。