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

## 编写控制器

在 Spring MVC 中，控制器是方法或者类添加了`@RequestMapping`注解的类，其本身所带的`@Controller`影响不大，如下面的代码。当然我们也可以将`@RequestMapping`放在类上。注解上的参数为数组，说明可以映射多个路径。`home`返回的字符串将配合上面 WebConfig 的`viewResolver`方法寻找网页视图

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

## 传递数据到视图中

## 接受视图数据输入

