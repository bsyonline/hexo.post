---
title: IDEA + Maven + Jetty 热部署
toc: true
date: 2016-07-15 22:25:09
tags:
 - Meven
 - Jetty
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

####1. 集成

pom.xml

	<build>
	    <plugins>
	        <plugin>
	            <groupId>org.mortbay.jetty</groupId>
	            <artifactId>jetty-maven-plugin</artifactId>
	            <version>8.1.15.v20140411</version>
	            <configuration>
	                <connectors>
	                    <!-- 修改端口 -->
	                    <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
	                        <port>8080</port>
	                    </connector>
	                </connectors>
	                <stopKey>exit</stopKey>
	                <stopPort>9090</stopPort>
	                <scanIntervalSeconds>10</scanIntervalSeconds>
	                <webAppConfig>
	                    <!-- 热部署 -->
	                    <defaultsDescriptor>src/main/resources/webdefault.xml</defaultsDescriptor>
	                    <contextPath>/redis-doc</contextPath>
	                </webAppConfig>
	            </configuration>
	        </plugin>
	    </plugins>
	</build>
