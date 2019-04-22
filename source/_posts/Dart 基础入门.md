---
title: Dart 基础入门
date: 2019-04-22 14:27:18
categories:
- Dart
tags: 
- Dart

---

## 引言

终于开始 Flutter 的具体学习，一切从 Dart 语言开始。

```
void main() {
    print('hello world');
}
```

<!--more-->

## 环境搭建

Dart 的 SDK 下载可能需要梯子。作者使用的开发环境是 Intellij IDEA，首先下载 Dart 的 Plugin，重启 IDEA 之后创建一个 Dart Project，当然首先需要确认 SDK 的路径。

![](Dart 基础入门\idea_create.png)



创建之后的文件窗口如下，右键 DartDemo 文件夹，新建一个 dart 文件，和 C/C++ 类似，Dart 语言以文件中的`main`函数作为运行的入口。在`run`之前需要`edit configuration`，很简单，只要指定对应的文件即可。

![](Dart 基础入门\idea_category.png)



## 声明变量

Dart 是一门完全面向对象的语言，包括基本数据类型、函数都是对象，继承自 Object。声明一个对象可以初始化，否则其值为`null`。可以使用具体的类型声明，也可以使用`var`、`dynamic`、`const`、`final`等关键字

```
void main() {
  var i = 0;
  var d = 2.0;
  var s = 'hello';
  var b = true;
  var l = [1, 2, 3];
  var m = {0: 'a', 1: 'b'};
  
  print(main is Function);  //true
}
```

通过查看官方库的`core`包，可以大概看出其结构

![dart_core](Dart 基础入门\dart_core.png)

## 数据类型

### Numbers

包括 int 和 double，分别代表整形和浮点型。

int 的数值范围不超过2的64位，具体与平台有关，一般为 -2^53 to 2^53。double 则属于64位的双精度浮点型数据。

### String

在 Dart 中，可以使用单引号或者双引号定义一个字符串变量，或者使用三引号定义格式字符串。Dart 的字符串使用 UTF-16 编码

```dart
var s1 = "hello";
var s2 = 'world';
var s3 = 'hello' + 'world';
var s4 = 'hello' 'world';
var s5 = """hello 
              world""";
```

如果要使用 UTF-32 编码，则要通过 Runes（符号文字），它可以把文字转换成符号表情或者特定文字。

```dart
var clapping = '\u{1f44f}';
print(clapping);

Runes runes = new Runes('\u{1f44d}');
print(new String.fromCharCode(runes.first));
```



上面的输出为

```
👏
👍
```

### Boolean

提供 bool 用来声明布尔型变量，默认为 false。

### List 和 Map

```dart
List<int> l = [1,2,3];
l.forEach((x) => print(x)); //forEach的参数为 Function
for(var x in l) { //使用for-in
  print(x);
}

Map<int, String> map = {0: "a", 1: "b"};
map[0] = "c";
```

## Function

### 普通Function

函数或者方法。和Java 不同，Dart 中方法是有类型的，属于 Function。

```dart
void main() {
  single1('Tom', 20);
  single2('Tom', 20);
  single3('Tom', 20, weight: 30); //由于没有位置约束，必须指定形参名称
  single4('Tom', 20, 20, 30);
}

bool single1(String name, int age) {
  return true;
}

//返回类型和参数类型可省略，支持返回表达式
single2(name, age) => true;

//可选命名参数，调用时没有顺序要求，同时可选参数可以指定默认值
bool single3(String name, int age, {int weight = 60, int height}) => true;

//可选位置参数，通过位置来确定参数值，要想指定 height 必须先指定 weight
bool single4(String name, int age, [int weight, int height]) => true;
```

对于`main`方法来说，可以定义其为一个有参的方法，同样可以作为入口方法。在 Flutter 项目中的入口方法为：

```dart
void main() => runApp(MyApp());
```

### Lambda表达式

在 Lambda 表达式中，函数可以“没有名字”，同样，也可以像 kotlin 一样，定义一个函数变量

```dart
var f = (c) {
  print(c);
};

f('hello');

var list = [1, 2, 3];
printElement(x) {	//方法签名写成 void printElement(int x) 更直观
  print(x);
}

list.forEach(printElement);

```

`forEach`的函数定义如下

```dart
void forEach(void f(E element)) {
  for (E element in this) f(element);
}

```

下面的例子直观展示将函数作为变量传递的思想

```dart
Function makeAdder(num n) {
  return (num i) => n + i;
}
  
//更明晰的写法
Function makeAdder_(num n) {
  Function add = (num i) {
    return i + n;
  };
  return add;
}

var adder2 = makeAdder(2);
print(adder2(3));

```

控制台将输出 5

## 运算符

### 赋值操作符

除了`=`，还有`??=`，表示如果左边的变量为 null，则将右边的值赋予它，否则左边值不变。

### 相等

`==`将比较两个对象的属性是否相等，判断是否为同一对象使用的是预定义的`identical`方法

```dart
external bool identical(Object a, Object b);

```

### 除法

| 操作符 | 含义               |
| :----: | ------------------ |
|   /    | 除，比如 5/2 = 2.5 |
|   ~/   | 整除， 5/2 = 2     |

### 类型判断

| 操作符 | 含义                       |
| :----: | -------------------------- |
|   is   | 对象属于指定类型则返回true |
|  is!   | 对象不属于指定类型返回true |
|   as   | 类型转换                   |

### 条件表达式

分为两种

```dart
condition ? expr1 : expr2	//通用表达式
expr1 ?? expr2	//如果 expr1 非空，返回其值，否则返回 expr2
```

### 级联调用与非空调用

```dart
//常规写法
var button = querySelector('#button');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));

//使用级联表达式
querySelector('#button') // Get an object.
  ..text = 'Confirm'   // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));

//非空调用
print(button?.text);
```

## 其他

Dart 语言在比如 `if/else`、`[do、]while`、`for`、`switch/case`等语句上跟 Java 类似，不再赘述。

异常处理的做法如下

```dart
//抛出异常
throw 'x should be less than 10';	
throw new FormatException('Expected at least 1 section');

//捕获异常
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}

try {
 //...
} on Exception catch (e) {
  print('Unknown exception: $e');
} catch (e, s) {	
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```