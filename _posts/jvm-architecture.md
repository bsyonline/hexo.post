---
title: JVM Architecture
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2018-08-13 19:12:30
thumbnail:
---


Java Virtual Mechine （JVM）包含三部分，分别是：类加载器子系统、运行时数据区和执行引擎。

<img src="https://s2.ax1x.com/2020/03/11/8A2ZHH.png" alt="8A2ZHH.png" border="0" style="width:500px">

#### **类加载子系统**
类加载是将 ```.class``` 文件加载到虚拟机内存并创建对象的整个过程。类加载子系统提供 Java 的类动态加载功能，它有三个主要阶段：加载、链接和初始化。  

**加载**

加载是 JVM 通过一个类的完全限定名来获取类的二进制字节流，将字节流代表的静态存储结构转化为方法区中的运行时数据结构，然后在内存中生成一个代表这个类的 Class 对象作为访问入口的过程。
在 Java 中有多种途径可以获取类的二进制字节流：
1. 从 zip 中获取，jar ，ear ，war 都属于这类。
2. 从网络中获取，比如 applet 。
3. 通过动态代理生成。
4. 通过文件生成，比如 jsp 被编译成 class 。
5. 等等。

加载既可以通过系统提供的类加载器加载，也可以通过自定义的类加载器加载。系统提供的类加载器有 3 个：Bootstrap ClassLoader 、Extention ClassLoader 和 Application ClassLoader 。
1. Bootstrap ClassLoader 负责加载 \lib 下的类。
2. Extention ClassLoader 负责加载 \lib\ext 下的类。
3. Application ClassLoader 负责加载应用程序级别的类，即系统类路径 ```classpath``` 下的类。

类加载器在加载类时使用委托算法。委托算法的原理是：如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载。
>除了顶层的类加载器，每个类加载器都要有父类加载器，这些加载器之间并不是继承关系，而是组合关系。

使用这种委托算法的优点有二：
1. 防止重复加载，父加载器加载过的类子类加载器就不用再加载一次
2. 安全，Java API 定义的类不会被篡改，因为父类已经加载了 Java 核心类，如果我们要加载自己修改的核心类，父加载器将不会加载。

**链接**

链接包含三个步骤：验证、准备和解析。
1. 验证
验证的目的在于确保 Class 文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。虽然一些非法的操作（比如将一个类型转成一个不存在的类型，或者将代码跳转到不存在行）在编译期间编译器拒绝编译，但是 JVM 并没有限制字节码一定要由 java 编译器编译生成，所以理论上甚至可以通过直接编写生成 class 文件，所以验证是非常重要的。
验证包括四种：
&nbsp;&nbsp;&nbsp;&nbsp;1) 文件格式验证，验证是否符合 class 文件格式规范。
&nbsp;&nbsp;&nbsp;&nbsp;2) 元数据验证，对元数据信息进行语义分析，比如是否有父类，是否可以继承等。
&nbsp;&nbsp;&nbsp;&nbsp;3) 字节码验证，通过数据流和控制流分析程序的合法性，比如在栈上放一个 int 类型的数据，使用时按 long 类型加载。
&nbsp;&nbsp;&nbsp;&nbsp;4) 符号引用验证，对类自身信息以外的信息进行校验，比如常量池。
2. 准备
准备是为类变量（即 ```static``` 修饰的字段变量）分配内存并且设置类变量的初始值的阶段。例如 ```static int i=5;``` 这里只将 i 初始化为 0 ，至于 5 的值将在初始化时赋值。这里不包含用 ```final``` 修饰的 ```static```，因为 ```final``` 在编译的时候就会分配了，注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到 Java 堆中。
3. 解析
解析是将常量池中的符号引用替换为直接引用的过程。**符号引用**就是一组符号来描述目标，可以是任何字面量，而**直接引用**就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
>符号引用转化成直接引用分为两种：在解析阶段转化叫静态解析，运行期转化叫动态连接。

**初始化**

初始化是类加载的最后阶段，将为所有静态变量分配原始值，并执行静态块。
在 JVM 规范中规定，有且只有 5 种情况必须进行初始化：
1. 遇到 new ，getstatic ，putstatic 和 invokestatic 4 个字节码指令时如果没有初始化，则必须进行初始化。也就是在使用 new 实例化对象时，读写类的静态变量时，以及调用静态方法时。
2. 使用反射调用时。
3. 在初始化一个类且它的父类还没有被初始化时。
4. 虚拟机启动执行主类时。
5. 使用 MethodHandle 解析 REF_getStatic ，REF_putStatic 和 REF_invokeStatic 的方法句柄时。

#### **运行时数据区**

**程序寄存器（Program Counter）**
线程私有。
存储当前线程执行程序的字节码指令的行号。
JVM 未规定异常。
**虚拟机栈（Java Vitrual Machine Stacks）**
线程私有。
存储使用栈帧来保存局部变量表、操作数栈、动态链接、方法出口等。

JVM 规定了两种异常：StackOutOfMemoryError 和 OutOfMemoryError 。
**本地方法栈（Native Method Stack）**
本地方法栈和虚拟机栈类似，区别是本地方法栈用于执行本地方法。
**堆（Java Heap）**
堆是共享的，是 Java 内存最大的区域。Java 内存回收主要发生在堆，按照分代收集算法进行内存回收。堆内存可以细分为新生代和老年代。新生代又分为 Eden 、From Survivor 和 To Survivor 。堆是逻辑连续的，物理可以不连续。
JVM 规定的异常为 OutOfMemoryError 。
**方法区（Method Area）**
线程共享的。
存储已被 JVM 加载的类信息、常量、静态变量、及时编译器编译后的代码。
JVM 规定的异常为 OutOfMemoryError 。


#### **执行引擎**
执行引擎用来执行分配给运行时数据区的字节码。
执行引擎分为：解释器、JIT 编译器和垃圾回收器。
**解释器**
解释会读取字节码，解释并逐行执行。解释器解释的速度很快但是执行速度很慢，而且一个方法被调用多次，解释过程也会进行多次。
**JIT 编译器**
JIT 编译器使用解释器进行字节码转换，遇到重复的代码，JIT 编译器会将整个字节码编译成机器码。机器码将用于重复方法的调用，从而提高性能。
**垃圾回收器**
垃圾回收器用于收集 ```new``` 产生的对象。垃圾回收可以通过 ```System.gc()``` 来触发，但是不一定会立刻执行。不是用 ```new``` 生成的对象可以使用 ```finalize()``` 进行回收。


