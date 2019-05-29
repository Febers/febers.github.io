---
title: Gradle 构建工具详解
date: 2019-05-25 22:37:41
tags:
- Gradle
- 构建工具
- Groovy
- Android  
- Java
categories:
- Gradle
---

## 引言

Gradle 是一个基于 JVM 的自动化构建工具，使用 Groovy  DSL 声明配置。现代软件开发包含众多步骤，包括编译、测试、打包等，如果需要手动重复每一过程，将会耗费大量时间、增加出错概率，项目自动化应运而生。在此之前，常见的 Java 构建工具包括 Ant、Gant 和 Maven 等，Gradle 结合了以上工具的优点，基于约定大于配置，通用灵活，是 Android 的官方构建工具。本文将介绍 Gradle 的基本知识、Groovy 基本语法以及 Android 开发中的 Gradle 知识。

![](https://plugins.gradle.org/shared-assets/shared/images/gradle-logo-horizontal.svg)

<!--more-->

## 入门

### 搭建环境

确保系统已安装 JDK 1.7及以上，此处将介绍 Gradle 在 Windows 平台下的手动安装。在 https://gradle.org/releases/ 下载最新的 release 包并解压至相应文件夹，然后在系统环境变量添加`GRADLE_HOME`，作者的变量值为`D:\gradle-5.4.1`，最后再将`%GRADLE_HOME%\bin`添加进`Path`变量中即可。在命令行中键入`gradle -v`验证环境搭建结果。

```bash
PS C:\Users\Febers> gradle -v

Welcome to Gradle 5.4.1!

Here are the highlights of this release:
 - Run builds with JDK12
 - New API for Incremental Tasks
 - Updates to native projects, including Swift 5 support

For more details see https://docs.gradle.org/5.4.1/release-notes.html
```

### Hello World

新建一个工程项目，比如`gradle_demo`，然后新建一个`build.gradle`文件，在其中输入

```groovy
task hello {
  doLast {
    println 'Hello World!'
  }
}
```

在当前文件夹命令行中键入`gradle -q hello`即可输出`Hello  World!`。这里使用的是基于 Groovy 的 DSL（Domain Specific Language，领域特定语言）。

task 和 action 是 Gradle 的重要元素，前者代表一个独立的原子操作，比如复制一个文件、编译一次 Java 代码，这里简单定义一个名为`hello`的 task；后者则是前者的组成部分，`doLast`代表 task 执行的最后一个 action，task 执行完毕之后会回调该 action。

### 日志级别

和 Android 类似，Gradle 也定义了日志级别

| 级别      | 用于     |
| --------- | -------- |
| ERROR     | 错误信息 |
| QUIET     | 重要信息 |
| WARNING   | 警告信息 |
| LIFECYCLE | 进度信息 |
| INFO      | 信息消息 |
| DEBUG     | 调试消息 |



上文运行任务所使用到的命令`gradle -q hello`中的`-q`即为日志的级别开关选项

| 开关选项      | 输出级别        |
| ------------- | --------------- |
| 无            | LIFECYCLE及以上 |
| -q 或 --quiet | QUIET及以上     |
| -i 或 --info  | INFO及以上      |
| -d 或 --debug | DEBUG及以上     |

### Project

每个 Gradle 项目都由一个或多个 Project 构成，每个 Project 又都由 Task 构成。一个 `build.gradle`文件便是对一个 Project 对象的配置。在 Android 项目中，根目录会存在一个`build.gradle`文件，每个模块下也会有一个该文件。

在构建脚本中调用的没有在构建脚本中定义的方法和属性都委派给 Project 对象，比如`project.copy()`等价于`copy()`、`project.buildDir`等价于`buildDir` 

### Task

#### 创建任务

```groovy
task hello {
  doLast {
    println 'Hello World!'
  }
}

//直接用任务名称
def Task helloo = task(helloo)
helloo.doLast {
  println 'Helloo World!'
}

//声明任务配置
def Task hellooo = task(hellooo, group: BasePlugin.BUILD_GROUP)
hellooo.doLast {
  println 'Hellooo World!'
}

//使用 TaskContainer 的 create 方法创建，以上三种方式最终都会调用该方法
tasks.create(name: 'helloooo') {
  doLast {
    println 'helloooo World!'
  }
}
```

输入`gradle -q hello*`之后的结果为，说明成功创建任务

```bash
Hello World!
Helloo World!
Hellooo World!
helloooo World!
```

#### 任务顺序

通过`dependsOn`指定任务的依赖，通过一个例子理解

```groovy
task hello {
  println 'hello'
    
  doFirst {
    println 'hello first'
  }
  doLast {
    println 'Hello last'
  }
}

task go(dependsOn: hello) {
  println 'go'

  doLast {
    println 'go last 0'
  } 

  doFirst {
    println 'go first'
  }

  doLast {
    println 'go last 1'
  }
}
```

输入`gradle -q go`之后的输出结果为

```bash
hello
go
hello first
Hello last
go first
go last 0
go last 1
```

#### 排除任务

在命令行后添加`-x go`来排除任务 go，输入`gradle hello -x go`之后

```bash
> Task :hello
hello
hello first
hello last

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

#### 动态任务

可以通过拓展方法`times`动态创建任务

```groovy
3.times {
  count -> task "task$count" {
    doLast {
      println "task $count"
    }
  }
}
```

输入`gradle -q task0`之后将输出`task 0`，已创建的三个任务中的一个


#### 任务属性

标准属性有`group`、`description`等，除此之外的自定义属性需添加`ext`前缀

```groovy
task hello {
  group = 'group0'
  description = 'description'
  ext.myTitle = 'title'
  ext.myId = 9527

  doLast {
    println "任务分组属性: $group"
    println "任务描述:属性 $description"
    println "自定义Title属性: $myTitle"
    println "自定义Id属性: $myId"
  }
}
```

输出正常

## Groovy 

![](Gradle-构建工具详解\groovy.png)

Gadle 使用 Groovy 的 DSL 编写，Groovy 是 Apache 推出的 JVM 语言。Groovy 的学习可以参考其[官方文档](http://cndoc.github.io/groovy-doc-cn/)，写得相当友好。Groovy 可以与 Java 无缝连接，甚至可以在其中直接使用 Java 语法，Java 中调用 Groovy 也相当方便。此处只介绍一些与 Java 不同的地方。

Groovy 会默认导入以下包，不需要显示导入

```bash
java.io.*
java.lang.*
java.math.BigDecimal
java.math.BigInteger
java.net.*
java.util.*
groovy.lang.*
groovy.util.*
```

可以导入 SDK 之后在 IDE 中编写 Groovy 程序，也可以在 `build.gradle` 中编写代码，在 task

中调用，使用 gradle 命令运行。以下的例子采用后一种方法。

### 变量与方法

使用 def 关键字来定义变量和方法，可以不指定变量的类型，默认访问修饰符为 public。Groovy 中使用双引号定义字符串的时候类型为`GString`而非`java.lang.String`，因此可以使用`$`输出表达式结果

```groovy
task t {
  //定义变量，可以使用 def，也可以使用具体类型，或者两者结合
  def a = 0
  def int b = 1
	
  //定义字符串，同 dart 
  String s = "s"	
  String ss = 'ss'
  String sss = """first row
  second row"""

  println "95-27=${minus(95,27)}"
  println "95+27=${add 95,27}"
}

//指定返回类型则 def 可省略，且参数类型可省略
//不使用 return 则返回最后一行
int minus(a, b) {
  println "before return"
  a - b
}

//定义方法
def add(int a, int b) {
  return a + b
}
```

控制台输出

```bash
before return
95-27=68
95+27=122
```

### 对象

Groovy 中的类与 Java 类似，不过由于没有访问修饰符，默认为`public`，要想实现 Java 默认的包访问权限（`default`），可以使用注解` @PackageScope`。对于没有可见性修饰的变量，Groovy 会隐式提供`setter/getter`方法。以下的两个类是等价的

```groovy
task t {
  def object = new ClassInGroovy()
  object.name = "Jack"
  println "${object.name}"
}

public class ClassInJava {
  public String name;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}

class ClassInGroovy {
  String name
}
```

此外 Groovy 使用`asType`函数进行类型转换，支持`object.with{ }`进行级联操作，支持使用`?.`进行非空操作，都是些很眼熟的语法特性

### 闭包

闭包在 Groovy 中是`groovy.lang.Closure`的实例，类似于 Dart 中的函数是 Function 的实例，因此可以像变量一样传递，其语法定义为

```groovy
//闭包的参数为可选项
def closure = { [closureParameters -> ] statements }
```

闭包跟函数不同的地方在于其可以访问外部变量

```groovy
task t {
  def str = "hello"
  def closure0 = {
    println str  
  }

  def closure1 = { String name, int t -> 
    println "before print"
    println "$name call $t times"
  }

  //调用，call 可省略
  closure0.call()	
  closure1("Jack", 10)
}
```

与 Java 中的 Lambda 表达式类似，闭包可以当做参数传递，当闭包为最后一个参数时可写在调用的括号后

### 文件读取

相较于 Java，Groovy 的文件读写非常简洁友好。在当前文件夹新建一个文件`静夜思.txt`

```groovy
task t {
  def path = "静夜思.txt"
  def file = new File(path).eachLine { line ->
    println line
  }
  //更简洁
  println file.text
	
  //写入，该方法会打印并覆盖原来的内容
  file.withPrintWriter {
    it.println "表达了诗人对家乡的思念"
  }
}
```

### 其他

Groovy 中的流程控制语句（比如`if/else`、`for/in`、`switch/case`）、数据结构（比如`List`、`Map`）与 Java/Dart 类似，在此便不过多费笔墨，参考官方文档即可。

## Gradle Wrapper

Gradle 包装器。为了应对团队开发中 Gradle 环境和版本的差异会对编译结果带来的不确定性，使用 Gradle Wrapper，它是一个脚本，可以指定构建版本、快速运行项目，从而达到标准化、提到开发效率。Android Studio 新建项目时自带 Gradle Wrapper，因此 Android 开发者很少单独下载安装 Gradle

Gradle Wrapper 的工作流程如下图

![](https://docs.gradle.org/current/userguide/img/wrapper-workflow.png)

使用 Gradle Wrapper 启动 Gradle 之后，如果指定版本的 Gradle 没有被下载关联，会先从官方仓库下载到用户本地，进行解包并执行批处理文件。后续的构建运行都会重用这个解包的运行时安装程序

### 构建 Gradle Wrapper

Gradle 内置 Wrapper Task，执行 Wrapper Task 就可以在项目目录中生成对应的目录文件。在项目根目录执行`gradle wrapper` 命令即可。之后根目录的文件结构如下

```bash
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
└── gradlew.bat
```

文件含义为

- gradle-wrapper.jar ：包含 Gradle 运行时的逻辑代码。
- gradle-wrapper.properties ：负责配置包装器运行时行为的属性文件
- gradlew：Linux 平台下，用于执行 Gralde 命令的包装器脚本。
- gradlew.bat：Windows 平台下，用于执行 Gradle 命令的包装器脚本。

查看`.properties`文件

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.4.1-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

属性含义为

- distributionBase：Gradle 解包后存储的主目录。
- distributionPath：distributionBase 指定目录的子目录。distributionBase+distributionPath 为 Gradle 解包后的存放位置。
- distributionUrl：Gradle 发行版压缩包的下载地址。
- zipStoreBase：Gradle 压缩包存储主目录。
- zipStorePath：zipStoreBase 指定目录的子目录。zipStoreBase+zipStorePath 为 Gradle 压缩包的存放位置。

如果官方包发行版下载缓慢，可以手动更改`distributionUrl`为可用地址


### 使用 Gradle Wrapper

使用`gradlew.bar`代替`gradle`运行 Gradle Project，首次使用会下载 Gradle 到配置文件指定的位置，作者的路径为`C:\Users\23033\.gradle\wrapper\dists\gradle-5.4.1-bin\e75iq110yv9r9wt1a6619x2xm\gradle-5.4.1`

```bash
PS D:\Work\Gradle\gradle_demo> .\gradlew.bat t
Downloading https://services.gradle.org/distributions/gradle-5.4.1-bin.zip
...................................................................................

Hello World!
```

再次使用该命令便不会重复下载。升级 Gradle 版本可以通过`gradlew wrapper –gradle-version 5.*.*`命令实现

## Gradle 插件

Gradle 中的插件可分为两类

- 脚本插件：额外的构建脚本，类似于一个 build.gradle
- 对象插件：又叫二进制插件，是实现了 Plugin 接口的类

应用插件又分为两个步骤，一是解析插件，二是通过`apply()`把插件应用到项目中。


### 脚本插件

在根目录新建一个`other.gradle`文件，内容如下

```groovy
ext {
  otherVersion = '1.0'
  otherUrl = 'https://febers.github.io'
}
```

将`build.gradle`内容修改为

```groovy
apply from: 'other.gradle'
task t {
  println "版本为: ${otherVersion},地址为: ${otherUrl}"
}
```

输出结果

```bash
PS D:\Work\Gradle\gradle_demo> .\gradlew.bat t

> Configure project :
版本为: 1.0,地址为: https://febers.github.io
```


### 对象插件

对象插件是实现了`org.gradle.api.plugin<Project>`接口的插件，又可分为内部插件和第三方插件


#### 内部插件

使用以下方法应用 Java 插件（因为默认导入`org.gradle.api.plugin`包，所以可以去掉包名）

```groovy
apply plugin: org.gradle.api.plugins.JavaPlugin
apply plugin: JavaPlugin	//去掉包名
apply plugin: 'java'	//使用 pluginid（实现 plugin 接口的插件的属性）
apply plugin: 'cpp'    //Gradle 中含有大量插件
```

#### 第三方插件

一般为 jar 文件，通过`buildscript`配置，此处引入 Android Gradle 插件

```groovy
buildscript {
  repositories {
    google()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:3.4.1'
  }
}
apply plugin: 'com.android.application'
```

如果第三方插件被托管到 [Gradle - Plugins](Gradle - Plugins
https://plugins.gradle.org/)，也可以不使用`buildscript`

```groovy
plugins {
  id "com.quittle.setup-android-sdk" version "1.2.0"
}
```

#### 自定义对象插件

Plugin 接口中定义了一个`apply`方法，重写该方法，在其中通过传进来的参数`Object o`（实际为 Project 类型）调用 task 新建一个任务

```groovy
class MyPlugin implements Plugin {
  @Override
  void apply(Object o) {
    o.task("myTask") {
      println "This is a custom task in custom plugin"
    }
  }
}
apply plugin: MyPlugin
```

输入命令`gradle myTask`即可验证

## Android 中的 Gradle

为了支持 Android 项目的构建，Google 为 Gradle 编写了 Android 插件，组成 Android 构建系统。Android Studio+Gradle 是目前最流行的 Android 开发构建环境。关于 Android 构建配置可查阅官方文档 [配置构建](https://developer.android.com/studio/build?hl=zh-cn)



### 模块类型

Android Studio 中的每个项目包含一个或多个含有源代码文件和资源文件的模块，这些模块可以独立构建测试，模块类型包含以下几种

-  Android 应用程序模块：可能依赖于库模块，构建系统会将其生成一个 apk 文件
- Android 库模块：包含可重用的特定于 Android 的代码和资源，构建系统将其生成一个 aar 文件
- App 引擎模块：包含应用程序引擎继承的代码和资源
- Java 库模块：包含可重用的代码，构建系统将其生成一个 jar 文件



### 项目结构

在 Android Studio 中，Android 项目视图如下，以开发者个人项目 [UESTC_BBS](https://github.com/Febers/UESTC_BBS) 为例

![](Gradle-构建工具详解\项目结构.png)

所有构建稳健位于 Gradle Scripts 层级下，文件作用如下

- 项目 build.gradle：配置项目整体属性，比如指定的代码仓库、依赖的 Gradle 版本等
- 模块 build.gradle：配置当前模块的编译参数
- gradle-wrapper-properties：配置 Gradle Wrapper
- gradle-properties：配置 Gradle
- setting.gradle：配置 Gradle 的多项目管理
- local.properties：存放 Android 项目的私有属性配置，如 SDK 路径
- multiDexKeep.pro、proguard-rules.pro：可选的混淆文件，用于配置放置在主 Dex 的类、声明避免混淆的类



### 项目 build.gradle

典型的项目`build.gradle`文件如下

```groovy
buildscript {
    ext.kotlin_version = '1.3.10'
    repositories {
        mavenCentral()
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url 'https://jitpack.io' }
        maven { url "https://maven.google.com" }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

其中`google()`是配置 Google 的 Maven 仓库，`maven { url "https://maven.google.com" }`同理


### 模块 build.gradle

典型的模块`build.gradle`文件如下

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'
ext.anko_version = '0.10.5'

dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'com.android.support:multidex:1.0.3'
    implementation 'androidx.appcompat:appcompat:1.0.2'
    testImplementation 'junit:junit:4.12'
}

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.febers.uestc_bbs"
        minSdkVersion 17
        targetSdkVersion 28
        versionCode 12
        versionName "1.1.4"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        multiDexEnabled true
        multiDexKeepProguard file('multiDexKeep.pro') // keep specific classes using proguard syntax
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

第一行 apply 的是一个`application`，说明当前模块为一个应用程序模块，Gradle 的 Android 插件分为以下几种

- 应用程序插件：插件 id 为`com.android.application`，构建生成 apk
- 库插件：插件 id 为`com.android.library`，构建生成 aar
- 测试插件：插件 id 为`com.android.test`，用于测试其他模块
- feature 插件：插件 id 为`com.android.feature`，用于创建 Android Instant App
- instant App 插件：插件 id 为`com.android.instantapp`，是 Android Instant App 的入口

很多属性都可以望文生义，此处不再赘述。



---

参考

[Gradle的Android插件入门](http://liuwangshu.cn/application/android-gradle/1-gradle-plug-in.html)

[Gradle核心思想](http://liuwangshu.cn/tags/Gradle%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3/)

