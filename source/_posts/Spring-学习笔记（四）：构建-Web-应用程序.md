---
title: Spring 学习笔记（四）：构建 Web 应用程序
date: 2019-06-03 18:56:48
tags:
- Spring
- Web
categories:
- Spring 
---

Spring MVC 基于模型-视图-控制器（ Model-View-Controller，MVC ）模式实现，用以构建灵活和松耦合的 Web 应用程序。本文将通过一个社交网站`Spittr`的实例，介绍 Spring MVC 框架，并在其基础上构建处理各种 Web 请求、参数和表单输入的控制器。

## Spring MVC

### 跟踪请求

![](Spring-学习笔记（四）：构建-Web-应用程序\spring-mvc-request.jpeg)

当用户在 Web 浏览器点击链接或提交表格的时候，发出的请求将通过一系列的路径，直到获取响应返回浏览器，这一过程在 Spring MVC 中的过程可以由上图展现

- 请求首先到达 DispatcherServlet，相当于一个前端控制器，其作用是查询处理器映射（Handler Mapping）后将请求发送给不同的 Spring MVC 控制器（Controller）
- 到达具体的控制器之后请求会卸下其负载信息，并等待控制器的处理结果
- 控制器完成逻辑处理后返回原始模型（Model）和视图名（View）
- DispatcherServlet 通过视图解析器（View Resolver）解析视图名得到匹配结果
- 请求最后到达视图实现（比如 JSP），交付模型数据
- 视图实现渲染输出模型并通过响应对象传递给客户端

### 搭建 Spring MVC

![](Spring-学习笔记（四）：构建-Web-应用程序\项目结构.png)

项目的结构图如上

Spring MVC 的核心是 DispatcherServlet，此处通过 Java 代码实现其配置，web.xml 文件的配置方式省略

```kotlin
class SpittrWebApplicationInitializer: AbstractAnnotationConfigDispatcherServletInitializer() {

    override fun getServletMappings(): Array<String> = arrayOf("/")
    
    override fun getRootConfigClasses(): Array<Class<*>> = arrayOf(RootConfig::class.java)

    override fun getServletConfigClasses(): Array<Class<*>> = arrayOf(WebConfig::class.java)
}
```

> 原理（特别绕）：拓展 AbstractAnnotationConfigDispatcherServletInitializer 的类都会自动配置 DispatcherServlet 和 Spring 应用上下文（由 ContextLoaderListener 实现），因为容器会在类路径中查找实现 ServletContainerInitializer 接口的实现并为其配置，而Spring 中的实现为 SpringServletContainerInitializer，这个实现又会查找 WebApplicationInitializer——关于这个接口的继承关系为
>
> > public abstract class AbstractContextLoaderInitializer implements WebApplicationInitializer
> >
> >
> > public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer
> >
> > public abstract class AbstractAnnotationConfigDispatcherServletInitializer extends AbstractDispatcherServletInitializer

我们重写了三个方法

- `getServletMappings`：将“/”路径映射到 DispatcherServlet 上，这也让其成为默认 Servlet
- `getRootConfigClasses`：该方法返回的带有`@Configuration`注解的类将会用来配置应用上下文中的 bean
- `getServletConfigClasses`：要求 DispatcherServlet 加载应用上下文时，使用定义在 WebConfig 中的 bean

RootConfig 和 WebConfig 的实现如下

```kotlin
@Configuration
@ComponentScan(basePackages = ["spittr"], excludeFilters = [ComponentScan.Filter(type = FilterType.ANNOTATION, value = [EnableWebMvc::class])])
open class RootConfig

@EnableWebMvc
@Configuration
@ComponentScan(value = ["spittr.web"])
open class WebConfig: WebMvcConfigurerAdapter() {

    @Bean
    open fun viewResolver(): ViewResolver = InternalResourceViewResolver().apply {
        setPrefix("/WEB-INF/views/")
        setSuffix(".jsp")
        setExposeContextBeansAsAttributes(true)
    }

    override fun configureDefaultServletHandling(configurer: DefaultServletHandlerConfigurer?) {
        configurer?.enable()
    }
}
```

RootConfig 使用`@ComponentScan`以实现对非 Web 组件的调用。

WebConfig 使用`@EnableWebMvc`启动 Spring MVC，使用`@ComponentScan`启用组件扫描，并继承 WebMvcConfigurerAdapter 重写其`configureDefaultServletHandling`方法以实现将静态资源的转发给默认 Servlet （貌似是多余的），同时添加一个 ViewResolver bean 查找对应的视图文件

## Spring 控制器

### 基本用法

在 Spring MVC 中，控制器是指添加了`@RequestMapping`注解的类，无论是在方法还是类上。控制器本身所带的`@Controller`则影响不大，如下面的代码。注解上的参数为数组，说明可以映射多个路径、多个请求类型。`home`返回的字符串将配合上面 WebConfig 的`viewResolver`方法寻找网页视图

