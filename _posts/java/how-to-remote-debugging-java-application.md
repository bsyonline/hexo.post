---
title: How to Remote Debugging Java Application
date: 2021-01-16 16:34:40
tags:
---





Java Platform Debugging Architecture (JPDA) 提供了一组 API ，可以通过 Java Debug Wire Protocol (JDWP) 协议进行远程调试。

先看下 JPDA 术语：

debugger： 调试者。

debuggee：被调试者。

JVM TI ： JavaVM Tool Interface，定义了一个虚拟机提供的调试服务。

JDI ： Java Debug Interface，高级的 Java 语言接口，开发者可以通过这些接口编写远程调试程序。

JDWP : Java Debug Wire Protocol，定义了 debuggee 和 debugger 之间的通信协议。

JDB : Java 提供的远程调试工具。

#### 使用 JDB 远程调试 Java App

加入我们需要远程调试一个 HelloWorld.java 程序。

```java
class HelloWorld {
    public static void main(String[] args) {
        int a = 1;
        int b = 2;
        int c = a + b;
        System.out.println(c);
    }
}
```

首先我们需要编译。

```shell
javac -g HelloWorld.java
```

然后在调试模式下运行 HelloWorld 。

```shell
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 HelloWorld
```

接着使用 JDB 连接到服务器，进入调试模式。

```shell
$ jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=5005
Picked up JAVA_TOOL_OPTIONS: -Duser.language
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
Initializing jdb ...
>
VM Started: No frames on the current call stack
```

设置断点。

```shell
main[1] stop in HelloWorld:6
Deferring breakpoint HelloWorld:6.
It will be set after the class is loaded.
main[1] run
> Set deferred breakpoint HelloWorld:6

Breakpoint hit: "thread=main", HelloWorld.main(), line=6 bci=8
```

接下来就可以使用 jdb 命令进行调试了。

```shell
main[1] print a
 a = 1
main[1] print b
 b = 2
main[1] print c
 c = 3
main[1] run
>
The application exited
```

#### 使用 IDEA 远程调试 Web App

这一节我们通过一个 springboot 程序来介绍如何使用 IDEA 进行远程调试。

首先我们需要准备一个 springboot 程序，编译打包，然后启动。

```shell
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 -jar remote-debug-demo-1.0-SNAPSHOT.jar
```

也是使用 jdwp ，server=y 说明是作为被调试的程序，以前我们使用 suspend=y 是将程序阻塞，当调试启动时在启动程序，这里由于是 springboot 程序，我们要调试的是程序接口，所以suspend=y ，让程序先启动。

第二步，我们需要在 IDEA 中创建一个 remote 类型的 Configuration ，监听端口和调试端口保持一致为 5005 ，host 填 jar 程序所在的机器 ip ，创建好后保存。

第三步，就可以在接口代码中设置断点，启动调试了。

> 本地代码应于调试的代码保持一致

#### 使用 JDB 调试 Web App

JDB 调试 Web App 方式和之前类似。首先需要启动 debugger server 。

```shell
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 -jar remote-debug-demo-1.0-SNAPSHOT.jar
```

 第二步，使用 JDB 命令连接到 debugger server 。

```shell
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=5005
```

第三步，设置断点。

```shell
$ jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=5005
Picked up JAVA_TOOL_OPTIONS: -Duser.language
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
Initializing jdb ...
> stop in com.rolex.alphax.remote.debug.controller.HelloController:17
Set breakpoint com.rolex.alphax.remote.debug.controller.HelloController:17
```

再访问接口，http://localhost:8099/hello?name=john 程序会执行到断点位置。

```shell
Breakpoint hit: "thread=http-nio-8099-exec-5", com.rolex.alphax.remote.debug.controller.HelloController.hello(), line=17 bci=0
```

接下来就可以调试了。

```shell
http-nio-8099-exec-5[1] print name
 name = "john"
http-nio-8099-exec-5[1] next
>
Step completed: "thread=http-nio-8099-exec-5", org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(), line=190 bci=19

http-nio-8099-exec-5[1] next
>
Step completed: "thread=http-nio-8099-exec-5", org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(), line=138 bci=59

http-nio-8099-exec-5[1] next
>
Step completed: "thread=http-nio-8099-exec-5", org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(), line=105 bci=7

http-nio-8099-exec-5[1] next
>
Step completed: "thread=http-nio-8099-exec-5", org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(), line=106 bci=9

http-nio-8099-exec-5[1] step up
>
Step completed: "thread=http-nio-8099-exec-5", org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(), line=894 bci=247

http-nio-8099-exec-5[1]
```

输入 run 结束本次调试。

```shell
http-nio-8099-exec-5[1] run
>
```

#### 使用 JDPA 编程接口

除了使用 JDB 和 IDEA ，还可以使用 JDI 提供的编程接口进行远程 debug 。

通过 socket 连接到 debugger server 。

```java
List<AttachingConnector> connectors = Bootstrap.virtualMachineManager().attachingConnectors();
SocketAttachingConnector socketAttachingConnector = null;
for (AttachingConnector ac : connectors) {
    if (ac instanceof SocketAttachingConnector) {
        socketAttachingConnector = (SocketAttachingConnector) ac;
        break;
    }
}
if (socketAttachingConnector == null) {
    throw new IllegalStateException("Could not find socket connector");
}
Map<String, Connector.Argument> defaultArguments = socketAttachingConnector.defaultArguments();
defaultArguments.get("port").setValue("5555");
vm = socketAttachingConnector.attach(defaultArguments);
```

通过事件处理完成调试，向 EventQueue 中注册事件。

```java
System.out.println("registerEvent -> 注册ClassPrepareRequest事件");
ClassPrepareRequest classPrepareRequest = eventRequestManager.createClassPrepareRequest();
classPrepareRequest.addClassFilter(className);
classPrepareRequest.addCountFilter(1);
classPrepareRequest.setSuspendPolicy(EventRequest.SUSPEND_ALL);
classPrepareRequest.enable();
```

注册一个 断点事件。

```java
ClassPrepareEvent evt = (ClassPrepareEvent) event;
ClassType classType = (ClassType) evt.referenceType();
System.out.printf("eventLoop -> 添加断点：HelloWorld(L%d)\n", 6);
Location location = classType.locationsOfLine(6).get(0);
System.out.println("eventLoop -> 注册ClassPrepareRequest事件");
BreakpointRequest breakpointRequest = eventRequestManager.createBreakpointRequest(location);
breakpointRequest.setSuspendPolicy(EventRequest.SUSPEND_EVENT_THREAD);
breakpointRequest.enable();
```

遍历 EventQueue 处理事件。

```java
eventQueue = vm.eventQueue();
while (true) {
    if (vmExit == true) {
        System.out.println("eventLoop -> vmexit");
        break;
    }
    eventSet = eventQueue.remove();
    EventIterator eventIterator = eventSet.eventIterator();
    while (eventIterator.hasNext()) {
        Event event = (Event) eventIterator.next();
        execute(event);
        if (!vmExit) {
            eventSet.resume();
        }
    }
}
```

#### 