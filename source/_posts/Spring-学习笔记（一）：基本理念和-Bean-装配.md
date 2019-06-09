---
title: Spring-学习笔记（一）：基本理念和-Bean-装配
date: 2019-05-28 22:13:09
tags:
- Spring 
- DI
- AOP
- Java EE
categories:
- Spring
---

很早就接触 Java 后端开发，不过大都浅尝辄止，并没有深刻地理解和实践，时间一长倒也跟没学过差不了多少。由于职业规划是要从 Android 开发慢慢转到后端开发，拖到现在终于是要系统学习了。学习的路线初步定为，从 Spring 出发，不断回顾 Java Web 的知识，重点在 Spring Boot 项目，通过具体的项目实践掌握相关知识。这篇文章属于第一篇学习笔记，参考手上的《Spring 实战》和网络上的技术文章，主要记录 Spring 的基本知识和理念、Bean  的装配。

![](https://github.com/spring-projects/spring-framework/raw/master/src/docs/asciidoc/images/spring-framework.png)

<!--more-->

## 关于 

### Spring

一个 Java EE 框架，同来替代更加重量级的企业级 Java 技术。其起源可以追溯到作者 Rod Johnson 2002年编写的《Expert One-to-One J2EE Design and Development》一书，书中他提出了一种基于普通 Java 类和依赖注入的解决方案。Rod Johnson 编写了超过30,000行基础结构代码，这便是最初的 Spring 框架。

Spring 的基本理念在于：简化 Java 开发，为此采用了4种关键措施：

- 基于 POJO 的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码

### 依赖注入

依赖注入（Dependency Injection，简称 DI）是最常见的控制反转（Inversion of Control，简称 IoC）方式，是面向对象编程中的一种设计原则，用于减少代码之间的耦合度。系统通过引入实现 IoC 模式的容器，管理对象的声明周期、依赖关系，常用的 IoC 容器有 Spring 、JBoss、EJB 等。可以把 IoC 模式看作工厂模式的升华，区别在于该工厂要生成的对象是通过 XML 文件、配置类或者注解来配置。

Java 项目大都是由众多类组成，这些类相互协作来完成特定的业务逻辑。传统的做法是每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，导致代码高度耦合。耦合具有两面性，一方面，紧密耦合的代码难以测试、难以复用、难以理解，并且修复 Bug 的过程中容易引发更多的 Bug；另一方面，一定的耦合又是必须的——完全无耦合的代码什么也做不了。通过 DI，对象的依赖关系交由系统中负责协调各对象的第三者在创建对象时设定，依赖关系被注入到需要它们的对象中。

通过一个简单的 Spring Demo 引入依赖注入的思想。

项目结构如下

```bash
├─src
│  ├─main
│  │  ├─java
│  │  │  └─knigt
│  │  │          BraveKnight.kt
│  │  │          SlayDragonQuest.kt
│  │  │          KnightConfig.kt
│  │  │          KnightMain.kt
│  │  │
│  │  └─resources
│  │          knights.xml

```

> 在 IDEA 内通过以下命令生成项目文档树
>
> tree  >>	D:/tree.txt 输出文件夹
> tree /f >>	D:/tree.txt 输出文件夹和文件

```kotlin
//BraveKnight.kt
interface Knight {
    fun embarkOnQuest()
}

class BraveKnight(private val quest: Quest): Knight {

    override fun embarkOnQuest() {
        quest.embark()
    }
}

//SlayDragonQuest.kt
interface Quest {
    fun embark()
}

class SlayDragonQuest(private val stream: PrintStream): Quest {

    override fun embark() {
        stream.println("Embarking on quest to slay the dragon!")
    }
}

//KnightConfig.kt
@Configuration
open class KnightConfig {

    @Bean
    open fun knight(): Knight = BraveKnight(quest())

    @Bean
    open fun quest(): Quest = SlayDragonQuest(System.out)
}

//KnightMain.kt
class KnightMain {
    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            val context: AnnotationConfigApplicationContext = AnnotationConfigApplicationContext(KnightConfig::class.java)
            //val context: ClassPathXmlApplicationContext = ClassPathXmlApplicationContext("knights.xml")   XML 方式配置
            val knight: Knight = context.getBean(Knight::class.java)
            knight.embarkOnQuest()
            context.close()
        }
    }
}
```

XML 的配置文件为

```xml
//knight.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>
```



输出结果为

```bash
Embarking on quest to slay the dragon!
```

### 面向切面编程

Aspect-Oriented Programming，AOP。DI 能够让相互协作的组件保持松耦合，而 AOP 则允许开发者把组件除自身核心功能之外，可重用的功能分离出来，比如日志、事务管理和安全等服务。AOP 思想中的相关术语包括

- Joinpoint：连接点，目标对象中所有可增强的方法
- Pointcut：切入点，目标对象中所有已增强的方法
- Advice：增强/通知，增强的代码
- Weaving：织入，将“通知”应用到切入点的过程
- Proxy：将“通知”织入到目标对象后，形成代理对象
- Aspect：切面，切入点+“通知”

使用一个传唱骑士事迹的吟游诗人服务类作为一个 AOP 的应用

```kotlin
//Minstrel.kt
open class Minstrel(private val stream: PrintStream) {
    open fun singBeforeQuest() {
        stream.println("Fa la la, the knight is so brave!")
    }

    open fun singAfterQuest() {
        stream.println("Tee hee hee, the brave knight did embark on a quest!")
    }
}
```
```html
//knights.xml 加入以下代码
<bean id="minstrel" class="Minstrel">
    <constructor-arg value="#{T(System).out}" />
</bean>

<aop:config>
	<aop:aspect ref="minstrel">
		<aop:pointcut id="embark" expression="execution(* *.embarkOnQuest(..))" />

		<aop:before pointcut-ref="embark" method="singBeforeQuest" />

		<aop:after pointcut-ref="embark" method="singAfterQuest" />
	</aop:aspect>
</aop:config>
```

*默认缺少 org.aspectj.aspectjweaver，可以引入 maven，添加对应的依赖*

运行之后，控制台输出结果为

```bash
Fa la la, the knight is so brave!
Embarking on quest to slay the dragon!
Tee hee hee, the brave knight did embark on a quest!
```

不使用 BraveKnight 和 Minstrel 组合的方式而是使用 AOP，通过少量的 XML 配置，就可以把 Minstrel 声明为一个 Spring 切面，进而实现相应的功能。Minstrel 仍然只是一个 POJO，但可以被应用到任何需要它的地方，只需要修改配置文件中的 AspectJ 切点表达式语言即可。

## 装配 Bean

### 容器

Spring 容器（Container）负责创建、装配和配置对象，并且管理它们的生命周期。Spring 的核心便是容器，它自带多个容器实现，可以归为两种不同的类型：Bean 工厂（由`org.springframework.beans.factory.BeanFactory`接口定义）是最简单的容器，提供基本的 DI 支持；应用上下文（由`org.springframework.context.ApplicationContext`接口定义），基于 BeanFactory 构建，并提供应用框架级别的服务，例如从属性文本解析文本信息以及发送应用事件给感兴趣的事件监听者。重点在于应用上下文上。

Spring 自带多种应用上下文

- AnnotationConfigApplicationContext：从一个或多个基于 Java 的配置类中加载 Spring 应用上下文
- AnnotationConfigWebApplicationContext：从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文
- ClassPathXmlApplicationContext：从类路径下的一个或多个 XML 配置文件中加载上下文定义，把应用上下文的定义文件作为类资源
- FileSystemXmlApplicationContext：从文件系统的一个或多个 XML 配置文件中加载上下文定义
- XmlWebApplicationContext：从 Web 应用下的一个或多个 XML 配置文件中加载上下文定义

### Bean 生命周期

![](Spring-学习笔记（一）：基本理念和-Bean-装配\生命周期.png)


1. Spring 对 bean 进行实例化
2. Spring 将值和 bean 的引用注入到 bean 对应的属性中
3. 如果 bean 实现了 BeanNameAware 接口，Spring 将 bean 的 ID 传递给`setBeanName`方法
4. 如果 bean 实现了 BeanFactoryAware 接口，Spring 将调用`setBeanFactory`方法，将 BeanFactory 容器实例传入
5. 如果 bean 实现了 ApplicationContextAware 接口， Spring 将调用`setApplicationContext`方法，将 bean 所在的应用上下文的引用传入
6. 如果 bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的`postProcessBeforeInitialization`方法
7. 如果 bean 实现了 InitializingBean 接口，Spring 将调用它们的 `afterPropertiesSet`方法。类似的，如果 bean 使用 init-method 声明了初始化方法，该方法也会被调用
8. 如果 bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的`postProcessAfterInitialzation`方法
9. 此时，bean 已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁
10. 如果 bean 实现了 DisposablBean 接口，Spring 将调用它的`destroy`接口方法。同样，如果 bean 使用 destroy-method 声明了销毁方法，该方法也会被调用

### 可选装配方案

有三种方式向 Spring 容器描述 Bean 如何装配

- 在 XML 文件中进行显示配置
- 在 Java 代码中进行显示配置
- 隐式的 bean 发现机制和自动装配

#### 自动化装配 bean

Spring 从两个角度实现自动化装配

- 组件扫描（Component Scanning）：Spring 会自动发现应用上下文中所创建的 bean
- 自动装配（Autowiring）：Spring 自动满足 bean 之间的依赖

通过一个 CD 播放的 Demo 说明，XML config 文件位于`/resources`目录下，Kotlin 文件位于`java/soundsystem`目录下

```kotlin
//conpactDisc.kt
@Component
class CompactDisc {

    private val title = "Sgt. Pepper's Lonely Hearts Club Band"
    private val artist = "The Beatles"

    fun play() {
        println("Playing $title by $artist")
    }
}

//CDPlayer.kt
@Component
class CDPlayer @Autowired
constructor(private val cd: CompactDisc) {

    fun play() {
        cd.play()
    }
}

//CDPlayerTest.kt
@RunWith(SpringJUnit4ClassRunner::class)
@ContextConfiguration(classes = [CDPlayerConfig::class])
class CDPlayerTest {
    @Autowired
    lateinit var player: CDPlayer

    @Test
    fun play() {
        player.play()
    }
}
```

以上便是准备好的工程文件，使用`@Autowired`注解在需要自动装配的任何属性、构造器、方法上，然后在 Java 代码或者 XML 配置文件启动自动装配。不同的自动装配方式需要修改 JUnit 测试框架的`@ContextConfiguration`注解参数

- XML 方式：`locations = ["classpath:spring-config.xml"]`

- Java 方式：`classes = [CDPlayerConfig::class]`

  

*JUnit 测试框架需要导入依赖*

接下来的一步便是启用 Spring 的组件扫描，也就是配置以上的两种注解参数之一，搭配 Bean 类注解`@Component`便可以实现组件扫描。以下两种做法都可以，第一种是使用一个`Config`类并添加注解`@Configuration`和`@ConponentScan`，第二种则是在 XML 文件中配置`context:component-scan`

```kotlin
//Java 代码
@Configuration
@ComponentScan
open class CDPlayerConfig
```

```xml
//XML 文件
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="soundsystem" />

</beans>
```

运行测试，控制台成功输出

```bash
Playing Sgt. Pepper's Lonely Hearts Club Band by The Beatles
```

#### Java 代码装配

大多数情况下都可以通过自动化装配的方式完成配置，但是当我们使用第三方的类库组件时，无法在组件类上直接添加注解，只能使用显示装配。这种情况下，使用 JavaConfig 是最好的方式。通常将 JavaConfig 放到同一个包中，并且它应该不包含任何的业务逻辑。

```kotlin
@Configuration
open class CDPlayerConfig {
    
    @Bean
    open fun sgtPeppers(): SgtPeppers = SgtPeppers()
    
    @Bean
    open fun cdPlayer(): CDPlayer = CDPlayer(sgtPeppers())
    
    @Bean
    open fun anotherCDPlayer(): CDPlayer = CDPlayer(sgtPeppers())
    
    @Bean
    open fun cdPlayerUseParam(compactDisc: CompactDisc): CDPlayer {
        val cdPlayer = CDPlayer(compactDisc)
        cdPlayer.setCompactDisc(compactDisc)
        return cdPlayer
    }
}
```

在上面的代码中，配置类`CDPlayerConfig`中提供了几种装配 Bean  的方式。`sgtPeppers`方法注入了一个 SgtPeppers 实例，`cdPlayer`和`anotherCDPlayer`需要调用该方法来注入 CDPlayer 实例。Spring 会拦截所有对该方法的调用，并直接返回该方法创建的 Bean，并且默认情况下 Bean 是单例的。

最后一个方法则展现了一个更具可读性、更多样化的注入方式。

#### XML 代码装配

最简单的 Spring XML 配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
</beans>
```

需要在配置文件顶部声明多个 XML 模式（XSD）文件，这些文件定义了配置 Spring 的 XML 元素。`<beans>`配置文件的根元素。

声明一个简单的 Bean 可以通过下面的方式实现

```xml
<bean id="cdPlayer" class="sound_system.CDPlayer">
	<constructor-arg ref="sgtPeppers"/>
</bean>
    
<bean id="sgtPeppers" class="sound_system.SgtPeppers" />
```

如果没有指定 id，该 Bean 将会根据全限定类名命名：`sound_system.CDPlayer#0`，`#0`为计数形式，用以区分相同类型的其他 Bean。

`&lt;constructor-arg&gt;`元素告知 Spring 要将一个 id 为`sgtPeppers`的 bean 引用传递到 CDPlayer 的构造器中。作为替代，也可以使用 Spring 的 c-命名空间，不再赘述。关于`&lt;constructor-arg&gt; `元素的更多知识：[IOC 之解析 bean 标签：constructor-arg、property 子元素](http://cmsblogs.com/?p=2754)

在该元素中，可以使用`value`属性将字面量传递给构造器，使用子元素`&lt;null/&gt;`空值，使用子元素`&lt;list&gt;`传递集合。

XML 配置也提供了设置属性的接口，通过元素`property`可以将对象引用、字面量、集合等注入 Bean 中。比如为 SgtPeppers 添加新的属性并对外提供修改接口

```kotlin
class SgtPeppers: CompactDisc {
    var title = "Sgt. Pepper's Lonely Hearts Club Band"
    var artist = "The Beatles"
    var tracks: MutableList<String> = ArrayList()
    
    override fun play() {
        println("Playing $title by $artist")
    }
}
```

然后在 XML 文件中注入属性设置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="sgtPeppers" class="sound_system.SgtPeppers">
        <property name="title" value="new title" />
        <property name="artist" value="new artist" />
        <property name="tracks">
            <list>
                <value>track 1</value>
                <value>track 2</value>
                <value>track 3</value>
                <value>track 4</value>
            </list>
        </property>
    </bean>

</beans>
```



### 导入和混合配置

当开发者使用自动装配的时候，Spring 会考虑容器中的所有 Bean，因此不用考虑混合配置的问题。混合配置应用在显式装配时，比如 XML 和 Java 互相引用配置的 Bean。

在 JavaConfig 中使用注解`@Import`导入其他 JavaConfig，使用注解`@ImportResource`混合 XML 配置，比如

```kotlin
@Configuration
@Import(CDPlayerConfig::class)
@ImportResource("classpath:sound_system.xml")
open class SoundSystemConfig 
```

在 XML 中使用元素`import`导入其他 XML 文件，而如果要导入 JavaConfig，做法很简单 ：将其声明为一个 bean 即可。