```kotlin
@Controller
class HomeController {

    @RequestMapping(value = ["/"], method = [RequestMethod.GET])
    fun home(): String {
        return "home"
    }
}
```

现在就可以通过配置 Tomcat 运行本地服务器检验项目了，首先需要下载 Tomcat 包并解压，在 IDEA 中`EDIT Configurations`，添加一个 local 的 Tomcat，添加完成之后大概如下

![](Spring-学习笔记（四）：构建-Web-应用程序\tomcat_server.png)

接下来还有最重要的一部，否则会出现一个很奇怪的错误

```
java.lang.ClassNotFoundException: org.springframework.web.context.ContextLoaderListener
```

这是因为项目中通过 Maven 引入的库并没有被 Tomcat 识别，需要手动添加，如下图，点击左下角的编辑图标

![](Spring-学习笔记（四）：构建-Web-应用程序\tomcat_deploy.png)

在右侧的`Available Elements`中右键项目`spittr`，选择`Put into Output Root`，成功之后左侧的`WEB-INF/lib`下应该有需要的库

![](Spring-学习笔记（四）：构建-Web-应用程序\tomcat_put.jpg)

配置结束，需要注意的是不要导入 IDEA 生成的 Spring Web 框架，因为我们已经使用了 Maven 导入所有项目需要的类库，而且配置文件都是通过 Java 代码而没有引入任何的 XML 文件（除了 Maven 的`pom.xml`）。成功运行项目之后，浏览器打开`http://localhost:8080/spittr_war_exploded/`即可看到`home.jsp`中的网页视图

### 传递数据到视图中

首先建立数据模型，定义一条`Spittr`的内容为`Spittle`，同时通过一个仓库接口对外提供数据

```kotlin
data class Spittle(val id: Long, val message: String, val time: Date, val latitude: Double, val longitude: Double)

interface SpittleRepository {
    fun findSpittles(max: Long, count: Int): List<Spittle>
}

@Component
class SpittleRepositoryImpl: SpittleRepository {

    override fun findSpittles(max: Long, count: Int): List<Spittle> {
        return createSpittles(count)
    }

    private fun createSpittles(count: Int): List<Spittle> {
        val spittles: MutableList<Spittle> = ArrayList()
        for (i in 0 until count) {
            spittles.add(Spittle(id = i.toLong(), message = "This is spittle$i", time = Date(), latitude = Math.random(), longitude = Math.random()))
        }
        return spittles
    }
}
```

配置对应的控制器 SpittleController

```kotlin
@Controller
@RequestMapping("/spittles")
class SpittleController
@Autowired constructor(private val spittleRepository: SpittleRepository) {

    @RequestMapping(method = [RequestMethod.GET])
    fun spittles(model: Model): String {
        model.addAttribute("spittleList", spittleRepository.findSpittles(Long.MAX_VALUE, 20))
        return "spittles"
    }
}
```

相应的 JSP 页面为`spittles.jsp`

```xml
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>SpittleList</title>
</head>
<body>
    <c:forEach items="${spittleList}" var="spittle">
        <li id="spittle_<c:out value="spittle.id"/>">
            <div class="spittleMessage">
                <c:out value="${spittle.message}"/>
            </div>
            <div>
                <span lass="spittleTime">
                    <c:out value="${spittle.time}" />
                </span>
                <span class="spittleLocation">
                    (<c:out value="${spittle.latitude}" />
                    <c:out value="${spittle.longitude}" />)
                </span>
            </div>
        </li>
    </c:forEach>
</body>
</html>
```

