---
title: Java设计模式：创建型模式
date: 2019-01-21 00:58:21
tags:
- 设计模式
- Java
categories: 
- Java
---
## 设计模式及其分类

### 设计模式
设计模式（Design pattern）是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

1994 年，Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四人合著出版了一本《Design Patterns - Elements of Reusable Object-Oriented Software（设计模式 - 可复用的面向对象软件元素）》 的书，该书首次提到了软件开发中设计模式的概念。<!--more-->

四位作者合称 GOF（四人帮，全拼 Gang of Four）。他们所提出的设计模式主要是基于以下的面向对象设计原则。

- 对接口编程而不是对实现编程。
- 优先使用对象组合而不是继承。

### 分类
<html>
<table class="tg"><tr><th class="tg-0pky">模式</th><th class="tg-0pky">描述</th><th class="tg-0pky">包含</th></tr><tr><td class="tg-0pky">创建型模式</td><td class="tg-0pky"><td class="tg-0pky">工厂模式（Factory Pattern）<br>抽象工厂模式（Abstract Factory Pattern）<br>单例模式（Singleton Pattern）<br>建造者模式（Builder Pattern）<br>原型模式（Prototype Pattern）</td></tr><tr><td class="tg-0pky">结构型模式</td><td class="tg-0pky">这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。</td><td class="tg-0pky">适配器模式（Adapter Pattern）<br>桥接模式（Bridge Pattern）<br>过滤器模式（Filter、Criteria Pattern）<br>组合模式（Composite Pattern）<br>装饰器模式（Decorator Pattern）<br>外观模式（Facade Pattern）<br>享元模式（Flyweight Pattern）<br>代理模式（Proxy Pattern）</td></tr><tr><td class="tg-0lax">行为型模式</td><td class="tg-0lax">这些设计模式特别关注对象之间的通信。</td><td class="tg-0lax">责任链模式（Chain of Responsibility Pattern）<br>命令模式（Command Pattern）<br>解释器模式（Interpreter Pattern）<br>迭代器模式（Iterator Pattern）<br>中介者模式（Mediator Pattern）<br>备忘录模式（Memento Pattern）<br>观察者模式（Observer Pattern）<br>状态模式（State Pattern）<br>空对象模式（Null Object Pattern）<br>策略模式（Strategy Pattern）<br>模板模式（Template Pattern）<br>访问者模式（Visitor Pattern）</td></tr>
</table>
</html>

可以使用一张图来展示设计模式之间的关系：

![设计模式之间的关系](Java设计模式：创建型模式/设计模式之间的关系.jpg)

### 六个原则：

- 开闭原则（Open Close Principle）
> 对扩展开放，对修改关闭。


- 里氏代换原则（Liskov Substitution Principle）
> 基类可以出现的任何地方，子类一定可以出现。

- 依赖倒转原则（Dependence Inversion Principle）
> 针对接口编程，依赖于抽象而不依赖于具体。

- 接口隔离原则（Interface Segregation Principle）
> 降低类之间的耦合度。

- 迪米特法则，又称最少知道原则（Demeter Principle）
> 实体应当尽量少地与其他实体发生相互作用，系统功能模块应相对独立。

- 合成复用原则（Composite Reuse Principle）
> 尽量使用合成/聚合的方式，而不是使用继承。

三大设计模式和六个原则构成了软件设计模式的基本内容，下面将介绍创建者模式。

## 创建者模式

### 工厂模式
创建对象时不暴露创建逻辑，通过使用共同的接口来指向新创建的对象。

工厂模式可分为简单工厂、工厂方法、抽象工厂。

#### 简单工厂
工厂类（SimpleFactory）拥有一个工厂方法（create），接受了一个参数，通过不同的参数实例化不同的产品类。

![简单工厂](Java设计模式：创建型模式/简单工厂.jpg)

简单工厂简单粗暴，但是其缺点也很明显，一是当产品种类繁多时代码量提高，二是增加新产品时，需要修改工厂实现，违背开闭原则（对拓展开放，对修改关闭）。工厂方法正好可以解决简单工厂的这两个缺点。

#### 工厂方法
工厂方法针对每一种产品提供一个工厂类，通过不同的工厂实例，创建不同的产品实例。

![工厂方法](Java设计模式：创建型模式/工厂方法.jpg)

从图中可以看到，在工厂方法中，增加产品种类并不需要修改工厂类，只需要添加相应的工厂即可，符合开放-封闭原则。其缺点在于，对于某些可以形成产品族的情况，处理起来比较复杂，这一问题使用抽象工厂解决。

#### 抽象工厂
工厂方法模式是一种极端情况的抽象工厂模式（即只生产一种产品的抽象工厂模式），而抽象工厂模式可以看成是工厂方法模式的一种推广，其面对产品族。

![抽象工厂](Java设计模式：创建型模式/抽象工厂.jpg)


