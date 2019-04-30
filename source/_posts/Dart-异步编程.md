---
title: Dart 异步编程
date: 2019-04-30 08:21:26
tags:
- Dart
- 异步
categories:
- Dart
---

## 引言

Dart 属于单线程编程语言，在进行 I/O 操作或者其他耗时操作的时候，程序会进入阻塞状态。异步是 Dart 并发方案的基础。

<!--more-->

## 事件循环

作为一个事件驱动语言，Dart 同样拥有事件循环（Event Loop，类似于 Android 中的Looper/Handler）。Dart 有两个队列，一个是微任务队列（MicroTask Queue），一个是事件队列（Event Queue）

- 微任务队列包含 Dart 内部的微任务，主要通过`scheduleMicrotask`调度
- 事件队列包含外部事件，如 I/O、Timer、绘制事件等

![](Dart-异步编程\事件循环.png.jpg)



从上图可以看出，Dart 处理事件循环的逻辑

- 首先处理所有微任务队列里的微任务
- 处理完所有微任务之后，处理事件队列里的一个事件
- 回到微任务队列继续循环

对于微任务队列，一次性全部处理，对于事件队列，一次只处理一个。

## 微任务和事件

### 微任务

`dart:async`定义了一个顶级函数`scheduleMicrotask`，使用其让代码以微任务的方式异步执行

```dart
import 'dart:async';	//下文不再显示导入

void main() {
  print('开始');
  scheduleMicrotask((){
    print('这是一个微任务');
  });
  print("结束");
}
```

控制台输出

> 开始
> 结束
> 这是一个微任务

### 事件

使用`Timer.run(callback)`让代码以事件的方式异步执行

```dart
void main() {
  print('开始');

  Timer.run((){
    print('这是一个事件');
  });

  scheduleMicrotask((){print('这是微任务0');});
  scheduleMicrotask((){print('这是微任务1');});
  scheduleMicrotask((){print('这是微任务2');});
  scheduleMicrotask((){print('这是微任务3');});
  scheduleMicrotask((){print('这是微任务4');});
  
  print("结束");
}
```

控制台输出

> 开始
> 结束
> 这是微任务0
> 这是微任务1
> 这是微任务2
> 这是微任务3
> 这是微任务4
> 这是一个事件

同时可以看出和 Java 使用`new Thread（Runnable r）`不同，在 Dart 中，微任务的执行顺序是有序的。

考虑下面的代码，会输出`这是一个事件`吗？

```dart
  Timer.run((){
    print('这是一个事件');
  });
  
  foo() {
    scheduleMicrotask(foo);
  }

  foo();
```

根据上面 Dart 处理事件循环的逻辑图，`Timer.run`永远不会被执行，因为`scheduleMicrotask`永远在执行。

仅仅使用回调函数实现异步很容易陷入“回调地狱（Callback hell）”，为此 Dart 引入了`Future`

## Future

Future 封装了一系列静态函数完成异步操作，其内部通过`scheduleMicrotask`和`Timer`实现。此外还有一个`then`方法，接收一个名为`onValue`的闭包作为参数，该闭包在 Future 成功完成时被调用

| 函数                                                    | 用途              |
| ------------------------------------------------------- | ----------------- |
| Future(FutureOr<T> computation())                       | 创建事件任务      |
| microtask(FutureOr<T> computation())                    | 创建microtask任务 |
| sync(FutureOr<T> computation())                         | 创建同步任务      |
| delayed(Duration duration, [FutureOr<T> computation()]) | 创建延迟任务      |

通过代码理解

```dart
void main() {
  print('开始');

  Timer.run(() => print('这是一个事件'));

  scheduleMicrotask((){print('这是微任务0');});
  scheduleMicrotask((){print('这是微任务1');});
  scheduleMicrotask((){print('这是微任务2');});

  print("结束");

  Future(() => print('普通Future，通过Timer实现'));

  Future.delayed(const Duration(seconds: 2), () => print('延迟Future，通过Timer实现'));

  Future.microtask(() => print('Future创建微任务，通过scheduleMicrotask实现'));

  Future.sync(() => print('同步Future，执行同步代码'))
      .then((a) => print('then中的代码0'))
      .then((b) => print('then中的代码1'))
      .then((c) { throw '抛出then中的错误'; })
      .catchError((error) => print('捕获Error $error'))
      .whenComplete((){print('then任务完成');});
}
```

输出结果

> 开始
> 结束
> 同步Future，执行同步代码
> 这是微任务0
> 这是微任务1
> 这是微任务2
> Future创建微任务，通过scheduleMicrotask实现
> then中的代码0
> then中的代码1
> 捕获Error 抛出then中的错误
> then任务完成
> 这是一个事件
> 普通Future，通过Timer实现
>
> //延迟2s
>
> 延迟Future，通过Timer实现

