---
title: Deep in Overload and Override
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-03-29 11:04:46
thumbnail:
---

封装，继承，多态是面向对象的 3 个特征。多态又可以分为静态多态 (overload) 和动态多态 (override) 。
#### Overload
Overload 通过参数类型来选择方法，我们通过一个例子来感受以下。
```
public class OverloadExample {

    public void sayHello(Human guy) {
        System.out.println("hello, guy");
    }

    public void sayHello(Man guy) {
        System.out.println("hello, gentleman");
    }

    public void sayHello(Woman guy) {
        System.out.println("hello, lady");
    }

    public static void main(String[] args) {
        Human man = new Man(); // 静态类型是Human，动态类型是Man
        Human woman = new Woman();
        OverloadExample oe = new OverloadExample();
        /*
            重载是根据静态类型来判断的
         */
        oe.sayHello(man);
        oe.sayHello(woman);
    }

    static class Human {
    }

    static class Man extends Human {
    }

    static class Woman extends Human {
    }
}

运行结果：
hello, guy
hello, guy
hello, gentleman
hello, lady
```
接下来我们来分析以下为什么。```Human man = new Man();``` Human 是静态类型，Man 是实际类型，变量本身的静态类型不会被改变，所以是编译期可知的，而实际类型是在运行时确定的。编译器在重载时是通过参数的静态类型来判断的。通过 javap 可以看到编译的字节码。
```
 #13 = Methodref          #11.#57        // com/rolex/microlabs/jvm/OverloadExample.sayHello:(Lcom/rolex/microlabs/jvm/OverloadExample$Human;)V

 26: invokevirtual #13                 // Method sayHello:(Lcom/rolex/microlabs/jvm/OverloadExample$Human;)V
 29: aload_3
 30: aload_2
 31: invokevirtual #13                 // Method sayHello:(Lcom/rolex/microlabs/jvm/OverloadExample$Human;)V
```
这种通过静态类型来寻找方法的方式叫做**静态分派**，Overload 就是典型场景。静态类型可能会匹配多个方法，这时编译器会选择一个最合适。比如下面这个例子。
```
public class OverloadExample1 {
    public static void sayHello(char arg) {
        System.out.println("hello char");
    }

    public static void sayHello(int arg) {
        System.out.println("hello int");
    }

    public static void sayHello(long arg) {
        System.out.println("hello long");
    }

    public static void sayHello(float arg) {
        System.out.println("hello float");
    }

    public static void sayHello(double arg) {
        System.out.println("hello double");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void sayHello(Comparable arg) {
        System.out.println("hello Comparable");
    }

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(char... arg) {
        System.out.println("hello char...");
    }

    public static void sayHello(int... arg) {
        System.out.println("hello int...");
    }

    public static void main(String[] args) {
        sayHello('a');
    }
}
```
根据 ```'a'``` 的类型，最匹配的当然是 char 。如果没有 char ，则会匹配 int ，因为 char 通过类型转化，也可以用 int 表示。如果 int 也没有，则再进行一次类型转化，来匹配 long 。依次类推，可以依次匹配 float 和 double 。如果 float 和 double 都没有，则会匹配 Character ，因为 Character 是 char 的包装类，通过自动装箱可以进行类型转化。如果 Character 也么有，则会匹配 Serializable 和 Comparable ，因为 Character 实现了 Serializable 和 Comparable ，它们的优先级是相同的，编译器就不知道该如何匹配了，会报编译错误。如果没有 Serializable 和 Comparable ，则会继续匹配 Object 因为 Object 是 Character 的父类。最后如果 Object 也没有，就会作为一个可变参数进行匹配。

#### Override

```
public class OverrideExample {
    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        Man man1 = new Man();
        Woman woman1 = new Woman();
        man.sayHello();
        woman.sayHello();
        man1.sayHello();
        woman1.sayHello();
    }

    static class Human {
        public void sayHello() {
            System.out.println("hello, guy");
        }
    }

    static class Man extends Human {
        public void sayHello() {
            System.out.println("hello, gentleman");
        }
    }

    static class Woman extends Human {
        public void sayHello() {
            System.out.println("hello, lady");
        }
    }
}

执行结果：
hello, gentleman
hello, lady
hello, gentleman
hello, lady
```
在编译 ```man.sayHello()``` 时，编译器并不知道最终是调用 Man 的 sayHello() ，我们可以通过 javap 查看。
```
#6 = Methodref          #14.#39        // com/rolex/microlabs/jvm/OverrideExample$Human.sayHello:()V
#7 = Methodref          #2.#39         // com/rolex/microlabs/jvm/OverrideExample$Man.sayHello:()V
#8 = Methodref          #4.#39         // com/rolex/microlabs/jvm/OverrideExample$Woman.sayHello:()V

34: invokevirtual #6                  // Method com/rolex/microlabs/jvm/OverrideExample$Human.sayHello:()V
37: aload_2
38: invokevirtual #6                  // Method com/rolex/microlabs/jvm/OverrideExample$Human.sayHello:()V
41: aload_3
42: invokevirtual #7                  // Method com/rolex/microlabs/jvm/OverrideExample$Man.sayHello:()V
45: aload         4
47: invokevirtual #8                  // Method com/rolex/microlabs/jvm/OverrideExample$Woman.sayHello:()V
```
编译的指令是相同的，但是最终执行的结果却不一样。这和 invokevirtual 指令的执行过程有关。invokevirtual 指令会首先从操作数栈顶对象开始找，如果找到匹配的方法，则返回。如果没有找到，则继续按照继承关系从下往上依次查找，直到找到为止。这种根据运行时实际类型进行方法匹配叫做**动态分派**。
>由于查找是在运行时进行的，为了性能考虑，JVM 并不会按照这种方式进行查找，而是通过虚方法表的方式进行查找。

不论是静态分派还是动态分派，Java 在进行方法匹配时可检查的不外乎对象的类型和方法的参数，这叫做方法的**宗量**。通过宗量的数量，又可以分为单分派和多分派。在 Overload 中，编译器需要看方法参数的静态类型，还要看调用的是哪个对象的方法，所以 Overload 是**静态多分派**。在 Override 中，方法的参数是固定的，只需要看匹配的是哪个对象中的方法，所以 Override 是**动态单分派**。