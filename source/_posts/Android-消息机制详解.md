---
title: Android 消息机制详解
date: 2019-03-16 14:13:19
tags:
- Android 
- Handler
categories:
- Android 
---

## 引言
Android 的消息机制，主要是指 Handler 的运行机制。<!--more-->

## ANR
Application Not Responding，即应用程序无响应，在介绍消息机制的相关知识之前先了解 ANR。

### 原因 
Android系统中，ActivityManagerService(AMS) 和 WindowManagerService(WMS) 会检测 App 的响应时间，如果App在特定时间无法响应屏幕触摸或键盘输入事件，或者特定事件没有处理完毕，就会出现ANR。

以下四种条件都可以造成 ANR
- InputDispatching Timeout：5秒内无法响应屏幕触摸或键盘输入事件
- BroadcastQueue Timeout ：在执行前台广播（BroadcastReceiver）的`onReceive`方法中10秒没有处理完成，后台则为60秒。
- Service Timeout ：前台服务20秒内，后台服务在200秒内没有执行完毕。
- ContentProvider Timeout ：ContentProvider 的 publish 在10s内没进行完。

### 分析和解决
#### 分析
- 查看 log 信息
- Java 线程调用分析，`jstack {pid}`，其中 pid 为虚拟机进程 id，可以通过`jps`查看当前所有线程。
- 查看 trace.txt 文件，其导出目录为 Android SDK 的 /platform-tools 目录，Windows 下在该目录使用命令`./adb pull /data/anr/traces.txt`查看

#### 解决
- 避免死锁的出现，使用子线程来处理耗时操作或阻塞任务。
- 避免在主线程 query provider、不要滥用SharePreferences
- 文件读写或数据库操作放在子线程异步操作，操作完成之后及时关闭流。
- BroadcastReciever 的`onRecieve`不要进行耗时操作。

## Handler 机制
由上文我们知道，在主线程进行耗时操作将会引发 ANR，而且 Android 禁止在子线程中直接 UI 操作，该机制由 ViewRootImpl 的`checkThread`进行验证
```Java
void checkThread() {
    if (mThread != Thread.currrentThread()) {
        throw new CalledFromWrongThreadException(
            "Only the original thread that created a view hierarchy can
            touch its views");
    }
}
```
使用该机制的原因，一是多线程中并发访问 UI 会造成不可预知状态；二是对控件使用锁机制会降低 UI 访问的效率。
那么，当我们使用多线程技术结束数据的存储、获取，想回到主线程操作 UI 该怎么切换？

答案是使用 Handler 。

### 创建
```Java
//接收消息
@SuppressLint("HandlerLeak")
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        if (msg.what == 1) {
            log.e("MSG", "收到消息")；
        }
    }
};

//发送消息
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            mHandler.sendEmptyMessage(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).start();
```
使用成员变量创建 Handler 的方式很有可能造成内存泄漏，正确的做法是使用静态内部类和弱引用
```Java
MyHandler hander = new MyHandler(context);

static class MyHandler extends Handler {
    private WeakReference<Context> out;
    
    MyHandler(Context ctx) {
        super();
        out = new WeakReference<>(ctx);
    }

    @Override
    public void handleMessage(Message msg) {
        if (out.get() != null) {
            //进行消息处理
        }
    }
}
```

Handler 的运行机制底层由 MessageQueue 和 Looper 支撑。简单概括，Handler 把一个线程消息发送给当前线程的消息队列，即 MessageQueue，而 Looper 负责管理消息队列的。

### Handler
观察 Handler 的构造函数
```Java
public Handler(Callback callback, boolean async) {
    ......
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()  + " that has not called" 
            + " Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```
在其中给 MessageQueue 、Callback 和 Asynchronous 赋值。观察 if 语句，如果 mLooper 为 null，则抛出异常，其实就是无法在未调用`Looper.prepare`的线程内创建 handler。

