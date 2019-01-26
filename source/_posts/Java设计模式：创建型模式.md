---
title: Java设计模式：创建型模式
date: 2019-01-21 00:58:21
categories: 
- Java
---
## 设计模式及其分类

### 设计模式
设计模式（Design pattern）是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

1994 年，Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四人合著出版了一本《Design Patterns - Elements of Reusable Object-Oriented Software（设计模式 - 可复用的面向对象软件元素）》 的书，该书首次提到了软件开发中设计模式的概念。

四位作者合称 GOF（四人帮，全拼 Gang of Four）。他们所提出的设计模式主要是基于以下的面向对象设计原则。

- 对接口编程而不是对实现编程。
- 优先使用对象组合而不是继承。

### 分类
<html>

<table class="tg">
  <tr>
    <th class="tg-0pky">模式</th>
    <th class="tg-0pky">描述</th>
    <th class="tg-0pky">包含</th>
  </tr>
  <tr>
    <td class="tg-0pky">创建型模式</td>
    <td class="tg-0pky">提供了创建对象的同时隐藏创建逻辑的方式，<br>而非使用 new 运算符直接实例化对象。<br>程序在判断针对某个给定实例需要创建哪些对象时更加灵活。</td>
    <td class="tg-0pky">工厂模式（Factory Pattern）<br>抽象工厂模式（Abstract Factory Pattern）<br>单例模式（Singleton Pattern）<br>建造者模式（Builder Pattern）<br>原型模式（Prototype Pattern）</td>
  </tr>
  <tr>
    <td class="tg-0pky">结构型模式</td>
    <td class="tg-0pky">这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。</td>
    <td class="tg-0pky">适配器模式（Adapter Pattern）<br>桥接模式（Bridge Pattern）<br>过滤器模式（Filter、Criteria Pattern）<br>组合模式（Composite Pattern）<br>装饰器模式（Decorator Pattern）<br>外观模式（Facade Pattern）<br>享元模式（Flyweight Pattern）<br>代理模式（Proxy Pattern）</td>
  </tr>
  <tr>
    <td class="tg-0lax">行为型模式</td>
    <td class="tg-0lax">这些设计模式特别关注对象之间的通信。</td>
    <td class="tg-0lax">责任链模式（Chain of Responsibility Pattern）<br>命令模式（Command Pattern）<br>解释器模式（Interpreter Pattern）<br>迭代器模式（Iterator Pattern）<br>中介者模式（Mediator Pattern）<br>备忘录模式（Memento Pattern）<br>观察者模式（Observer Pattern）<br>状态模式（State Pattern）<br>空对象模式（Null Object Pattern）<br>策略模式（Strategy Pattern）<br>模板模式（Template Pattern）<br>访问者模式（Visitor Pattern）</td>
  </tr>
</table>
</html>

可以使用一张图来展示设计模式之间的关系：

![设计模式之间的关系](Java设计模式：创建型模式/设计模式之间的关系.jpg)

### 六个原则：

- 开闭原则（Open Close Principle）
```
对扩展开放，对修改关闭。
```

- 里氏代换原则（Liskov Substitution Principle）
```
基类可以出现的任何地方，子类一定可以出现。
```

- 依赖倒转原则（Dependence Inversion Principle）
```
针对接口编程，依赖于抽象而不依赖于具体。
```

- 接口隔离原则（Interface Segregation Principle）
```
降低类之间的耦合度。
```

- 迪米特法则，又称最少知道原则（Demeter Principle）
```
实体应当尽量少地与其他实体发生相互作用，系统功能模块应相对独立。
```

- 合成复用原则（Composite Reuse Principle）
```
尽量使用合成/聚合的方式，而不是使用继承。
```

三大设计模式和六个原则构成了软件设计模式的基本内容，下面将介绍创建者模式。

## 创建者模式

### 工厂模式
创建对象时不暴露创建逻辑，通过使用共同的接口来指向新创建的对象。

工厂模式可分为简单工厂、工厂方法、抽象工厂。

#### 简单工厂

#### 工厂方法

#### 抽象工厂

### 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。

单例模式可分为线程安全懒汉式、线程不安全懒汉式、饿汉式、双重检查锁式以及静态内部类等多种实现方式。

#### 懒汉式

#### 饿汉式

#### 双检式

#### 静态内部类式

### 建造者模式
使用多个简单的对象逐步构建一个复杂的对象，其中存在一个独立于其他对象的Builder类。

### 原型模式
实现了一个用于创建当前对象的克隆的原型接口。当直接创建对象的代价比较大时（例如数据库对象操作），可以采用这种模式。
