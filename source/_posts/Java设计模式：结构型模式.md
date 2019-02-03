---
title: Java设计模式：结构型模式
date: 2019-01-21 00:56:04
categories:
- Java
---

## 适配器模式
在 Android 开发中，经常能见到各种各样的 Adapter 类，其采用的正是适配器模式。该设计模式作为两个不兼容的接口之间的桥梁，将一个类的接口转换成客户希望的另外一个接口，从而使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。<!--more-->

用电源接口举例，笔记本电脑的电源需要接入5V的电压，无法直接使用220v的交流电压，两者并不匹配，在软件开发中，该现象称之为接口不兼容，此时就需要适配器来进行接口转换。这个转换的中间层便是 Adapter 层。在这个例子中，我们需要5v的电压，因此`Volt5V`称为`Target`，而不兼容的220V电压称之为`Adaptee`，我们的目的是适配设计一个`Adpater`，方法有两种。

### 类适配器模式

```Java
interface Volt5V {
    int get5V();
}

class Volt220V {
    public int get220V() {
        return 220;
    }
}

class Adapter extends Volt220V implements Volt5V {
    @Override
    public int get5V() {
        return 5;
    }
}

public static void main(String[] args) {
    Adapter adapter = new Adapter();
    System.out.println("获取需要的5V电源：" + adapter.get5V());
}

```