不过，为什么在主线程中创建 Handler 不需要调用`Looper.prepare`和`Looper.loop`方法呢？其实是因为在创建 Activity 的时候，会经过一系列调用过程，最终执行 ActivityThread 的`main`方法，在其中调用了`prepareMainLooper`
```Java
public static void main(String[] args) {
        ......
        Looper.prepareMainLooper();
        ......
        Looper.loop();
        ......
}

public static void prepareMainLooper() {
    prepare(false);     //quitAllowed 参数传false
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```
关于 ActivityThread：[Android线程管理（二）——ActivityThread](http://www.cnblogs.com/younghao/p/5126408.html)

### MessageQueue
顾名思义，指消息队列。首先认识 Message，当用户在UI界面时点击一个按钮，该事件会被封装成一条 Message，被添加到 MessageQueue 中。Message 内部封装了一个属性 taget，其实质是一个 Handler 对象。

由于一个线程在一段时间只能对一种操作进行相关的处理，因此使用 MessageQueue 来管理所有消息的先后顺序，其内部维护一个单链表用以存储消息列表。

当 Handler 调用`sendMessage`时，最后会调用到

```Java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```
查看`enqueueMessage`方法，

```Java
boolean enqueueMessage(Message msg, long when) {
    ......
    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        ......
    }
    return true;
}
```
- 首先判断消息队列里有没有消息，没有则将当前插入的消息作为队头，并且如果此时消息队列处于等待状态则将其唤醒
- 如果队列已有消息，则根据 Message 创建的时间进行插入

### Looper
通常运行在一个消息的循环队列当中，线程默认不会提供消息循环去管理消息队列，需要在线程当中调用`Looper.prepare`方法使消息循环初始化，并且调用`Looper.loop`使消息循环一直处于运行状态，**取出 MessageQueue 中的消息分发给 Handler**。

```Java
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```
可以看到，`prepare(boolean quitAllowed)`实例化了一个 Looper ，然后将其设置进 sThreadLocal 中。当`Looper.prepare`执行完毕之后才可以执行`loop`方法
```Java
public static void loop() {
        //获取当前线程绑定的Looper
        final Looper me = myLooper();
        
        //当前线程的MessageQueue
        final MessageQueue queue = me.mQueue;
        ......
        //循环从 MessageQueue 取出消息.
        for (;;) {
            Message msg = queue.next();
            ......
            //将消息分发出去
            msg.target.dispatchMessage(msg);
            ......
            //将消息回收
            msg.recycle();
        }
}
```
可以看到，如果消息队列的 next 返回了新消息，就会调用`msg.target.dispatchMessage(msg)`，target 即为发送消息的 Handler 对象。成功将代码逻辑切换到指定的线程中执行。

```Java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
- 首先检查 Message 的 callBack 是否为空，不为空则`handlerCallback(msg)`，最终调用 callback 的`run`方法
- 如果为空，检查 mCallBack 是否为空，不为空则调用它的`handleMassage`，CallBack 是一个接口，利用它我们可以传一个 Callback 来创建 Handler。
- 如果都为空，调用 Handler 内部的`handleMessage`，也就是大多数时候我们创建 Handler 实例的时候需要重写的方法。

### ThreadLocal
一个线程内部的数据存储类，当某些数据是以线程为作用域而且不同线程具有不同的数据副本的时候，可以采用 ThreadLocal。对于 Handler 来说，它需要获取当前线程的 Looper，很明显不同的线程具有各自的 Looper，此时就可以通过 ThreadLocal 实现 Looper 在线程中的存取。如果不采用 ThreadLocal，那么系统就必须提供一个全局的哈希表供 Handler 查找 Looper，再提供一个类似 LooperManager 的类。

使用一个简单的例子演示 ThreadLocal 的真正含义
```Java
//新建一个 boolean 类型的变量
private ThreadLocal<Boolean> value = new ThreadLocal<>();

//在主线程中将其设为 true
value.set(true);
log.e("MainThread", value.get());

//子线程中设为 false
new Thread("Thread1") {
    @Override
    public void run() {
        value.set(false);
        log.e("Thread1", value.get());
    }
}

//另一个子线程直接读取
new Thread("Thread2") {
    @Override
    public void run() {
        log.e("Thread2", value.get());
    }
}
```
运行日志如下
> MainThread, true
> Thread1, false
> Thread2, null

由此可以看出，虽然访问的是同一个 ThreadLocal 对象，但是不同线程获取的值是不一样的。ThreadLocal 的实现原理：[Java并发编程：深入剖析ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)

## 总结

Handler 机制是如此重要，开发中我们总会显式或隐式的用到它，比如 Activity 的`runOnUiThread`

```Java
public final void runOnUiThread(Runnable action) { 
	if (Thread.currentThread() != mUiThread) { 
		mHandler.post(action); 
	} else { 
		action.run(); 
	} 
	...... 
}
```

比如 View 的`post`

```Java
public boolean post(Runnable action) {
	final AttachInfo attachInfo = mAttachInfo;
    	if (attachInfo != null) {
            return attachInfo.mHandler.post(action);         ①
        }
        // Assume that post will succeed later
        ViewRootImpl.getRunQueue().post(action);        ②
        return true;
    }
```



最后用一张图来结束本文

![流程图](Android-消息机制详解\消息流程图.jpg)