---
title: Mybatis in Action
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-29 07:59:40
thumbnail:
---



### 

### 执行流程

1. 实例化 org.mybatis.spring.SqlSessionFactoryBean 。
2. 解析 xml 构造 Configuration 和 Context 。
   2.1 将 statement ，resultMap 等信息加入到 context 。
   2.2 将 sql 进行绑定到 SqlSource 。
   2.3 将 statement 信息构造成 MappedStatement ，放到 Configuration 。
3. 创建查询接口 mapper 的代理对象 org.apache.ibatis.binding.MapperProxy ，通过“包名+类名+方法名”从 Map 中取出对应的 sql 语句执行。

### Springboot 集成 Mybatis

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



### 缓存

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



### 参数映射

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



### 结果集映射

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





### 动态SQL

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



### 映射

#### 一对多

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

#### 一对一

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





### 使用注解写 SQL

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





### 分页插件 PageHelper 使用

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

### mybatis-generator-maven-plugin 插件使用

除了自己写实体类，接口和映射之外，还可以使用 mybatis-generator-maven-plugin 来自动生成。

1. 添加 maven plugin 配置。

```
<build>
	<plugins>
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.5</version>
			<dependencies>
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.5</version>
				</dependency>
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>5.1.34</version>
				</dependency>
			</dependencies>
			<executions>
				<execution>
					<id>mybatis-generator</id>
					<phase>package</phase>
					<goals>
						<goal>generate</goal>
					</goals>
				</execution>
			</executions>
			<configuration>
				<!-- 允许移动生成的文件 -->
				<verbose>true</verbose>
				<!-- 允许覆盖，生产环境应设置成false -->
				<overwrite>true</overwrite>
				<configurationFile>
					src/main/resources/mybatis-generator.xml
				</configurationFile>
			</configuration>
		</plugin>
	</plugins>
</build>
```

2. 修改 src/main/resources/mybatis-generator.xml 。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!--实体类配置-->
        <javaModelGenerator targetPackage="generator.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--查询接口配置-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/java/generator">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--mapper映射配置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="generator.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!--数据库表和实体映射配置-->
        <table schema="test" tableName="t_user" domainObjectName="User"
               enableCountByExample="true"
               enableUpdateByExample="true"
               enableDeleteByExample="true"
               enableSelectByExample="true"
               selectByExampleQueryId="true">
        </table>
    </context>
</generatorConfiguration>
```

3. 运行插件。
   执行命令 mvn mybatis-generator:generate 即可在对应目录生成相应文件。




