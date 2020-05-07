---
title: Java Request 的测试方法
date: 2016-10-17 14:28:17
tags:
 - Jmeter
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
Jmeter 是一款简单的性能测试工具，以前都是用来测试 API 接口，没试过测试 Java 程序，这是一个例子。

<!--more-->
Jmeter 使用，可参考 [Jmeter 使用入门](../../../../2016/09/07/Jmeter-使用入门/)。
![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/jmeter.png)

### java request 测试
使用 jmeter 测试 java 程序，需要结合 jemter_java 编写测试代码。
1. 创建 maven 工程
   pmx.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.rolex.jmeter</groupId>
    <artifactId>java-request-sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>attached</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <targetPath>libs/</targetPath>
                <directory>libs/</directory>
                <includes>
                    <include>**/certNoToMd5.jar</include>
                    <include>**/Lite-20111106.jar</include>
                    <include>**/quantum-auth-1.0-SNAPSHOT.jar</include>
                </includes>
            </resource>
        </resources>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.rolex.jmeter.test</groupId>
            <artifactId>jmeter-test</artifactId>
            <version>1.0.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/libs/certNoToMd5.jar</systemPath>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.jmeter/ApacheJMeter_core -->
        <dependency>
            <groupId>org.apache.jmeter</groupId>
            <artifactId>ApacheJMeter_core</artifactId>
            <version>3.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.jmeter/ApacheJMeter_java -->
        <dependency>
            <groupId>org.apache.jmeter</groupId>
            <artifactId>ApacheJMeter_java</artifactId>
            <version>3.0</version>
        </dependency>
    </dependencies>

</project>
```


```java
import org.apache.jmeter.config.Arguments;
import org.apache.jmeter.protocol.java.sampler.AbstractJavaSamplerClient;
import org.apache.jmeter.protocol.java.sampler.JavaSamplerContext;
import org.apache.jmeter.samplers.SampleResult;

import com.seeks.support.lxcernointerface.LxcernoClientUtils;

public class JavaRequest extends AbstractJavaSamplerClient {

	SampleResult result;
	String param;

	@Override
	public void setupTest(JavaSamplerContext context) {
		result = new SampleResult();
		super.setupTest(context);
	}

	@Override
	public Arguments getDefaultParameters() {
		Arguments params = new Arguments();
		params.addArgument("arg1", "0");
		return params;

	}

	@Override
	public SampleResult runTest(JavaSamplerContext arg0) {

		boolean success = true;
		result.sampleStart();
		param = arg0.getParameter("arg1");
		try {
			// do some test
		} catch (Exception e) {
			e.printStackTrace();
			success = false;
		} finally {
			result.sampleEnd();
			result.setSuccessful(success);
		}
		return result;

	}

	@Override
	public void teardownTest(JavaSamplerContext context) {
		super.teardownTest(context);
	}

}
```
![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/javaRequest.png)