### 对象适配器模式
与类适配器模式一样，对象适配器模式把被适配的类的API转换成为目标类的API，不同的是，对象适配器模式的连接方式不是继承，其使用代理关系连接到Adaptee类。 
　　 
```Java
class ObjectAdapter implements Volt5V {

    private Volt220V volt220V;

    public ObjectAdapter(Volt220V adaptee) {
        volt220V = adaptee;
    }

    @Override
    public int get5V() {
        return 5;
    }
        
    public int get220V() {
        return volt220V.get220V();
    }  
}
```
关于 Android 中 ListView 中的 Adapter，可以参考：[Android源码之ListView的适配器模式](https://blog.csdn.net/bboyfeiyu/article/details/43950185)

## 组合模式
将对象组合成树形结构来表现”部分-整体“的层次结构，使得客户以一致的方式处理单个对象以及对象的组合。<br>组合模式实现的最关键的地方是——简单对象和复合对象必须实现相同的接口。这就是组合模式能够将组合对象和简单对象进行一致处理的原因。

- 组合部件（Component）：它是一个抽象角色，为要组合的对象提供统一的接口。
- 叶子（Leaf）：在组合中表示子节点对象，叶子节点不能有子节点。
- 合成部件（Composite）：定义有枝节点的行为，用来存储部件，实现在Component接口中的有关操作，如增加（Add）和删除（Remove）。

对于透明组合模式来说，Component 中声明所有管理子对象的方法，其中包括Add，Remove等。这样做的好处是叶节点和枝节点具备完全一致的接口，对于外界没有区别。

```Java
//抽象构件，声明一个接口用于访问和管理Component的子部件
abstract class Component {

    public Component() { }

    public abstract void add(Component component);

    public abstract void remove(Component component);

    //显示层级结构
    public abstract void Display(int level);
}

//叶子节点
class Leaf extends Component {

    public Leaf() {
        super();
    }

    //无意义的实现
    @Override
    public void add(Component component) { }

    //无意义的实现
    @Override
    public void remove(Component component) { }

    @Override
    public void Display(int level) {
        System.out.println("-" + level);
    }
}

//枝节点
class Composite extends Component {

    public Composite() {
        super();
    }

    private List<Component> children = new ArrayList<>();

    @Override
    public void add(Component component) {
        children.add(component);
    }

    @Override
    public void remove(Component component) {
        children.remove(component);
    }

    @Override
    public void Display(int level) {
        children.forEach(
                component -> component.Display(level + 2)
        );
    }
}

```
对于这种方式来说，叶子节点并不具备 add、remove 等方法，对他们的实现是没有意义的，可以采用安全式组合模式，将叶子节点不具备的功能下放到枝节点中实现。该做法的弊端是对于客户端来说，必须对叶节点和枝节点进行判定，使用不便。

## 装饰器模式
向一个现有的对象添加新的功能，同时又不改变其结构。<br>
创建一个 Shape 接口和实现了 Shape 接口的实体类。然后创建一个实现了 Shape 接口的抽象装饰类 ShapeDecorator，并把 Shape 对象作为它的实例变量。
RedShapeDecorator 是实现了 ShapeDecorator 的实体类，用来装饰 Shape 对象。

```Java
interface Shape {
    void draw();
}

class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Draw a circle");
    }
}

//实现了 Shape 接口的抽象装饰类。
abstract class ShapeDecorator implements Shape {

    Shape decoratedShape;

    public ShapeDecorator(Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    @Override
    public void draw() {
        decoratedShape.draw();
    }
}

//具体的装饰类
class RedShapeDecorator extends ShapeDecorator {

    public RedShapeDecorator(Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    public void draw() {
        decoratedShape.draw();
        setRedBorder(decoratedShape);
    }

    private void setRedBorder(Shape decoratedShape) {
        System.out.println("Border color: Red");
    }
}
```

## 代理模式
下面的例子展示了静态代理的过程，创建一个 Image 接口和实现了 Image 接口的实体类。ProxyImage 是一个代理类，减少 RealImage 对象加载的内存占用。

```Java
interface Image {
    void display();
}

class RealImage implements Image {

    private String fileName;

    public RealImage(String fileName){
        this.fileName = fileName;
        loadFromDisk(fileName);
    }

    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }

    private void loadFromDisk(String fileName){
        System.out.println("Loading " + fileName);
    }
}

class ProxyImage implements Image {

    private RealImage realImage;
    private String fileName;

    public ProxyImage(String fileName){
        this.fileName = fileName;
    }

    @Override
    public void display() {
        if(realImage == null){
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

静态代理的缺点是显而易见的，由于代理类和委托类实现相同的接口，导致代码冗杂。接口的变化将引起所有实现类和代理类的变化，同时当代理类需要为多对象服务时也会显得力不从心，为此引入动态代理的方式。<br>
Java中的动态代理类必须实现 reflect 包中的 InvocationHandler 接口，同时使用 Proxy 类。当通过动态代理对象调用一个方法时候，该方法的调用就会被转发到实现 InvocationHandler 接口类的 invoke 方法。还是以静态代理中 Image 举例，这次不再创建一个 ProxyImage， 而是通过一个 DisplayHandler 实现动态代理。

```Java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DisplayHandler implements InvocationHandler {

    //要代理的真实对象
    private Object obj;

    public DisplayHandler(Object obj) {
        this.obj = obj;
    }

    /**
     *
     * @param proxy 代理类代理的真实代理对象
     * @param method 所要调用某个对象真实的方法的Method对象
     * @param args 指代代理对象方法传递的参数
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //真实的对象执行之前
        System.out.println("Before invoke...");

        Object invoke = method.invoke(obj, args);

        //真实的对象执行之后
        System.out.println("After invoke...");
        return invoke;
    }
}

public static void main(String[] args) {
    Image image = new RealImage("hello.jpg");
    InvocationHandler handler = new DisplayHandler(image);

    /*
     * 参数一：handler.getClass().getClassLoader()，使用handler对象的classloader对象来加载我们的代理对象
     * 参数二：image.getClass().getInterfaces()，提供真实对象实现的接口，使代理对象能调用接口中的所有方法
     * 参数三：handler，将代理对象关联到上面的InvocationHandler对象上
     */
    Image proxy = (Image) Proxy.newProxyInstance(
            handler.getClass().getClassLoader(),
            image.getClass().getInterfaces(),
            handler);
    proxy.display();
}
```
控制台输出结果为 
```
Loading hello.jpg
Before invoke...
Displaying hello.jpg
After invoke...
```

代理模式和装饰模式非常类似，二者最主要的区别是：代理模式中，代理类对被代理的对象有控制权，决定其执行或者不执行。而装饰模式中，装饰类对代理对象没有控制权，只能为其增加一层装饰，以加强被装饰对象的功能，仅此而已。

## 其他

### 过滤器模式
简单的说，筛选器提供一个输入参数为特定类型集合的筛选方法，返回筛选之后的集合，同时筛选器之间可以组合。

### 桥接模式
桥接模式即将抽象部分与它的实现部分分离开来，使他们都可以独立变化。

举个例子，在画画这一行为中，画笔需要选择不同的颜色、绘画形状等，为了方便拓展，使用桥接模式，抽象出形状和颜色两个父类，然后根据需要对颜色和形状进行组合。实际上这是“面向接口编程”设计思想最直观的体现。

### 外观模式
外观模式隐藏系统的复杂性，并向客户端提供了一个可以访问系统的接口。

简单的来说就是对外提供一个简单接口，隐藏实现的逻辑。比如计算机的电源键，用户只需按电源键，就可以启动或者关闭计算机，无需知道它是怎么启动的(启动CPU、启动内存、启动硬盘)，怎么关闭的(关闭硬盘、关闭内存、关闭CPU)。


### 享元模式
所谓享元模式就是运行共享技术有效地支持大量细粒度对象的复用。系统使用少量对象,而且这些都比较相似，状态变化小，可以实现对象的多次复用。通过其中的享元工厂来展示该模式的特点

```Java
public class FlyweightFactory{

    static Map<String, Shape> shapes = new HashMap<String, Shape>();
    
    public static Shape getShape(String key){
        Shape shape = shapes.get(key);
        if(shape == null){
            shape = new Circle(key);
            shapes.put(key, shape);
        }
        return shape;
    }
    
    public static int getSum(){
        return shapes.size();
    }
}
```

### 补充
从代码上看，结构型模式中的很多模式具有相当大的相似性，具体区分的话，装饰器模式是“新增行为”，代理模式是“控制访问行为”，适配器模式是"转换行为"，外观模式是一种"简化行为"。