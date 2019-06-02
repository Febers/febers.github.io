---
title: Spring 学习笔记（二）：高级装配
date: 2019-06-01 16:17:51
tags:
- Spring
categories:
- Spring
---

开发软件的过程中要涉及到不同的环境，需要配置不同的数据库配置、加密算法和外部环境的集成等组件，为此必须要考虑不同的环境下对应不同的配置。本文将从 Spring profile 出发，进而介绍条件化的 Bean 声明、自动装配的歧义性和 Bean 的作用域，以及 Spring 表达式语言。<!--more-->

## Spring profile

### 配置 profile

使用注解`@Profile`指定某个 Bean 属于哪一个 profile，以不同环境下的数据库 Bean 为例

```kotlin
@Configuration
open class DataSourceConfig {
    
    @Bean
    @Profile("dev")
    open fun embeddedDataSource(): DataSource = EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("classpath:schema.sql")
            .build()

    @Bean
    @Profile("prod")
    open fun jndiDataSource(): DataSource {
        val jndiObjectFactoryBean = JndiObjectFactoryBean()
        jndiObjectFactoryBean.jndiName = "jndi/myDS"
        jndiObjectFactoryBean.isResourceRef = true
        jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource::class.java)
        return jndiObjectFactoryBean.`object` as DataSource
    }
}
```

同样支持使用 XML 文件配置 profile

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd"
       profile="dev">
    <jdbc:embedded-database id="dataSource2">
        <jdbc:script location="classpath:schema.sql" />
    < /jdbc:embedded-database>
</beans>
```

可以将  profile 设置为`prod`，创建适用于生产环境的 JNDI 获取的 DataSource Bean，所有的配置文件都会被放进部署单元之中（如 WAR 文件），但是只有 profile 属性与当前激活 profile 相匹配的配置文件才会被使用。

不过更方便的方法是把所有的 Bean 放进同一个配置文件中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:jee="http://www.springframework.org/schema/jee"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee.xsd">

    <beans profile="dev">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:schema.sql" />
        </jdbc:embedded-database>
    </beans>

    <beans profile="prod">
        <jee:jndi-lookup id="dataSource"
                         jndi-name="jdbc/myDS"
                         resource-ref="true"
                         proxy-interface="javax.sql.DataSource" />
    </beans>
</beans>
```

### 激活 profile

Spring 借助两个独立的属性`spring.profiles.active`、`spring.profiles.default`来确定哪个 profile 处于激活状态，优先级从高到低。如果两个值都没有设置，那就没有激活的 profile，此时只会创建那些没有定义在 profile 中的 Bean。以下是设置这两个属性的方式

- 作为 DispatcherServlet 的初始化参数
- 作为 Web 应用的上下文参数
- 作为 JNDI 条目
- 作为环境变量
- 作为 JVM 的系统属性
- 在集成测试类上，使用`@ActiveProfiles`注解设置

例如在 Web 应用中，设置`spring.profiles.default`的 web.xml 文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <context-param>
        <param-name>spring.profiles.default</param-name>
        <param-value>dev</param-value>
    </context-param>
    
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>spring.profiles.default</param-name>
            <param-value>dev</param-value>
        </init-param>
    </servlet>
</web-app>
```

该文件分别为上下文和 Servlet 设置了默认的 profile。当应用程序部署到 QA、生产或者其他环境中时，负责部署的人根据情况使用系统属性、环境变量或 JNDI 设置`spring.profiles.active`即可——其优先级比`spring.profiles.default`高

## 条件化的 Bean 

使用 Spring 提供的注解`@Conditional`，可以实现 Bean 在符合条件的情况下才会配置的效果，不符合条件则该 Bean 会被忽略。下面的例子假设只有设置了`magic`环境属性时 Spring 才会实例化 MagicBean（事先准备），否则它将被忽略。

```kotlin
@Configuration
open class MagicConfig {

    @Bean
    @Conditional(MagicExistsCondition::class)
    open fun magicBean(): MagicBean = MagicBean()

}
```

`@Conditional`需要一个`class`参数，并且该类要实现`Condition`接口，重写方法`match`，如果该方法返回 true 则创建带有`@Conditional`的 Bean。

```kotlin
class MagicExistsCondition: Condition {
    
    override fun matches(p0: ConditionContext?, p1: AnnotatedTypeMetadata?): Boolean {
        val env = p0?.environment
        return env?.containsProperty("magic") ?: false
    }
    
}
```

该方法传入两个参数，第一个为`ConditionContext`

```java
public interface ConditionContext {
    BeanDefinitionRegistry getRegistry();

