---
title: Guide to Java Agent
date: 2020-08-25 09:31:02
tags:
---



```
public class MyAgent {
    public static void premain(String args, Instrumentation instrumentation) {
        System.out.println("premain start");
        System.out.println(args);
    }

    public static void premain(String args) {
        System.out.println("premain start");
        System.out.println(args);
    }
}
```

/METE-INF/MANIFEST.MF

```
Manifest-Version: 1.0
Premain-Class: com.rolex.microlabs.agent.MyAgent
```

使用 maven 打包，防止 maven 生成的 MANIFEST.MF 覆盖我们自己的。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <archive>
                    <manifest>
                        <addClasspath>true</addClasspath>
                    </manifest>
                    <manifestEntries>
                        <Premain-Class>
                            com.rolex.microlabs.agent.MyAgent
                        </Premain-Class>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```







```
java -javaagent:D:\MyAgent-1.0.jar=helloworld test.Test
```