以上介绍的三种工厂方法各有优缺点
>简单工厂： 用来生产同一等级结构中的任意产品。（不支持拓展增加产品） <br>工厂方法：用来生产同一等级结构中的固定产品。（支持拓展增加产品）<br>  抽象工厂：用来生产不同产品族的全部产品。（不支持拓展增加产品；支持增加产品族）<br>  

### 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。其关键在于构造方法是私有的，面向所有对象提供全局单例。单例模式的缺陷在于没有抽象层，无法进行拓展，同时其类结构复杂，违背了单一职责原则。

单例模式可分为线程安全懒汉式、线程不安全懒汉式、饿汉式、双重检查锁式以及静态内部类等多种实现方式。

#### 懒汉式
支持延迟初始化，代码如下，由于其不支持多线程，严格意义上来说并不算单例模式。

```Java
public class Singleton {  

    private static Singleton instance;  
    private Singleton (){}  
  
    public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}  
```

要支持多线程，可以给`getInstance`方法加锁`synchronized`，但是效率会变得很低。

#### 饿汉式
不会延迟初始化，多线程安全，由于没有加锁，所以效率更高。缺点是类加载时就初始化，占用内存，同时也很容易产生垃圾对象。

```Java
public class Singleton {  

    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    
    public static Singleton getInstance() {  
        return instance;  
    }  
}  
```

#### 双检式
双检即Double Checked Locking，安全、支持延迟初始化且在多线程情况下能保持高性能。`getInstance` 的性能对应用程序很关键。

```Java
public class Singleton {  

    private volatile static Singleton singleton;  
    private Singleton (){}  
    
    public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
                if (singleton == null) {  
                    singleton = new Singleton();  
                }  
            }  
        }  
        return singleton;  
    }  
} 
```

该方法使用了关键字`volatile`，关于该关键字的分析：[Java并发编程：volatile关键字解析
](https://www.cnblogs.com/dolphin0520/p/3920373.html)

> 在不使用 volatile 时，假设两个线程A、B第一次调用单例方法，如果线程A先执行 instance = new Instance()，由于构造方法是一个非原子操作，编译后会生成多条字节码指令，因为 Java 的指令重排序，可能会先执行 instance 的赋值操作——在内存中开辟一片存储对象的区域，然后直接返回内存的引用。此时虽然 instance 不为空，但实际的初始化操作却还未执行，如果线程B进入，就会看到一个不为空的但是不完整（没有完成初始化）的 instance 对象。
volatile 关键字保证不同线程对变量进行操作时的可见性，禁止指令重排序优化，从而安全的实现单例。


#### 静态内部类式
能达到双检锁方式一样的功效，但实现更简单。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

与双检式方式一样利用`ClassLoder`机制来保证初始化`instance`时只有一个线程。
关于 ClassLoader：[一看你就懂，超详细java中的ClassLoader详解](https://blog.csdn.net/briblue/article/details/54973413)

```Java
public class Singleton {  

    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    
    private Singleton (){}  
    
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE;  
    }  
} 
```

相较于双重检查锁，由于静态内部类的特性——需要使用才会装载到内存中，所以实际上在第一次调用`getInstance`之前，SingletonHolder 是没有被装载进来的，只有在第一次调用了 getInstance() 之后，内部静态类的实例才会真正装载。

#### 枚举
实现单例模式的最佳方法。简洁、支持序列化、绝对防止多次实例化。

```Java
public enum Singleton {  

    INSTANCE;  
    public void whateverMethod() {}  
    
}  
```

### 建造者模式
使用多个简单的对象逐步构建一个复杂的对象。下面的例子比较简单直观，实际上在复杂的系统中可以包含单独的 Director 、Builder 甚至抽象层等元素。

```Java
public class Human {

    private final String name;
    private final int height;
    private final int weight;

    public static class Builder {
        // 必要参数
        private final int name;

        // 可选参数
        private int height = 170;
        private int weight = 60;

        public Builder(String name) {
            this.name = name
        }

        public Builder height(int height) {
            this.height = height;
            return this;
        }

        public Builder weight(int weight) {
            this.weight = weight;
            return this;
        }

        public Human build() {
            return new Human(this);
        }
    }

    private Human(Builder builder) {
        name = builder.name;
        height = builder.height;
        weight = builder.weight;
    }

    public static void main(String[] args) {
        Human human = new Human.Builder("Jack")
                    .height(175)
                    .weight(60)
                    .build();
    }
}
```

### 原型模式
当直接创建对象的代价比较大时（例如数据库对象操作），可以采用这种模式克隆出多个一模一样的对象。

Java的`clone`方法便是使用了这种方法，关于该方法：[java对象克隆以及深拷贝和浅拷贝](https://www.cnblogs.com/xuanxufeng/p/6558330.html)

```Java
public inteface Prototype {
    Prototype clone();
}

public class ConcretePrototype implement Prototype {
    
    public override Prototype clone() {
        Prototype prototype = new ConcretePrototype();
        return prototype;
    }
    
    public static void main(String[] args) {
        Prototype p1 = new ConcretePrototype();
        Prototype p2 = p1.clone();
    }
}
```