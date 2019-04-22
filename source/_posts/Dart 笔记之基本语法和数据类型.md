---
title: Dart笔记（一）：基本语法和数据类型
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

![](Dart笔记(一)：基本语法和数据类型/idea_create.png)



创建之后的文件窗口如下，右键 DartDemo 文件夹，新建一个 dart 文件，和 C/C++ 类似，Dart 语言以文件中的`main`函数作为运行的入口。在`run`之前需要`edit configuration`，很简单，只要指定对应的文件即可。

![](Dart笔记(一)：基本语法和数据类型/idea_category.png)



## 上手

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

![dart_core](Dart笔记(一)：基本语法和数据类型/dart_core.png)

## 数据类型

### Numbers

包括 int 和 double，分别代表整形和浮点型。

int 的数值范围不超过2的64位，具体与平台有关。

double 的