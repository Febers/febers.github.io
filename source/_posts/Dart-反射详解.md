---
title: Dart 反射初识
date: 2019-05-20 10:16:44
tags:
- Dart
- 反射
categories:
- Dart
---

## 引言

在 [Java 反射详解](https://febers.github.io/Java-反射详解/) 一文中，我们知道`反射是一种计算机处理方式，是程序可以访问、检测和修改它本身状态或行为的一种能力`。换一个角度说，反射可以细分为自省——程序在运行时决定自身结构的能力，以及自我修正——程序在运行时改变自身的能力。Dart 的反射基于 mirror 概念，它指的是反映其他对象的对象，并且目前只支持自省，不支持自我修改。

<!--more-->

## Mirror

### Example

```dart
main() {
    ClassMirror cm = reflectClass(ChildClass);
    cm.instanceMembers.forEach((key, value) => print('$key >>> $value'));
    
    ClassMirror simpleCM = reflectClass(Simple);
    Simple simple = simpleCM.newInstance(Symbol.empty, ['hey']) as Simple;
}
```
```dart
class Simple {
  Simple(a) {
    print('A new Simple: $a');
  }
}
```

```dart

class SuperClass {
  int superField = 0;
  final int superFinalField = 1;
  int get superGetter => 2;
  set superSetter(x){ superField = x; }
  int superMethod(x) => 4;

  static int superStaticField = 5;
  static final int superStaticFinalField = 6;
  static const superStaticConstField = 7;
  static int get superStaticGetter => 8;
  static set superStaticSetter(x) { }
  static int superStaticMethod(x) => 10;
}

class ChildClass extends SuperClass {
  int aField = 11;
  final int aFinalField = 12;
  get aGetter => 13;
  set aSetter(x) { aField = x; }
  int aMethod(x) => 15;

  static int staticField = 16;
  static final staticFinalField = 17;
  static const staticConstField = 18;
  static int get staticGetter => 19;
  static set staticSetter(x) { staticField = x; }
  static int staticMethod(x) => 21;
}
```



控制台输出为

```bash
Symbol("==") >>> MethodMirror on '=='
Symbol("hashCode") >>> MethodMirror on 'hashCode'
Symbol("toString") >>> MethodMirror on 'toString'
Symbol("noSuchMethod") >>> MethodMirror on 'noSuchMethod'
Symbol("runtimeType") >>> MethodMirror on 'runtimeType'
Symbol("superField") >>> Instance of '_SyntheticAccessor'
Symbol("superField=") >>> Instance of '_SyntheticAccessor'
Symbol("superFinalField") >>> Instance of '_SyntheticAccessor'
Symbol("superGetter") >>> MethodMirror on 'superGetter'
Symbol("superSetter=") >>> MethodMirror on 'superSetter='
Symbol("superMethod") >>> MethodMirror on 'superMethod'
Symbol("aField") >>> Instance of '_SyntheticAccessor'
Symbol("aField=") >>> Instance of '_SyntheticAccessor'
Symbol("aFinalField") >>> Instance of '_SyntheticAccessor'
Symbol("aGetter") >>> MethodMirror on 'aGetter'
Symbol("aSetter=") >>> MethodMirror on 'aSetter='
Symbol("aMethod") >>> MethodMirror on 'aMethod'
A new Simple: hey
```

### 分类

在官方 API 页面可以看到所有的 Mirror 类型：[dart:mirrors library](https://api.dartlang.org/stable/2.3.0/dart-mirrors/dart-mirrors-library.html)。Mirror 的主要类型如下

- ClassMirror：Dart 类的反射类型

- InstanceMirror：Dart 实例的反射类型

- ClosureMirror： 闭包的反射类型

- DeclarationMirror：类属性的反射类型

- IsolateMirror：Isolate 的反射类型

- MethodMirror：Dart 方法（包括函数、构造函数、getter/setter 函数）的反射类型

  

通过`dart:mirrors`包内顶层函数`reflecClass` 获得类的“镜像”的实例，该实例的`instanceMembers`属性如下

```dart
  Map<Symbol, MethodMirror> get instanceMembers;
```



由控制台输出结果可以看到，对于普通字段（属性），除自身外还列出了以“=”结尾的 setter 字段，对于不提供 setter 的`final`字段则只出现一次。

使用`staticMembers`将列出所有的静态字段

```dart
cm.staticMembers.forEach((key, value) => print('$key >>> $value'));
```

输出如下

```bash
Symbol("staticField") >>> Instance of '_SyntheticAccessor'
Symbol("staticField=") >>> Instance of '_SyntheticAccessor'
Symbol("staticFinalField") >>> Instance of '_SyntheticAccessor'
Symbol("staticConstField") >>> Instance of '_SyntheticAccessor'
Symbol("staticGetter") >>> MethodMirror on 'staticGetter'
Symbol("staticSetter=") >>> MethodMirror on 'staticSetter='
Symbol("staticMethod") >>> MethodMirror on 'staticMethod'
```

可以发现父类静态成员没有出现在列表中，这是因为静态属性不会被继承、不能被`ChildClass`调用。

### Symbol

`Symbol`表示使用 Dart 的 mirror API 反射得到的实例类型，位于`dart:core`包

```dart
part of dart.core;

/// Opaque name used by mirrors, invocations and [Function.apply].
abstract class Symbol {

  static const Symbol unaryMinus = const Symbol("unary-");

  static const Symbol empty = const Symbol("");

  //工厂构造方法
  //也可以直接通过 Symbol s = #name; 创建
  const factory Symbol(String name) = internal.Symbol;

  int get hashCode;
  
  bool operator ==(other);
}
```

### 源码

通过 ClassMirror 的源码，可以大概看出 Dart 语言关于反射的设计思想以及对外提供的 API

```dart
abstract class ClassMirror implements TypeMirror, ObjectMirror {

  ClassMirror get superclass;	//父类 ， Object的父类为null

  List<ClassMirror> get superinterfaces;	//接口列表

  bool get isAbstract;

  bool get isEnum;

  Map<Symbol, DeclarationMirror> get declarations;	//不包含父类属性和方法

  Map<Symbol, MethodMirror> get instanceMembers;	//实例属性

  Map<Symbol, MethodMirror> get staticMembers;	//静态属性

  //如果S = A with B ,那么ClassMirror（S）.mixin 为 ClassMirror（B），否则返回本身
  ClassMirror get mixin;
    
  /**
   * 调用构造方法
   * @param constructorName 构造方法名称（默认构造方法为空字符串，命名构造方法为其命名）
   * @param positionalArguments 参数列表
   */
  InstanceMirror newInstance(Symbol constructorName, List positionalArguments,
      [Map<Symbol, dynamic> namedArguments]);

  bool operator ==(other);

  bool isSubclassOf(ClassMirror other);
}
```



## 影响

在 Java 中，当开发者多次（10w 次以上）访问、修改某一属性时，使用反射的成本会比正常访问高很多，同时会让`private`修饰符失去作用。在 Dart 中，反射的影响主要在于，编译器使用`tree shaking`的过程确定应用真正运行时使用的代码，以减少程序的大小。但是使用反射将使`tree shaking`失效，因为任何代码都有可能被使用，由此严重影响应用的启动时间和内存占用。

解决👆一问题的有效方法是，通过代码生成执行反射。为了“告知”编译器使用反射的代码和方式，开发者可以使用`dart:reflectable`库，通过特定元数据注解反射代码。[reflectable](https://github.com/dart-lang/reflectable)

另一个影响在于最小化，其表示对下载到 Web 浏览器的源程序进行压缩的过程。在最小化过程中，源代码使用的名称在编译代码中被压缩成了短名称。这一过程会对反射带来不良影响，因为最小化之后，原来表示声明的名称的字符串，不再对应程序中的实际名称。

为了解决这一问题， Dart 反射使用 symbol 而非字符串作为 key，symbol 会被执行最小化的程序`minifier`识别并使用与标识符同样的压缩方式。这也是上面的输出中出现`Symbol(...)`的原因。开发者也可以通过 MirrorSystem 提供的`static String getName(Symbol symbol)`方法获得非最小化名称字符串。 

## 小结

目前来看 Dart 还不算一门“足够完善”的语言，比如反射机制的不完全、文档教程匮乏等等，相信随着 Flutter 的发展，这门语言的发展会更加地好。关于 Dart 反射的知识全部来自 Gilad Bracha 所著《Dart 编程语言》，不知道是不是翻译的问题，写得不够明晰，看得也是一头雾水。希望有朝一日，更加掌握 Dart 的反射机制，再写一篇《Dart 反射机制详解》的文章 😄。



![](https://images-na.ssl-images-amazon.com/images/I/51r64LJDGuL._SX369_BO1,204,203,200_.jpg)