---
title: Spring 学习笔记（三）：面向切面
date: 2019-06-02 12:52:24
tags:
- Spring 
- AOP
categories:
- Spring
---

[第一篇笔记](https://febers.github.io/Spring-学习笔记（一）：基本理念和-Bean-装配/) 曾简单提及了 AOP 的知识，本文将重点展开 Spring 对切面的支持，包括如何使用和 AspectJ 的具体应用<!--more-->

## 面向切面编程

Aspect-Oriented Programming，AOP。每个应用除了其核心业务之外，还会需要一些模块化的通用功能——比如日志、安全、通知等。重用通用功能的方式一般有继承（inheritance）、 委托（delegation），缺点在于前者会导致对象体系复杂、难以维护，后者则可能需要对委托对象进行复杂的调用。切面提供了不一样的思路，并且在很多场景下更加清晰简洁。

### AOP 术语

![](Spring-学习笔记（三）：面向切面\AOP_1.jpg)

#### Advice

通知，如果翻译为“增强”就能更直观地理解它所扮演的角色。Spring 切面可以应用 5 种类型的通知

- 前置通知（ Before ）：在目标方法被调用之前调用通知功能
- 后置通知（ After ）：在目标方法完成之后调用通知，不关心方法的返回值
- 返回通知（ After-returning ）：在目标方法成功执行之后调用通知
- 异常通知（ After-throwing ）：在目标方法抛出异常之后调用通知
- 环绕通知（ Around ）：通知包裹了被通知的方法，在被通知的方法调用之前喝调用之后执行自定义的行为

#### Joint point

连接点，应用可能有数以千计的时机应用通知，这些时机被称为连接点。根据上面的图理解，连接点是在应用执行过程中能够插入切面的一个点，改点可以是调用方法时、抛出异常时、甚至修改一个字段时。

#### Pointcut

切点，通知定义了切面的“什么”和“何时”，连接点定义了“时间点”，切点则定义了“何处”。切点会匹配通知所要织入的一个或多个连接点。通常使用明确的类和方法名称、或者使用正则表达式定义所匹配的类和方法来指定切点，有些 AOP 框架也会允许动态创建切点。

#### Aspect

切面，通知+切点，这两者共同定义了切面的全部内容

#### Introduction

引入，允许开发者向现有的类添加方法或属性

#### Weaving

织入，把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中，在目标对象的生命周期里有多个点可以进行织入：

- 编译期：切面在目标类编译时被织入。需要特殊的编译器，如 AspectJ 的织入编译器
- 类加载期：切面在目标类加载到 JVM 时被织入。这种方式需要特殊的类加载器，从而在目标类被引用之前增强其字节码。比如 AspectJ 5 的加载时织入（load-time weaving，LTW）
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，织入时 AOP 容器会为目标对象动态地创建一个代理对象。比如 Spring AOP

### Spring AOP

Spring 提供了 4 中类型的 AOP 支持：

- 基于代理的经典 Spring AOP
- 纯 POJO 切面
- `@AspectJ`注解驱动的切面
- 注入式 AspectJ 切面

相比起其他方式，第一种显得笨重而古典，不再介绍。纯 POJO 切面需要借助 Spring 的 `aop`命名空间，虽然足够简便，但也不再赘述。Spring 借鉴了 AspectJ 的切面以提供注解驱动的 AOP，其本质仍然是代理，但是编程模型几乎与成熟的 AspectJ 注解完全一致。如果开发者对 AOP 的需求超过了简单的方法调用（如构造器或属性拦截），那么可以考虑第四种方式——使用 AspectJ。

通过再代理类中包裹切面，Spring 在运行期把切面织入到 Spring 管理的 bean 中。代理类封装了目标类并拦截被通知方法的调用，再把调用转发给真正的目标 bean

![](Spring-学习笔记（三）：面向切面\AOP_2.jpg)

由于 Spring  AOP 基于动态代理，所以只支持方法连接点，不支持字段连接点——无法创建细粒度的通知、不支持构造器连接点——无法在 bean 创建时应用通知。

## 通过切点选择连接点

使用 AspectJ 的切点表达式语言来定义切点。Spring 仅支持 AspectJ 切点指示器的一个子集，因为 Spring 基于代理，而某些切点表达式与代理无关。下表是 Spring AOP 所支持的指示器

| AspectJ 指示器 | 描述                                                     |
| -------------- | -------------------------------------------------------- |
| arg()          | 限制连接点匹配参数为指定类型的执行方法                   |
| @args()        | 限制连接点匹配参数由指定注解标注的执行方法               |
| execution()    | 用于匹配是连接点的执行方法                               |
| this()         | 限制连接点匹配 AOP 代理的 bean 引用为指定类型的类        |
| target()       | 限制连接点匹配目标对象为指定类型的类                     |
| @target()      | 限制连接点匹配特定的具有指定类型注解的执行对象           |
| within()       | 限制连接点匹配指定的类型                                 |
| @within()      | 限制连接点匹配指定注解所标注的类型（方法定义在该类型中） |
| @annotation    | 限定匹配带有指定注解的连接点                             |

尝试使用其他指示器时，将抛出`IllegalArgumentException`异常

以上的指示器只有`execution()`指示器是实际执行匹配的，其他指示器都是用来限制匹配的。

### 编写切点

准备一个主题 Performance，可代表任何类型的现场表演，假设要编写其中`perform`方法触发的通知

```kotlin
package concert

interface Performance {
    fun perform()
}
```

使用 AspectJ 表达式编写切点

```java
execution(* concert.Performance.perform(..)) && within(concert.*)
```

使用`execution()`指示器选择`perform`方法，方法表达式以`*`开始表示不关心返回值类型，然后使用全限定类名和方法名；使用两个点号`..`表示不关心方法参数列表，切点会选择为所有的`perform`方法。`&&`操作符把`execution()`和`within`指示器连接在一起形成`与（and）`关系，限制需要配置的切点仅匹配`concert`包，类似也可以使用`||`和`!`来标识`或（or）`和`非（not）`操作

当然也可以通过 Spring 引入的指示器`bean()`指示特定的 bean，参数为 bean 的 id

```java
execution(* concert.Performance.perform(..)) && bean('woodstock')
execution(* concert.Performance.perform(..)) && !bean('woodstock')  
```

## 使用注解创建切面

### 简单通知

对于一场演出，我们将“观众”定义为切面

```kotlin
@Aspect
class Audience {

    @Before("execution(** concert.Performance.perform(..)")
    fun silenceCellPhones() {
        println("Silencing cell phones")
    }

    @Before("execution(** concert.Performance.perform(..)")
    fun takeSeats() {
        println("Taking seats")
    }

    @AfterReturning("execution(** concert.Performance.perform(..)")
    fun applause() {
        println("CLAP CLAP CLAP!!")
    }

    @AfterThrowing("execution(** concert.Performance.perform(..)")
    fun demandRefund() {
        println("Demanding a refund")
    }
}
```

> AspectJ 库需要通过依赖引入

Audience 定义了四个方法，通过 AOP 注解，形成以下期望的行为——演出之前观众就坐、将手机静音，演出很精彩则鼓掌欢呼，演出没有达到预期则要求退款。以上的方式有一点不足，每个方法的切点表达式都是一样的，重复了四次。为此可以使用`@Pointcut`注解定义可重用的切点

```kotlin
@Aspect
class Audience {

    @Pointcut("execution(** concert.Performance.perform(..))")
    fun performance(){ }
    
    @Before("performance()")
    fun silenceCellPhones() {
        println("Silencing cell phones")
    }

    @Before("performance()")
    fun takeSeats() {
        println("Taking seats")
    }

    @AfterReturning("performance()")
    fun applause() {
        println("CLAP CLAP CLAP!!")
    }

    @AfterThrowing("performance()")
    fun demandRefund() {
        println("Demanding a refund")
    }
}
```

除了注解和作为标识的空`performance`方法，Audience 仍然是一个 POJO，可以添加`@Component`注解将其注入容器中。但此时 Audience 不会视为切面，需要在配置类的的类级别通过使用`@EnableAspectJAutoProxy`注解启动自动代理功能

```kotlin
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
open class ConcertConfig
```

同时为 Performance 提供一个实现类

```kotlin
@Component
open class MyPerformance: Performance {
    
    override fun perform() {
        println("Start perform")
    }

}
```

测试类如下

```kotlin
@RunWith(SpringJUnit4ClassRunner::class)
@ContextConfiguration(classes = [ConcertConfig::class])
class PerformTest {
    
    @Autowired
    lateinit var performance: Performance

    @Test
    fun perform() {
        performance.perform()
    }
}
```

输出结果为

```bash
Silencing cell phones
Taking seats
Start perform
CLAP CLAP CLAP!!
```

### 环绕通知

环绕通知时最强大的通知类型，能够使编写的逻辑将被通知的目标方法完全包装，就像 在一个通知方法中同时编写前置通知和后置通知，重写上面的 Audience 切面

```kotlin
@Aspect
@Component
class Audience {

    @Pointcut("execution(** concert.Performance.perform(..))")
    fun performance(){ }

    @Around("performance()")
    fun watchPerformance(jointPoint: ProceedingJoinPoint) {
        try {
            println("Silencing cell phones")
            println("Taking seats")
            jointPoint.proceed()
            println("CLAP CLAP CLAP!!")
        } catch (e: Exception) {
            println("Demanding a refund")
        }
    }
}
```

上面的代码将实现同样的功能。环绕通知注解的方法`watchPerformance`中类型为`ProceedingJoinPoint`的参数是必须的，由此拿到切点的引用

### 带参数的通知

通过一个例子来展现带参数的通知如何实现，首先修改 MyPerformance 类

```kotlin
@Component
open class MyPerformance: Performance {

    private var performList: List<String> = arrayListOf("Song", "Dance", "Magic")

    override fun perform() {
        perform(Math.round(10f) % 2)
    }

    override fun perform(type: Int) {
        println("Start perform: ${performList[type]}")
    }
}
```

假设现在有一个计数类 PerformanceCounter，其将跟踪每一次表演节目，记录下该节目的类型以及次数并打印

```kotlin
@Aspect
@Component
class PerformanceCounter {

    private var playCounter: MutableMap<Int, Int> = HashMap()

    @Pointcut("execution(** concert.Performance.perform(..)) && args(whichType)")
    fun performanceTypePlay(whichType: Int) { }

    @After("performanceTypePlay(typeIntValue)")
    fun countPerformance(typeIntValue: Int) {
        val currentCount = getPlayCount(typeIntValue)
        playCounter[typeIntValue] = currentCount+1
        println("=== type: ${getDesByInt(typeIntValue)}, played ${currentCount+1} times ===")
    }

    private fun getPlayCount(type: Int): Int = playCounter[type] ?: 0

    private fun getDesByInt(type: Int): String = when(type) {
        0 -> "Song"
        1 -> "Dance"
        2 -> "Magic"
        else -> "null"
    }
}
```

着重注意不同方法中的`type`参数，参数名称各有不同，但本质都代表“表演类型”。而相同的参数名称则表示一一对应的关系：定义切点时，AspectJ 表达式内使用的参数名称为`whichType`，与标识该切点的空函数的参数是一致的；通知方法上的参数名称`typeIntValue`则与通知时机中引用切点时 AspectJ 表达式内的参数名称一致

测试类如下

```kotlin
@RunWith(SpringJUnit4ClassRunner::class)
@ContextConfiguration(classes = [ConcertConfig::class])
class PerformTest {

    @Autowired
    lateinit var performance: Performance

    @Test
    fun perform() {
        for (i in 0..4) {
            performance.perform(i % 3)
        }
    }
}
```

控制台输出为

```bash
Start perform: Song
=== type: Song, played 1 times ===
Start perform: Dance
=== type: Dance, played 1 times ===
Start perform: Magic
=== type: Magic, played 1 times ===
Start perform: Song
=== type: Song, played 2 times ===
Start perform: Dance
=== type: Dance, played 2 times ===
```

## 注入 AspectJ 切面

AspectJ 和 Spring 实际上是独立的，只不过 Spring AOP 借助了前者的指示器。通过一个例子展示如何注入原始的 AspectJ 切面。

首先准备一个评论员，在表演之后发表一段言论，其类型为`aspect`，在 IDEA 中可通过右键 New -> Aspect 新建一个该类型的文件

```java
public aspect CriticAspect {

    public CriticAspect() { }

    pointcut performance(): execution(* concert.MyPerformance.perform(..));

    after(): performance() {
        System.out.println("something");
    }

}
```

在 Spring 框架中，CriticAspect 将不会由容器创建，因为它属于 AspectJ 切面，由 AspectJ 在运行时创建。所以需要通过 AspectJ 切面提供的静态`aspectOf`方法给 Spring 返回切面的单例，Spring XML 配置写成以下形式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    <context:component-scan base-package="concert" />

    <aop:aspectj-autoproxy />

    <bean class="concert.CriticismEngineImpl" id="criticismEngine" />

    <bean class="concert.MyPerformance" id="performance" />

    <bean class="concert.CriticAspect" factory-method="aspectOf" />
</beans>
```

如果是 JavaConfig，则需要以下形式

```kotlin
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
open class ConcertConfig {
    @Bean
    open fun criticAspect(): concert.CriticAspect = org.aspectj.lang.Aspects.aspectOf(CriticAspect::class.java)
}
```

但是运行时 JRE 将不会识别类 CriticAspect，无法运行。目前未找到解决办法，故使用 XML 配置的方法

之后最重要的一步是使用`ajc`编译器，以编译`.aj`文件。通过一个插件[ Mojo's AspectJ Maven Plugin](https://www.mojohaus.org/aspectj-maven-plugin/usage.html) 引入`ajc`，注意此时 IDEA Compiler 的选项截图为

![](Spring-学习笔记（三）：面向切面\Compiler.png)

测试类如下

```kotlin
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:concert.xml"})
public class PerformTest {

    @Autowired
    Performance performance;

    @Test
    public void test() {
        performance.perform();
    }
}
```

按理说应该运行成功，然而此时还是报错

```bash
Error:ajc: can't find critical required type java.io.Serializable
Error:ajc: can't determine whether missing type java.io.Serializable is an instance of concert.CriticAspect
......
Error:ajc: can't find critical required type java.lang.Cloneable
Error:ajc: can't determine whether missing type java.lang.Cloneable is an instance of concert.CriticAspect
......

```

目前并无解决办法， V2EX 上的求助帖为 [诚心求助 Spring 注入式 AspectJ 切面时 ClassNotFoundException 的问题](
https://www.v2ex.com/t/570139#reply11)

Spring AOP 的内容到此为止，日后大概率会对本篇文章进行增删查改，继续进行下一步的学习吧

