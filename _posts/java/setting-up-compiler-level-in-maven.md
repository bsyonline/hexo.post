---
title: Maven 设置编译器等级
date: 2014-01-10 15:49:54
tags:
 - Maven
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "解决 IDEA 使用 maven 每次更新 pom 重置 LanguageLevel 和 JavaCompiler 问题的办法"
---


在pom.xml中加入

	<build>
	    <plugins>
	        <plugin>
	            <groupId>org.apache.maven.plugins</groupId>
	            <artifactId>maven-compiler-plugin</artifactId>
	            <version>2.3.2</version>
	            <configuration>
	                <source>1.7</source>
	                <target>1.7</target>
	            </configuration>
	        </plugin>
	    </plugins>
	</build>
