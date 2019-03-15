---
title: Java设计模式：行为型模式
date: 2019-01-21 00:57:00
tags:
- 设计模式
- Java
categories:
- Java
---

## 责任链模式
责任链模式（Chain of Responsibility Pattern）为请求创建一个接收者对象的链，每个接收者都包含对另一个接收者的引用。如果不能处理该请求，那么它会把相同的请求传给下一个接收者。该模式在 Java Web中有很多应用，如 Apache Tomcat 对 Encoding 的处理，Struts2 的拦截器，JSP Servlet 的 Filter等等。<!--more-->

创建抽象类 AbstractLogger，带有详细的日志记录级别，在其基础上拓展出三个记录器，如果消息的级别属于自己，则记录器将其打印，否则把消息传给下一个记录器。

```Java
abstract class AbstractLogger {
    public static int INFO = 1;
    public static int DEBUG = 2;
    public static int ERROR = 3;

    protected int level;

    //责任链中的下一个元素
    protected AbstractLogger nextLogger;

    public void setNextLogger(AbstractLogger nextLogger) {
        this.nextLogger = nextLogger;
    }

    public void logMessage(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }

    abstract protected void write(String message);
}
```
在 DebugLogger 中
```Java
class DebugLogger extends AbstractLogger {

    public DebugLogger() {
        super();
        level = 2;
    }

    @Override
    protected void write(String message) {
        System.out.println("DebugLogger: " + message);
    }
}
```
调用
```Java
    static AbstractLogger getChainOfLogger() {
        AbstractLogger infoLogger = new InfoLogger();
        AbstractLogger debugLogger = new DebugLogger();
        AbstractLogger errorLogger = new ErrorLogger();
        
        infoLogger.setNextLogger(debugLogger);
        debugLogger.setNextLogger(errorLogger);
        return infoLogger;
    }
    
    public static void main(String[] args) {
        AbstractLogger logger = getChainOfLogger();

        logger.logMessage(AbstractLogger.INFO, "an info msg");
        logger.logMessage(AbstractLogger.DEBUG, "a debug msg");
        logger.logMessage(AbstractLogger.ERROR, "an error msg");
    }
```
控制台输出结果为
```
InfoLogger: an info msg
InfoLogger: a debug msg
DebugLogger: a debug msg
InfoLogger: an error msg
DebugLogger: an error msg
ErrorLogger: an error msg
```

## 其他

### 命令模式
一种数据驱动的设计模式，请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并将命令传给它以执行命令。调用对象寻找合适对象的过程就像开关控制电器，它并不需要知道电器的具体情况，只需要根据命令，控制不同电线的连通状态。模式实现的代码较繁琐，具体的例子移步：[Java设计模式--命令模式（以管理智能家电为例）](http://cache.baiducontent.com/c?m=9f65cb4a8c8507ed19fa950d100b92235c4380146d8b804b2281d25f93130a1c187babf37d714c518a82213a1cfc091ab1a16825761e2bb490c38f40d7ac925f75ce786a6459db0144dc41fc8f1532c050872be8b86d96ad813184d9a5c4de2444bb55120981e7fa291764bc&p=8b2a971c87dd11a05db0e63c49&newp=882a900d959603ee19be9b7c4553d8224216ed6039d0c44324b9d71fd325001c1b69e7bf20271707d7ce786d0ba54f5beefa3476301766dada9fca458ae7c4606cdd657531&user=baidu&fm=sc&query=java%C3%FC%C1%EE%C4%A3%CA%BD&qid=c0e757ac00089354&p1=5)
