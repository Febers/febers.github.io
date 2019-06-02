---
title: Java 反射详解
date: 2019-03-16 23:25:47
tags:
- Java
- 反射
categories:
- Java
---

反射是一种计算机处理方式，是程序可以访问、检测和修改它本身状态或行为的一种能力。Java 反射使得我们可以在程序运行时动态加载一个类，动态获取类的属性、方法、构造函数等信息。反射使 Java 这一静态语言有了动态的特性。<!--more-->

## 使用
### Example
```Java
public class Apple {

    private int price;

    public Apple(){}

    public Apple(int price) {
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public static void main(String[] args) throws ClassNotFoundException, 
            NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        //正常调用
        Apple apple = new Apple();
        apple.setPrice(10);
        System.out.println("Price is " + apple.getPrice());

        //反射调用
        Class clazz = Class.forName("Apple");

        //使用 getFields() 无法获取私有属性
        Field[] fields = clazz.getDeclaredFields();
        for (Field field: fields) {
            System.out.println("Field is " + field.getName());
        }
        Method setPriceMethod = clazz.getMethod("setPrice", int.class);

        Constructor appleConstructor = clazz.getConstructor();
        Object appleObj = appleConstructor.newInstance();
        //Constructor appleConstructor = clazz.getConstructor(int.class);   //获得有参构造器
        //Object appleObj = appleConstructor.newInstance(int.class)     //调用有参构造器

        setPriceMethod.invoke(appleObj, 12);

        Method getPriceMethod = clazz.getMethod("getPrice");
        System.out.println("Price is " + getPriceMethod.invoke(appleObj));
    }
}
```
### API
#### 类的实例化和构造函数

> 获取公有构造函数，不包括父类，Class.class
> public Constructor<?>[] getConstructors() 
> public Constructor<T> getConstructor(Class<?>... parameterTypes)
>
> 获取当前类构造函数，忽略修饰符
> public Constructor<?>[] getDeclaredConstructors()
> public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)

> 构造函数调用，Constructor.class
> public T newInstance(Object... initargs)
>
> 忽略修饰符，强制调用
> public void setAccessible(boolean flag)

#### 类成员变量的获取

> 获取公有变量，包括父类，Class.class
> public Field[] getFields()
> public Field getField(String name)
>
> 获取当前类成员变量，忽略修饰符
> public Field[] getDeclaredFields()
> public Field getDeclaredField(String name)

> 成员变量赋值，Field.class
> //obj为实例对象
> public void set(Object obj,Object value)
>
> 忽略修饰符，强制调用
> public void setAccessible(boolean flag)

#### 类方法的获取

> 获取公有方法，包括父类，Class.class
> public Method[] getMethods()
> public Method getMethod(String name, Class<?>... parameterTypes)
>
> 获取当前类方法，忽略修饰符
> public Method[] getDeclaredMethods()
> public Method getDeclaredMethod(String name, Class<?>... parameterTypes)

> 方法调用，Method.class
> //obj为类实例化对象，如果为静态方法obj为Null
> invoke(Object obj, Object... args)
>
> 忽略修饰符，强制调用
> public void setAccessible(boolean flag)

#### 类注解的获取

> 获取类的"annotationClass"类型的注解，包括父类
> public Annotation<A>    getAnnotation(Class annotationClass)
>
> // 获取类的全部注解 ，包括父类
> public Annotation[]    getAnnotations()
>
> // 获取类自身声明的全部注解 ，忽略修饰符
> public Annotation[]    getDeclaredAnnotations()

#### 类父类的获取

> 获取实现的全部接口
> public Type[]    getGenericInterfaces()
>
> 获取父类
> public Type    getGenericSuperclass()

## 原理

### RTTI和Class对象

RTTI，即 Run-Time Type Identification，运行时类型识别。RTTI 能在运行时就能够自动识别每个编译时已知的类型。

很多时候需要进行向上转型，比如 Fruit 类派生出 Apple 类，当现有的方法将 Fruit 作为参数时，如果我们传入其派生类的引用，那么 RTTI 就会起作用。通过 RTTI 识别出 Apple 类是 Fruit 的派生类，然后向上转型。特别是在用接口类型作为参数的时候，这一特性更是被频繁使用。

而这些类型信息是通过一个特殊对象**Class（java.lang.Class）**实现的，它包含跟类相关的信息。

当程序要使用某个类时，如果该类还未被加载到内存中，则系统会通过加载、连接、初始化三步来实现对这个类进行初始化。

- 加载 ：将 class 文件读入内存，并为之创建一个 Class 对象。任何类被使用时系统都会建立一个 Class 对象。
- 连接：
  - 验证 是否有正确的内部结构，并和其他类协调一致
  - 准备 负责为类的静态成员分配内存，并设置默认初始化值
  - 解析 将类的二进制数据中的符号引用替换为直接引用

- 初始化：如果该类有超类，则对其初始化，执行静态域和静态初始化块。

获取 Class 对象的方式有三种

1. Object 类的 getClass() 方法，执行静态块和动态构造块

2. 数据类型的静态属性 class ，不会初始化该类

3. Class类中的静态方法`public static Class forName(String className)`，执行静态块，不执行动态构造块

### RTTI和反射

Java 有两种 RTTI 方式，一种是传统的，假设在编译时已经知道了所有的类型；还有一种，是利用反射机制，在运行时再尝试确定类型信息。

RTTI和反射之间的真正区别只在于：

> RTTI：编译器在编译时打开和检查.class文件
> 反射：运行时打开和检查.class文件

严格的说，反射也是一种形式的RTTI，不过，一般的文档资料中把RTTI和反射分开，因为一般的，大家认为 RTTI指的是传统的 RTTI，通过继承和多态来实现，在运行时通过调用超类的方法来实现具体的功能

未完待续