    ConfigurableListableBeanFactory getBeanFactory();

    Environment getEnvironment();

    ResourceLoader getResourceLoader();

    ClassLoader getClassLoader();
}
```

可以看到该参数为我们提供了很多接口。第二个参数为`AnnotatedTypeMetadata`

```java
public interface AnnotatedTypeMetadata {
    boolean isAnnotated(String var1);

    Map<String, Object> getAnnotationAttributes(String var1);

    Map<String, Object> getAnnotationAttributes(String var1, boolean var2);

    MultiValueMap<String, Object> getAllAnnotationAttributes(String var1);

    MultiValueMap<String, Object> getAllAnnotationAttributes(String var1, boolean var2);
}
```

该参数为我们提供了一系列检查`@Bean`方法上的其他注解的接口

观察第一节提到的`Profile`注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({ProfileCondition.class})
public @interface Profile {
    String[] value();
}
```

可以看到其本身也使用了`@Conditional`，引用`ProfileCondition.class`作为 Condition 实现

```java
class ProfileCondition implements Condition {
    ProfileCondition() {
    }

    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        if (context.getEnvironment() != null) {
            MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
            if (attrs != null) {
                Iterator var4 = ((List)attrs.get("value")).iterator();

                Object value;
                do {
                    if (!var4.hasNext()) {
                        return false;
                    }

                    value = var4.next();
                } while(!context.getEnvironment().acceptsProfiles((String[])((String[])value)));

                return true;
            }
        }

        return true;
    }
}
```

该类通过 AnnotatedTypeMetadata 获得用于`@Profile`注解的所有属性，然后检查`value`属性，得到`profile`名称。然后通过 ConditionContext 检查该`profile`是否激活

## 自动装配的歧义性

在自动装配中，只有当仅有一个 Bean 匹配所需的结果时，自动装配才是有效的，出现歧义则将阻碍 Spring 自动装配属性、构造器参数或方法参数。

比如下面的例子，提供两个 CompactDisc 的实现类

```kotlin
interface CompactDisc {
    fun play()
}

@Component
class SgtPeppers: CompactDisc {
	......
}

@Component
class BlankDisc: CompactDisc {
	......
}
```

测试代码为

```kotlin
@RunWith(SpringJUnit4ClassRunner::class)
@ContextConfiguration(classes = [CDPlayerConfig::class])
class CDPlayerTest {

    private lateinit var cd: CompactDisc

    @Test
    fun play() {
    	cd.play()
    }

    @Autowired
    fun setCompactDisc(cd: CompactDisc) {
        this.cd = cd
    }
}
```

测试将会报错

```bash
org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'soundsystem.CompactDisc' available: expected single matching bean but found 2: blankDisc,sgtPeppers
```

针对这种情况，可以使用`@Primary`和`@Qualifier`解决

### @Primary

在其中一个实现上添加注解

```kotlin
@Component
@Primary
class SgtPeppers: CompactDisc {
	......
}
```

该注解也可以应用在 JavaConfig 代码中，在 XML 文件的配置方式为

```xml
<bean id="sgtPeppers" class="sound_system.SgtPeppers"
    primary="true">
</bean>

```

### @Qualifier

该注解比`@Primary`更加灵活，使用简单

```kotlin
@Autowired
@Qualifier("sgtPeppers")
fun setCompactDisc(cd: CompactDisc) {
	this.cd = cd
}
```

为`@Qualifier`注解设置的参数就是想要注入的 Bean 的 id，一般为 Bean 的类名首字母变为小写之后的字符。当然开发者也可以自定义 Bean 的限定符防止重命名之后原来的注解失效

```kotlin
@Component
@Qualifier("byBeatles")
class SgtPeppers: CompactDisc {

    private val title = "Sgt. Pepper's Lonely Hearts Club Band"
    private val artist = "The Beatles"

    override fun play() {
        println("Playing $title by $artist")
    }
}
```

相应的代码也更改为

```kotlin
@Autowired
@Qualifier("byBeatles")
fun setCompactDisc(cd: CompactDisc) {
	this.cd = cd
}
```

更进一步，开发者可以自定义一个使用`@Qualifier`注解的注解类，防止以后再出现一个`byBeatles`的 Bean

```kotlin
import org.springframework.beans.factory.annotation.Qualifier

@Target(AnnotationTarget.CONSTRUCTOR, AnnotationTarget.FIELD, AnnotationTarget.TYPE, AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
@Qualifier
annotation class FavoriteCD 
```