---

在`dart:async`中，除了 Future，还有 Completer，用来将具体的 Future 流程控制权交给开发者

```dart
  var completer = Completer();
  var future = completer.future;
  future.then((d) => '返回的字符串')
      .then((e) => print('获得Completer中的future $e'));
  completer.complete((e) => print('设为完成状态'));
```

控制台输出

> 获得Completer中的future 返回的字符串



虽然 Future 缓解了回调地狱的问题，但如果串太多的`then`代码，可读性仍然会非常差，特别是各种 Future 嵌套的时候。与 JavaScript 类似，Dart 引入了`async/await`。

## async 和 await

async 关键字修饰的函数与传统函数并无区别，只是将返回值类型使用 Future 进行了封装。

通过代码具体理解

```dart
void main() async {
  getInt().then((i) => print('getInt: $i'));
  getString().then((s) => print('getString: $s'));
  print('main');
}

getInt() async => 2333;
getString() async => 'hello';
```

控制台输出

> main
> getInt: 2333
> getString: hello

可以看到，调用`async`方法的代码转换成了异步任务。要想使之变成同步顺序，使用`await`关键字。不过需要注意的是，该关键字必须要在`async`函数中使用

```dart
void main() async {
  var i = await getInt();
  print('getInt: $i');
  
  var s = await getString();
  print('getString: $s');
  
  print('main');
}
```

控制台输出

> getInt: 2333
> getString: hello
> main

继续下面的例子

```dart
void main() {
  print('main 0');
  foo();
  print('main 1');
}

foo() async {
  print('Foo');
  var s = await bar();
  print('from bar: $s');
}

bar() {
  print('Bar');
  return 'hello';
}
```

控制台输出为

> main 0
> Foo
> Bar
> main 1
> from bar: hello

也就是说，在`foo`中，除了第一行代码以及`bar()`这一函数调用之外的其他代码均为异步执行。当使用`await`的时候，其右边会马上返回一个 Future 对象，下面的代码则会以`then`的形式运行。

上面的代码转换成 Future 风格

```dart
foo() {
  print('Foo');
  return Future.sync(bar).then((s) => print('from bar: $s'));
}
```



## Generator

### stream 

stream 是 Dart 中一个长度不确定的值列表，可以是有限的或者无限的，重要的是我们不知道 stream 何时结束或已经结束。随时间改变的鼠标位置、所有素数的列表或者网络上的视频流，都可以看做一个 stream。

可以通过为 stream 注册一个或多个回调函数的方式，对其进行订阅监听。

### yield

yield 语句被用于生成器函数内，目的是给生成的集合添加新的结果。yield 语句总是使它的表达式被求值，通常情况下，求值结果会被追加到外层生成器所关联的集合中。如果生成器是同步的，则关联的集合是一个 iterable；如果是异步的，则关联的集合是一个 stream。

此外，yield 也会因外层的生成器是否同步产生不同的行为：同步时 yield 会暂停外层生成器，直至调用`moveNext`且返回值为 true，异步时生成器的执行会继续。

### 异步

一个函数体标记有`async*`修饰符的函数，将作为 stream 的生成函数。下面的函数生成一个包含自然数序列的 stream

```dart
get naturals async* {
  int k = 0;
  while (k < 3) {
    yield await k++;
  }
}

void main() {
  await for (var i in naturals) {
    print('get a natural $i');
  }
}
```

运行`main`函数，控制台将输出

> get a natural 0
> get a natural 1
> get a natural 2

当 naturals 被调用时，立即返回一个新的 stream，一旦 stream 被监听，函数体将运行，以便生成值来填充 stream。每一次迭代执行一次 yield 语句，k 将自增（由于 await 的存在，函数会有短暂停止），然后函数将继续执行并使用新的 k 值，该值将被 yield 追加到 stream 中。

### 同步

上述函数的同步形式

```dart
Iterable naturalsTo(n) sync* {
  int k = 0;
  while(k < n) {
    yield k++;
  }
}
```

---

通过一个混合编程的例子来体会两者的区别

```dart
Iterable nSync(n) sync* {
  int k = 0;
  while (k < n) {
    print('sync before k++ and k is $k');
    yield k++;
    print('sync after k++ and k is $k');
  }
}

Stream nAsync(n) async* {
  int k = 0;
  while (k < n) {
    print('async before k++ and k is $k');
    yield await k++;
    print('async after k++ and k is $k');
  }
}

void main() {
  nAsync(2).last;
  nSync(2).last;
  print('main');
}
```

控制台输出为

> sync before k++ and k is 0
> sync after k++ and k is 1
> sync before k++ and k is 1
> sync after k++ and k is 2
> main
> async before k++ and k is 0
> async after k++ and k is 1
> async before k++ and k is 1
> async after k++ and k is 2