需要注意的是，使用 JSTL 需要通过 Maven 引入两个库，参考[这里](https://howtodoinjava.com/spring-mvc/how-to-add-jstl-support-in-spring-3-using-maven/)——最重要的一步，将下载好之后的类库添加进 Tomcat lib 文件夹中

运行项目之后在浏览器打开`http://localhost:8080/spittr_war_exploded/spittles`即可查看效果

## 接受请求输入

Spring MVC 允许多种方式将客户端中的数据传送到控制器的处理方法中

- 查询参数（Query Parameter）
- 表单参数（Form Parameter）
- 路径变量（Path Parameter）

### 处理查询参数

添加一个新功能，让用户可以查看某一页的 Spittle 历史，为此需要获得一个 Spittle 的 ID——其恰好小于当前页最后一条 Spittle 的 ID。修改 SpittleController 中的方法 

```kotlin
    @RequestMapping(method = [RequestMethod.GET])
    fun spittles(
            @RequestParam("max", defaultValue = Long.MAX_VALUE.toString()) max: Long,
            @RequestParam("count", defaultValue = "20") count: Int,
            model: Model): String {
        model.addAttribute("spittleList", spittleRepository.findSpittles(max, count))
        return "spittles"
    }
```

现在客户端可以通过再链接后添加可选的查询参数获得不同的结果，比如`http://localhost:8080/spittr_war_exploded/spittles?count=5`将显示 5 条 Spittle

### 处理路径参数

假设应用程序需要根据给定的 ID 展现某一条 Spittle 记录，从面向资源的角度，这时 Spittle 应该通过 URL 路径标示，而不是通过查询参数，控制器添加新的方法，同时 SpittleRepository 添加一个查询单个 Spittle 的新接口

```kotlin
   @RequestMapping(value = ["/{spittleId}"])
    fun showSpittle(@PathVariable spittleId: Long, model: Model ): String {
        model.addAttribute("spittle", spittleRepository.findSpittle(spittleId))
        return "spittle"
    }

    override fun findSpittle(id: Long): Spittle {
        return Spittle(id = id, message = "This is spittle$id", time = Date(), latitude = Math.random(), longitude = Math.random())
    }
```

运行项目，现在就可以通过在原有路径上添加`/id`的方式访问特定的 Spittle，比如`http://localhost:8080/spittr_war_exploded/spittles/9527`

![](Spring-学习笔记（四）：构建-Web-应用程序\spittle_by_id.png)

### 处理表单

为 Spittr 提供用户注册和通过`username`查看个人信息的接口，为此首先要定义一个用户类型 Spitter 以及对应的 Repository。需要注意的是，使用 Kotlin 的`data class`时，属性要添加默认值并且要使用`var`而不能用`val`，不符合其中一个条件，提交表格时 Spring 都不能实例化 Spitter。同时还引入了`javax.validation.validation-api`包进行错误检验

```kotlin
data class Spitter(@NotNull @Size(min = 3, max = 16) var firstName: String = "",
              @NotNull @Size(min = 3, max = 16) var lastName: String = "",
              @NotNull @Size(min = 3, max = 16) var username: String = "",
              @NotNull @Size(min = 3, max = 16) var password: String = "")

interface SpitterRepository {
    fun save(spitter: Spitter)
    fun findByUsername(username: String): Spitter?
}

@Component
class SpitterRepositoryImpl: SpitterRepository {

    private val spitterMap: MutableMap<String, Spitter> = HashMap()

    override fun save(spitter: Spitter) {
        spitterMap[spitter.username] = spitter
    }

    override fun findByUsername(username: String): Spitter? = spitterMap[username]
}
```

添加一个控制器 SpitterController，实现下面的逻辑：用户打开`/spitter/register`页面时显示注册页面，其中包含一个表单。提交注册信息之后，判断注册信息有误错误，正确则重定向至`/spitter/username`路径，展示 Profile 信息

*奇怪的是当表单参数不符合注解要求时并不会触发 Error，尚不知道原因，已排除 Kotlin 代码的问题*

```kotlin
@Controller
@RequestMapping("/spitter")
class SpitterController
@Autowired constructor(private val spitterRepository: SpitterRepository) {

    @RequestMapping(value = ["/register"], method = [RequestMethod.GET])
    fun showRegistrationForm(): String = "registerForm"

    @RequestMapping(value = ["/register"], method = [RequestMethod.POST])
    fun processRegistration(@Validated spitter: Spitter, errors: Errors): String {
        if (errors.hasErrors()) {
            return "registerForm"
        }
        spitterRepository.save(spitter)
        return "redirect:/spitter/${spitter.username}"
    }

    @RequestMapping(value = ["/{username}"], method = [RequestMethod.GET])
    fun showSpitterProfile(@PathVariable username: String, model: Model): String {
        val spitter = spitterRepository.findByUsername(username)
        model.addAttribute("spitter", spitter)
        return "profile"
    }
}
```

需要准备两个 View 文件，`registerForm.jsp`和`profile.jsp`。第一个文件如下

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Spitter</title>
</head>
<body>
    <h1>Register</h1>

    <form method="post" action="register">
        First Name: <input type="text" name="firstName"><br>
        Last Name: <input type="text" name="lastName"><br>
        User Name: <input type="text" name="username"><br>
        Password: <input type="password" name="password"><br>

        <input type="submit" value="Register">
    </form>
</body>
</html>
```

第二个文件为

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Profile</title>
</head>
<body>
    <h1>Spitter Profile</h1>
    <c:out value="Username is ${spitter.username}" /><br>
    <c:out value="First name is ${spitter.firstName} " />
    <c:out value="and last name is ${spitter.lastName}" />
</body>
</html>
```

运行项目即可检验开发成果

---

最后再给出主页视图的文件`home.jsp`，关于本章的内容基本就是这样

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Spittr</title>
</head>
<body>
    <h1>Welcome to Spittr</h1>

    <a href="<c:url value="/spittles" />">Spittles</a> |
    <a href="<c:url value="/spitter/register" />">Register</a>
</body>
</html>
```