使用方法和`@Qualifier("id")`类似，在需要注解的类上添加`@FavoriteCD`，然后在自动装配的地方同样使用该注解即可。

## Bean 的作用域

 在上一篇文章中我们提到 Spring 中的 Bean 默认是单例的，但是显然这样的实例将会保持一定的“状态”，因此重用将是不安全的。Spring 定义了多种 Bean 作用域

- 单例（Singleton）：整个应用中只创建 Bean 的一个实例
- 原型（Prototype）：每次注入或者通过 Spring 应用上下文获取的时候都会创建一个新的实例
- 会话（Session）：在 Web 应用中，为每个会话创建一个实例
- 请求（Request）：在 Web 应用中，为每个请求创建一个实例

要使用作用域，可以通过`@Scope`注解实现

```kotlin
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
class SgtPeppers
```

当然在自动装配的地方使用该注解也可以，XML 文件的配置方式同理，不再赘述

### 会话和请求作用域

在典型的电子商务应用中，可能有一个 Bean 代表用户的购物车，当其为单例时意味着所有的用户共用一个购物车。这种情况下使用会话作用域显然更加合适

```kotlin
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION,
        proxyMode = ScopedProxyMode..TARGET_CLASS)
open class ShoppingCart
```

注解的第一个参数`value`将告诉 Spring 为 Web 应用的每个会话创建一个 ShoppingCart，在会话相关的操作中其相当于一个单例。第二个参数`proxyMode`被设为`ScopedProxyMode.INTERFACES`，解决将会话或者请求作用域的 Bean 注入到单例 Bean 中所遇到的问题

假设 ShoppingCart 注入到一个单例 StoreService 中，由于单例 Bean 会在 Spring 应用上下文加载的时候创建，然而此时属于会话作用域的 ShoppingCart 并不存在；此外，系统中将会有多个 ShoppingCart 实例，我们并不想让某一个固定的 ShoppingCart 实例到 StoreService 中，而是希望当 StoreService 处理购物车功能时，其所使用的 ShoppingCart 实例刚好是当前会话所对应的那个

`proxyMode`的作用就在于，Spring 不会将实际的 ShoppingCart 注入到 StoreService 中，而是将它的代理注入。这一代理会暴露与 ShoppingCart 相同的方法，所以 StoreService 会认为它就是一个购物车。当 StoreService 调用方法时，代理会进行懒解析并将调用委托到会话作用域内真正的 ShoppingCart 实例

XML 方式配置不再赘述

## Spring 表达式语言

Spring Expression Language，简称 SpEL，能够以一种强大和简洁的方式将值装配到 Bean 属性和构造器参数中。其有很多特性

- 使用 ID 来引用 Bean
- 调用方法和访问对象的属性
- 对值进行算数、关系和逻辑运算
- 正则表达式匹配
- 集合操作

SpEL 要放到`#{ ... }`中，与属性占位符放到`${ ... }`中类似。对于下面的表达式

```kotlin
#{T(System).currentTimeMillis()}
```

将得到表达式计算的那一刻时间的毫秒数。`T()`表达式将其中参数视为 Java 中的类型，从而可以调用该类型的静态方法。

`#{ ... }`中既可以放字面值如整数、浮点数、字符串、布尔值，也可以引用对象属性和方法

```kotlin
#{sgtPeppers.artist}

#{systemProperties['sgtPeppers.title']}

#{sgtPeppers.toString()?.toUpperCase}
```

上面的第二个表达式将调用`properties`文件中`sgtPeppers`的`title`属性值。第三个参数表示我们可以对方法返回值使用安全调用

SpEL 支持各种常用的运算，包括算术运算、比较运算、逻辑运算、条件运算（`? :`和`?:`）、正则表达式（`matches`），下面是一个正则表达式的例子，用以验证邮箱

```kotlin
#{admin.email matches '[a-zA-Z0-9._&+-]+@[a-zA-Z0-9._&+-]'+\\.com}
```

下面是一个操作集合的例子

```kotlin
#{cd.tracks[1].title}

#{'the string'[1]} //得到 h

#{cd.tracks.?[title eq 'Title'} //得到 title 为 Title 的新集合

#{cd.tracks.^[title eq 'Title'} //得到第一个 title 值为 Title 的元素
              
#{cd.tracks.$[title eq 'Title'} //得到最后一个 title 值为 Title 的元素             

#{cd.tracks.![title]} //得到 title 的集合
         
```



