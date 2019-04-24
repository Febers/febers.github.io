---
title: Dart 面向对象
date: 2019-04-23 09:10:53
tags:
- Dart
categories:
- Dart
---

## 引言

作为一门面向对象的语言，Dart 在很多方面跟 Java 都很相似。Dart 中所有对象都是类的实例，所有类都属于 Object 的子类，类的继承则使用 Mixin 机制。

<!--more-->

## 定义

使用 class 关键字定义一个类。与 Java 类似，如果没有显示地定义构造函数，会默认一个无参构造函数。使用 new 关键字和构造函数来创建对象。

```dart
class Point {
  num x;
  num y;
  num z;
}

void main() {
  var point = new Point();
  print(point.hasCode);//未定义父类的时候，默认继承自Object
}
```

### 构造函数

可以在构造函数的参数前加 this 关键字直接赋值

```
class Point {
    num x;
    num y;
    num z;
    
    //第一个值传递给this.x，第二个值传递给this.y
    Point(this.x, this.y, z) {	
            this.z = z;
    }
    
    //命名构造函数，格式为Class.name(var param)
    Point.fromeList(var list): 
            x = list[0], y = list[1], z = list[2]{	//使用冒号初始化变量
    }

    //当然，上面也可以简写为：
    //Point.fromeList(var list):this(list[0], list[1], list[2]);

     String toString() => 'x:$x  y:$y  z:$z';
}

//调用父类的构造方法
class ColorPoint extends Point {
  String color;

  ColorPoint.fromXYZAndColor(num x, num y, num z, String color)
      : super.fromXYZ(x, y, z) {
    this.color = color;
    print('ColorPoint');
  }
}

void main() {
    var p1 = new Point(1, 2, 3);
    var p2 = new Point.fromeList([1, 2, 3]);
    print(p1);	//默认调用toString()函数
}
```

需要创建不可变对象的话，可以在构造函数前使用`const`关键字定义编译时常量对象

```dart
class ImmutablePoint {
    final num x;
    final num y;
    const ImmutablePoint(this.x, this.y); // 常量构造函数
    static final ImmutablePoint origin = const ImmutablePoint(0, 0); // 创建一个常量对象不能用new，要用const
}
```

### 工厂构造函数

Dart 中一种获取单例对象的方式，使用工厂模式来定义构造函数。对于调用者来说，仍然使用 new 关键字来获取对象，具体的实现细节对外隐藏。

```dart
class Logger { 
	final String name; 
    bool mute = false; 
    
    // 变量前加下划线表示私有属性 
    static final Map<String, Logger> _cache = <String, Logger>{}; 
    
    factory Logger(String name) { 
        if (_cache.containsKey(name)) { 
            return _cache[name]; 
        } else { 
            final logger = new Logger._internal(name); 
            _cache[name] = logger; 
            return logger; 
        } 
    }

    Logger._internal(this.name); 
    
    void log(String msg) { 
        if (!mute) { 
            print('$name: $msg'); 
        } 
    } 
} 

var logger = new Logger('UI'); 
logger.log('Button clicked');
```



### Getter / Setter

用来读写对象的属性，每个属性都对应一个隐式的 Getter 和 Setter，通过`obj.x`调用。类似于 Kotlin，可以使用`get`、`set`关键字拓展相应的功能。如果属性为`final`或者`const`，则只有对外的 Getter。

```dart
class Rectangle {
    num left;
    num top;
    num width;
    num height;

    Rectangle(this.left, this.top, this.width, this.height);
    
    // right 和 bottom 两个属性的计算方法
    num get right => left + width;
    set right(num value) => left = value - width;
    
    num get bottom => top + height;
    set bottom(num value) => top = value - height;
}

void main() {
    var rect = new Rectangle(3, 4, 20, 15);
    assert(rect.left == 3);
    
    rect.right = 12;
    assert(rect.left == -8);
}
```

### 类方法

实例方法跟 Java 类似，抽象方法有所不同，不需要使用`abstract`显示定义，只需要在方法签名后用`;`来代替方法体即表示其为一抽象方法，Dart 中非抽象类也可以定义抽象方法。

```dart
abstract class Bird {
  void fly();
}

class Sparrow extends Bird {
  void fly() {

  }
    
  void sleep();
}
```



## 继承

在 Dart 中没有“接口”这一概念，类分为抽象类和非抽象类，唯一的区别是后者不可直接实例化。Dart 中仍然使用了`implements`和`extends`关键字，不过两者有所不同

- implements 代表实现，子类无法访问父类的参数，可以实现多个类
- extends 代表继承，可继承父类的非私有变量，使用单继承机制

在构造函数体前使用 `super`关键字调用父类构造函数，使用`@override`来注解复写父类的方法。

### Mixin 继承机制

#### 继承歧义

在了解该机制之前先认识“继承歧义”，也叫“菱形问题”。当两个类 B 和 C 继承自 A，D类继承自 B 和 C 时将产生歧义。当 A 中有一个方法在 B 和 C 中已经重写，而 D 没有 重写，那么 D 继承的方法的版本是 B 还是 C？

![继承歧义](Dart-面向对象\继承歧义.png)



不同的编程语言有不同的方法处理该问题

| 语言   |                           解决方案                           |
| ------ | :----------------------------------------------------------: |
| C++    | 需要显式地声明要使用的特性是从哪个父类调用的(如：`Worker::Human.Age`)。C++不支持显式的重复继承，因为无法限定要使用哪个父类 |
| Java 8 | Java 8 在接口上引入默认方法。如果`A、B、C`是接口，`B、C`可以为`A`的抽象方法提供不同的实现，从而导致`菱形问题`。`D`类必须重新实现该方法，否则发生编译错误。（Java 8 之前不支持多重继承，不受钻石问题风险的影响） |

Dart 和 Python、Scala 等语言类似，使用 Mixin 机制解决该方法，或者写作“mix-in（混入）”更容易理解。

### Mixin

