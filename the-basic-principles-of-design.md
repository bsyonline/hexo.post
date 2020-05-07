---
title: The Basic Principles of Design
tags:
  - Interview
category:
  - Design Pattern
author: bsyonline
lede: 没有摘要
date: 2020-04-05 00:11:57
thumbnail:
---


#### **1. 单一职责原则 (Single Resposibility Principle, SRP)**
通常我们设计一个类，比如 UserInfo ，会像这样。
<img src="https://s1.ax1x.com/2020/04/05/G0vs61.png" alt="20200405002557" border="0" style="width:250px;">
这样做非常简单，相信大多数人也都是这样做的，但是这并不符合单一职责原则。**单一职责原则是说有且只有一种原因会引起变更。**在 UserInfo 中，```void setUserId(Integer userId);``` ，```void setUserName(String userName);``` ，```void setPassword(String password);``` 是用户的属性，```void changePassword(String password);``` 是用户的行为，所以它不符合单一职责原则。我们对它进行修改以符合单一职责原则。
<img src="https://s1.ax1x.com/2020/04/05/G0vrlR.png" alt="20200405002919" border="0" style="width:350px;">
现在用户属性在 IUserBO 中，用户的行为在 IUserBiz 中，这样就符合单一职责原则了。
单一职责原则
* 优点：每个类的复杂度降低，定义清晰明确，易于维护和扩展。
* 缺点：类之间的耦合增高，数量增多，增加了设计的复杂度。

在实际中，单一原则有时很难使用，主要是因为职责是不可度量也不是一成不变的。但是我们还是应该尽可能应用它，不管是接口，类，还是方法。设计尽量要让使用的人清楚类或方法是干什么的，而不要让人猜他是怎么用的。

#### **2. 里氏替换原则 (Liskov Substitution Principle, LSP)**
通俗的讲，**所有使用父类的地方都可以换成子类，且不会出现错误，反之则不行，这就是里氏替换原则**。
<img src="https://s1.ax1x.com/2020/04/05/GrYNQS.png" alt="GrYNQS.png" border="0" style="width:350px;"/>
```
public class Client {
    public static void main(String[] args) {
        Person alice = new Person();
        Dog dog = new Dog();
        alice.set(dog);
        alice.play();
    }
}
```
```void set(Animal animal)``` 可以换成子类，满足里氏替换原则。
里氏替换原则的优点：增强程序的健壮性和可扩展性。
#### **3. 依赖倒置原则 (Dependence Inversion Principle, DIP)**
**依赖倒置原则的意思是上层不依赖下层，抽象不依赖细节，而细节应该依赖抽象，通俗的说就是面向接口编程。**
<img src="https://s1.ax1x.com/2020/04/05/GrdhPs.png" alt="GrdhPs.png" border="0" style="width:350px;" />
Benz 是细节，ICar 是抽象，如果把 Benz 换成 BMW 不应该影响 ICar 。
```
public class Client {
    public static void main(String[] args) {
        IDriver tom = new Driver();
        tom.drive(new Benz());
    }
}
```
Client 调用 Benz ，所以 Client 是上层，Benz 是下层，将 Benz 换成 BMW ，Client 其他细节不需要改变。
依赖倒置原则的优点：减少因需求变化导致的变更，程序易于扩展和维护。

#### **4. 接口隔离原则 (Interface Segregation Principle, ISP)**
接口隔离原则是说接口应该细化，接口中的方法也应尽可能少。接口隔离原则和单一职责有点类似，也有些冲突，需要灵活应用。接口隔离原则目的是提高内聚，增加灵活性。

#### **5. 迪米特法则 (Law of Demeter, LoD)**
**迪米特法则也叫最少知识原则，简单来说就是一个类对需要调用的类知道的越少越好。**比如我们设计一个软件安装引导程序，我们可以这样设计。
<img src="https://s1.ax1x.com/2020/04/06/GyKGxH.png" alt="20200406170228" border="0" style="width:200px;">
这样设计，Wizard 暴露了太多的方法给 SoftwareInstaller ，耦合性太高，按照迪米特法则进行优化。
<img src="https://s1.ax1x.com/2020/04/06/GyKYMd.png" alt="20200406170346" border="0" style="width:200px;">
修改完后，Wizard 只暴露了一个方法给 SoftwareInstaller ，Wizard 中的 private 方法的修改不会影响到 SoftwareInstaller 。

#### **6. 开闭原则 (Open Closed Principle, OCP)**
**开闭原则是说对扩展开放，对修改关闭。简单来说，我们的程序应该通过扩展功能来实现新的功能，而不应该通过修改已有的代码来实现新的功能。**假设我们已经开发好了商品模块，现在需要进行促销，我们应该怎么做呢？有 3 种方法。
1. 在接口种增加打折方法。不建议，接口不应该经常发生变化，如果接口变了，所有的实现都需要改变。
2. 将实现类中的价格改成折扣价格。容易造成业务上的混乱，分不清是折扣还是原价。
3. 通过新增折扣商品的方式实现功能。
<img src="https://s1.ax1x.com/2020/04/06/GymBiF.png" alt="GymBiF.png" border="0" style="width:400px;"/>

开闭原则的优点：
1. 易于进行单元测试。
2. 提高了复用性。
3. 提高了可靠性。