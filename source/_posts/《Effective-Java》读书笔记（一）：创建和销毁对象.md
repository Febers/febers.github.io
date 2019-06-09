---
title: 《Effective Java》读书笔记（一）：创建和销毁对象
date: 2019-06-04 22:21:51
tags:
- Java
- Effective
categories:
- Java
---

Spring 系列还在进行，又开了新坑（苦笑）。这一系列基于《Effective Java》——相比《Spring in Action》，标题就友好很多，大部分是 Java 的一些编程思想，探讨如何写出简洁、高效和健壮的代码，写起来会很轻松（~~毕竟就是抄嘛~~）。这本书买了有一年多，反复翻阅了几次。记忆最深的是，有一次从成都到武汉，因为赶高铁不及，只能转坐绿皮火车。在车上的二十多个小时，大部分时间花在这本书上。算是一本质量很高的指导手册。<!--more-->

## 用静态工厂方法代替构造器

比如下面的代码，来自 Boolean 的简单示例

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE
}
```

通过静态工厂方法（区别于设计模式中的工厂方法），代替传统的公有构造器，向客户端提供实例，这么做的优势在于

- 静态工厂方法具有名称：这是显而易见的。另一方面，拥有多个参数的类很可能提供多个构造器，如何区分这些构造器就是问题。

- 静态工厂方法可以提供单例：从这个角度看，相当于实现了简单的单例模式。

- 静态工厂方法可以返回任何子类型的对象：典型的应用为`Java.util.Collections`类，其中定义了很多静态内部类比如`EmptyList`，并通过静态方法`emptyList`返回实现

- 静态工厂方法可以提供更简洁的实例化代码：实际上在较新的 JDK 版本已经去掉了多余的类型参数，但是静态工程方法确实可以做到更简洁

  ```java
           List< String> strings = new ArrayList<>();
           List< String> emptyList = Collections.emptyList();
  ```

  

静态工厂方法的缺点在于，第一如果类不包含公有构造器，则外部无法继承它——勉强算一个缺点吧——第二它与其他静态方法没有任何区别，只是返回的是自身的一个实例。这就造成如何类没有提供公有构造方法，那么外部调用者将苦恼于它的实例化。下面是一些静态工厂方法的惯用名称：

- `valueOf`：实际上属于类型转换方法
- `of`：上面名称的简洁形式
- `getInstance`：返回的实例通过方法参数描述。在单例模式中保证返回唯一的实例
- `newInstance`：跟上面的相似，但保证返回的实例与所有其他实例不同
- `getType`：不了解，`Character`提供了该工厂静态方法返回`CharacterData`的不同实现
- `newType`：不了解

## 多个构造器时考虑使用构建器

实际上就是使用简单的建造者模式实例化对象，还是通过代码说明

```java
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

这种方式广泛运用于 Java 开发中，优点显而易见，唯一的缺点可能为了创建对象需要先创建其建造器（不值一提）

## 使用私有构造器、枚举强化单例

其实就是单例模式，简单贴一下代码吧，忘记了可以回去看设计模式的笔记：[Java设计模式：创建型模式](https://febers.github.io/Java%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%9B%E5%BB%BA%E5%9E%8B%E6%A8%A1%E5%BC%8F/)

```java
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

下面是枚举类型实现单例

```java
public enum  Singleton {
    INSTANCE
}
```

## 通过私有构造器强化不可实例化的能力

用得最多的地方就是各种`Util`类，一般只有一个静态方法，实现一个简单的功能。最典型的例子就是`java.lang.Math`

~~划水而过~~

## 避免创建不必要的对象

最常见的例子就是 Java 的基本数据类型，直接使用`int`、`char`而不是它们的包装类型`Integer`、`Character`，因为基本数据类型直接保存在内存模型中的栈中，而且具有复用性，比对象类型更加节省内存

Java 的经典面试题中经常牵扯到`String`的实例化

```java
        String s0 = "9527";
        String s1 = "9527";
        System.out.println(s0 == s1);

        String s2 = new String("9527");
        System.out.println(s0 == s2);
```

（第一个比较是多余的，因为对于基本类型来说，`==`会直接比较他们的值）。**一定范围内**的基本数据类型会保存在栈内的常量池中。第一行代码，虚拟机首先创建一个引用`s0`，然后到常量池中寻找是否存在字面量为`9527`的值，如果有则直接返回栈地址，没有则新建一个常量`9527`并将`s0`指向它，然后返回。到了`s1`被创建的时候，根据上面的步骤，`s1`将直接指向`s0`所指向的栈地址

第二个比较结果显然是`false`。如果使用构造器构造一个`String`类型的变量，则虚拟机会先在栈中创建引用`s2`，在堆中开辟内存新建`String`对象，保存传入的参数`9527`，然后将堆地址指向栈内的`s2`

对于`String`来说，直接使用构造器更糟糕的一点还在于，其被声明为`final`，每`new`一次就创建一个对象。可以想象如果在循环中使用了构造器来创建字符串，将会造成多么大的内存浪费

## 消除过期的对象引用

通过一个栈的实现说明

```java
public class Stack {
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object o) {
        ensureCapacity();
        elements[size++] = o;
    }
    
    public Object pop(){
        if (size == 0) 
            throw new EmptyStackException();
        return elements[--size];
    }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

该实现的问题出现在，`pop`方法中弹出的对象将不会被系统回收，因为栈内部仍然维护着它们的过期引用，解决办法如下

```java
    public Object pop(){
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```

## 避免使用终结方法

终结方法其实就是`Object`内的`finalize`方法，其声明为

```java
protected void finalize() throws Throwable { }

```

该方法是当 JVM 发现不可达对象时，将其回收之前调用的方法。问题在于，从发现该回收对象到真正回收这之间的时间是不确定的。JVM 不但不保证终结方法会被及时执行，甚至不保证其会被执行，如果重写终结方法，并期望在其中添加资源回收业务，将造成巨大隐患——资源可能永远都不会被回收。

使用终结方法的另一个弊端在于，如果子类重写了父类的`finalize`，但是却没有在其中调用父类的`finalize`，那么父类（非  Object）永远不会被回收。

------

好了，这就是第一章的~~划水~~内容。可以看到大都是一些有用的编程经验和技巧，用来时刻提醒自己不要犯低级的错误。可能这本书提及到的内容有些开发者一辈子都碰不到，但是要想成为一个卓越的工程师，注重细节和原理永远是不可获取的。这也算是对自己的勉励吧。

------

不知道是不是前面写 Dart、Gradle、Spring 的东西太多，突然这么轻松地写博客，竟然感到“周身血气运转通畅、心情愉悦”，简直好惨一博主。那下一篇也继续划水吧~