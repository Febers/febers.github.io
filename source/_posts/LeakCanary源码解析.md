---
title: LeakCanary源码解析
date: 2019-02-13 13:28:31
tags:
- 内存泄漏
- Android
- 源码
categories:
- Android
---

内存泄漏是 Android 开发中无法避免的问题，[LeakCanary](https://github.com/square/leakcanary) 框架是 Square 公司开源的内存泄漏分析工具，其集成方便，使用便捷。<!--more-->

## ActivityLifecycleCallbacks
LeakCanary 的实现基础之一，是 Android 官方在4.0（API 14）之后引入的 ActivityLifecycleCallbacks，它是 Application 类的内部接口，提供应用生命周期回调的注册方法，以集中管理应用的生命周期。

### 接口方法
可以看到该接口定义了如下的 Activity 的生命周期回调方法，其与 Activity 的完整声明周期几乎是一一对应的。

```Java
    public interface ActivityLifecycleCallbacks {
        void onActivityCreated(Activity activity, Bundle savedInstanceState);
        void onActivityStarted(Activity activity);
        void onActivityResumed(Activity activity);
        void onActivityPaused(Activity activity);
        void onActivityStopped(Activity activity);
        void onActivitySaveInstanceState(Activity activity, Bundle outState);
        void onActivityDestroyed(Activity activity);
    }
```

### 简单用法
开发中如果我们想要统一管理项目中所有的 Activity，可以通过自定义 ActivityManager 实现，然后在 BaseActivity 中对每一个 Activity 做 put/get 操作，该做法的缺陷是无法控制一些第三方框架的 Activity。<br>此时我们可以使用 ActivityLifecycleCallbacks ，在自定义的 Application 中维护一个 Activity链表， 由于所有的 Activity 的生命周期都会回调该接口，就能够实现对所有 Activity 的统一控制和管理。
```Java
public class MyApplication extends Application {

    public static List<Activity> activityList;
    public static final int ACTIVITY_MAX_NUM = 10;

    @Override
    public void onCreate() {
        super.onCreate();
        activityList = new LinkedList<>();
        registerActivityLifecycleCallbacks(new MyActivityCallbacks());
    }

    class MyActivityCallbacks implements ActivityLifecycleCallbacks {

        @Override
        public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
            if (activityList.size() >= ACTIVITY_MAX_NUM) {
                activityList.remove(activityList.size()-1).finish();
            }
            activityList.add(activity);
        }

        @Override
        public void onActivityStarted(Activity activity) { }

        @Override
        public void onActivityResumed(Activity activity) { }

        @Override
        public void onActivityPaused(Activity activity) { }

        @Override
        public void onActivityStopped(Activity activity) { }

        @Override
        public void onActivitySaveInstanceState(Activity activity, Bundle outState) { }

        @Override
        public void onActivityDestroyed(Activity activity) {
            activityList.remove(activity);
        }
    }
    
    public static Activity getCurrentActivity() {
        return activityList.get(0);
    }
}
```

## 引用类型
在JDK 1.2以前的版本中，若一个对象不被任何变量引用，那么程序就无法再使用这个对象。也就是说，只有对象处于可触及（reachable）状态，程序才能使用它。从JDK 1.2版本开始，把对象的引用分为4种级别，从而使程序能更加灵活地控制对象的生命周期。这4种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

### 强引用
强引用（StrongReference）是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。比如经常使用的`A a = new A()`中的引用a。

### 软引用
软引用（SoftReference）用来描述一些有用但并不是必需的对象，只有在内存不足的时候 JVM 才会回收该对象。因此，这一点可以很好地用来解决OOM的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中。
```Java
SoftReference<String> sr = new SoftReference<String>(new String("hello"));
System.out.println(sr.get());
```

### 弱引用
弱引用（WeakReference）与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级低的线程，因此不一定会很快发现那些只具有弱引用的对象。
```Java
WeakReference<String> sr = new WeakReference<String>(new String("hello"));
         
System.out.println(sr.get());
System.gc();                //通知JVM的gc进行垃圾回收
System.out.println(sr.get());
```
打印的结果为
> hello
<br>null

弱引用可以和一个引用队列（ReferenceQueue）联合使用，当弱引用所引用的对象被垃圾回收器回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。
```Java
ReferenceQueue queue = new ReferenceQueue();
WeakReference pr = new WeakReference(object, queue);
```

### 虚引用
PhantomReference，顾名思义，就是形同虚设。与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，查看源码可以发现，其`get()`永远返回`null`

虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用、弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

```Java
public class PhantomReference<T> extends Reference<T> {

    public T get() {
        return null;
    }

    public PhantomReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
```

## LeakCanary源码
实际上 LeakCanary 正是利用 Activity 的生命周期回调，配合弱引用检测实现内存泄漏的分析。

### 执行流程
跟踪调用的入口方法`install`
```Java
public static @NonNull RefWatcher install(@NonNull Application application) {
    return refWatcher(application).listenerServiceClass(DisplayLeakService.class)
        .excludedRefs(AndroidExcludedRefs.createAppDefaults().build())
        .buildAndInstall();
}
```

`listenerServiceClass`方法位于 AndroidRefWatcherBuilder

```Java
public @NonNull AndroidRefWatcherBuilder listenerServiceClass(
      @NonNull Class<? extends AbstractAnalysisResultService> listenerServiceClass) {
      
    enableDisplayLeakActivity = DisplayLeakService.class.isAssignableFrom(listenerServiceClass);
    return heapDumpListener(new ServiceHeapDumpListener(context, listenerServiceClass));
}
```
该类的父类（RefWatcherBuilder）已经默认实现 heapDumpListener 和`excludedRefs`方法（定义一些开发者可以忽略的路径，即使发生了内存泄漏，LeakCanary 也不会弹出通知。大多是系统 Bug 导致）。下面来看最终调用的方法：

```Java
public @NonNull RefWatcher buildAndInstall() {
    if (LeakCanaryInternals.installedRefWatcher != null) {
        throw new UnsupportedOperationException("buildAndInstall() should only be called once.");
    }
    RefWatcher refWatcher = build();
    if (refWatcher != DISABLED) {
        if (enableDisplayLeakActivity) {
            LeakCanaryInternals.setEnabledAsync(context, DisplayLeakActivity.class, true);
        }
        if (watchActivities) {
            ActivityRefWatcher.install(context, refWatcher);
        }
        if (watchFragments) {
            FragmentRefWatcher.Helper.install(context, refWatcher);
        }
    }
    LeakCanaryInternals.installedRefWatcher = refWatcher;
    return refWatcher;
}
```
该方法主要做了三件事：Build RefWatcher；启用 DisplayLeakActivity， 用于显示性能统计结果；install ActivityRefWatcher。继续跟踪`install`方法

```Java
public static void install(@NonNull Context context, @NonNull RefWatcher refWatcher) {
    Application application = (Application) context.getApplicationContext();
    ActivityRefWatcher activityRefWatcher = new ActivityRefWatcher(application, refWatcher);

    application.registerActivityLifecycleCallbacks(activityRefWatcher.lifecycleCallbacks);
}

//成员变量 lifecycleCallbacks
private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
    new ActivityLifecycleCallbacksAdapter  {
        @Override public void onActivityDestroyed(Activity activity) {
            refWatcher.watch(activity);
        }
    };
```

由此可知 LeakCanary 内部实现了对 Activity 生命周期的监听，ActivityLifecycleCallbacksAdapter 其实是对  Application.ActivityLifecycleCallbacks 中所有方法都做了空实现的抽象类。

`watch`方法由 RefWatcher 默认实现：

```Java
public void watch(Object watchedReference, String referenceName) {
    ......  
    retainedKeys.add(key);
    final KeyedWeakReference reference =
            new KeyedWeakReference(watchedReference, key, referenceName, queue);

    ensureGoneAsync(watchStartNanoTime, reference);
}
```

其中 retainedKeys 是一个 Set<String> 集合，每个检测的对象都对应一个唯一的 key，存储在 retainedKeys 中。 
KeyedWeakReference 是 WeakReference 的子类，在其基础上添加了 key 和 name 两个属性用以跟踪记录。其中传入了一个 queue 参数，明显是引用队列。

```Java
private void ensureGoneAsync(final long watchStartNanoTime, final KeyedWeakReference reference) {
    watchExecutor.execute(new Retryable() {
        @Override public Retryable.Result run() {
            return ensureGone(reference, watchStartNanoTime);
        }
    });
}
```

watchExecutor 为 AndroidWatchExecutor 对象
```
public AndroidWatchExecutor(long initialDelayMillis) {
    mainHandler = new Handler(Looper.getMainLooper());
    HandlerThread handlerThread = new HandlerThread(LEAK_CANARY_THREAD_NAME);
    handlerThread.start();
    backgroundHandler = new Handler(handlerThread.getLooper());
    this.initialDelayMillis = initialDelayMillis;
    maxBackoffFactor = Long.MAX_VALUE / initialDelayMillis;
}

@Override public void execute(@NonNull Retryable retryable) {
    if (Looper.getMainLooper().getThread() == Thread.currentThread()) {
        waitForIdle(retryable, 0);
    } else {
        postWaitForIdle(retryable, 0);
    }
}
```
在`execute`中，不管是`waitForIdle`还是`postWaitForIdle`都会切换到主线程执行，最终会调用以下代码：
```Java
Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
      @Override public boolean queueIdle() {
        postToBackgroundWithDelay(retryable, failedAttempts);
        return false;
      }
    });
```
那么 IdleHandler 到底是什么呢？

我们都知道 Handler 是循环处理 MessageQueue 中的消息的，当消息队列中没有更多消息需要处理，且声明了 IdleHandler 接口的时候，就会去处理这里的操作。即指定一些操作，当线程空闲的时候来处理，此处执行的便是内存泄漏检测工作。

`ensureGoneAsync`方法最终会调用`ensureGone`

```Java
Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    ......
    removeWeaklyReachableReferences();
    ......
    gcTrigger.runGc();
    removeWeaklyReachableReferences();
    if (!gone(reference)) {
        ......
        File heapDumpFile = heapDumper.dumpHeap();
        ......
        HeapDump heapDump = heapDumpBuilder.heapDumpFile(heapDumpFile).referenceKey(reference.key)
            .referenceName(reference.name)
            .watchDurationMs(watchDurationMs)
            .gcDurationMs(gcDurationMs)
            .heapDumpDurationMs(heapDumpDurationMs)
            .build();

        heapdumpListener.analyze(heapDump);
    }
    return DONE;
}
```

- `removeWeaklyReachableReferences`遍历引用队列 queue，判断队列中是否存在当前 Activity 的弱引用，存在则删除 retainedKeys 中对应引用的 key值。
- 调用`gcTrigger.runGc`去进行内存回收，这里没有使用`System.gc`，是因为它仅仅是通知系统在合适的时间进行一次垃圾回收操作，并不能保证一定执行。
- 主动进行 GC 之后会再次调用`removeWeaklyReachableReferences`清除 retainedKeys 中弱引用的 key 值，再判断是否移除。如果仍然没有移除，判定为内存泄漏。
- 生成性能统计文件.hprof，进行内存泄漏的分析。

那么 hprof 文件是被解析成信息的呢

```Java
AnalysisResult result = heapAnalyzer.checkForLeak(heapDump.heapDumpFile, heapDump.referenceKey);
```

```Java
public AnalysisResult checkForLeak(File heapDumpFile, String referenceKey) {
    ......
    MemoryMappedFileBuffer e = new MemoryMappedFileBuffer(heapDumpFile);
    HprofParser parser = new HprofParser(e);
    Snapshot snapshot = parser.parse();
    this.deduplicateGcRoots(snapshot);
    Instance leakingRef = this.findLeakingReference(referenceKey, snapshot);
    return leakingRef == null ? AnalysisResult.noLeak(this.since(analysisStartNanoTime)) ：
        this.findLeakTrace(analysisStartNanoTime, snapshot, leakingRef);
}
```

`checkForLeak`方法使用了 Square 公司的另一个库 haha 来分析 Android heap dump，可以看到，执行转化的方法为 HprofParser 的`parse`方法，然后再将转化的信息封装成 AnalysisResult 对象，其中 HprofParser 是自定义的类，里面有一定的篇幅解析 .hprof 文件。

得到结果后回调给 DisplayLeakService，该 Service 会根据传入进来的数据发送通知，存入数据。点击对应的通知进入 DisplayLeakActivity 界面，就能查看泄漏日志。

### 总结
LeakCanary 检测内存泄漏的设计思路十分巧妙、清晰。整个框架对于 ActivityLifecycleCallbacks、WeakReference、ReferenceQueue 和 IdleHandler 的使用，四大组件的开启和关闭等等，都很有意思，值得深和借鉴。
