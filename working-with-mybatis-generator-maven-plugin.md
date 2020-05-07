---
title: Working with Mybatis Generator Maven Plugin
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-29 09:16:46
thumbnail:
---

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