---
title: Kotlin 委托属性详解
date: 2019-05-21 16:52:14
tags:
- Kotlin
- 委托属性
categories: 
- Kotlin
---

委托属性算是 Kotlin 语言中的高级特性，初次接触可能毫无头绪，再次接触还是一脸懵逼。只有在深入理解其语言特性和实现原理之后，才能对这一甜之又甜的“语法糖”有所认识，从而极大提高代码效率。在笔者的项目 [UESTC_BBS](https://github.com/Febers/UESTC_BBS) 中，对自带 SharedPreferences 的封装 [PreferenceUtils ](https://github.com/Febers/UESTC_BBS/blob/master/app/src/main/java/com/febers/uestc_bbs/utils/PreferenceUtils.kt)就使用了委托属性，故一直想找机会写一篇笔记类型的文章将其记录下来。委托属性的基础是**委托**，一种设计模式，操作的对象不用自己执行，而是委托给另一个辅助对象。<!--more-->

## 委托

### 基本使用

定义一个委托属性的基本语法为 `val/var <属性名>: <类型> by <表达式>`，在 *by* 后面的表达式即为*委托*， 属性对应的 `get()`与`set()`会被委托给它的 `getValue()` 与 `setValue()` 方法。 

```kotlin
class Example {
    var p: String by Delegate()

    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            var e = Example()
            println(e.p)
            e.p = "newValue"
        }
    }
}

class Delegate {

    operator fun getValue(thisRef: Any?, property: Any): String {
        return "$thisRef, thank you for delegating '$property' to me!"
    }

    operator fun setValue(thisRef: Any?, property: Any, value: String) {
        println("$value has been assigned to '$property' in $thisRef.")
    }
}
```

上述代码中，Delegate 内方法的参数 property 原本为 KProperty<*> 接口类型，为了手动调用其方法，改成 Any 以实现传参。

控制台输出结果为

```bash
Example@1175e2db, thank you for delegating 'p' to me!
newValue has been assigned to 'p' in Example@1175e2db.
```

可以看到属性 p 委托给了 Delegate() 对象实例，按照约定，该对象必须声明具有`getValue`、`setValue`方法，且方法参数个数必须大于2、3。

为了更清楚的了解这一过程，可以将代码改写成另一种形式

```kotlin
class Example {

    var delegate: Delegate = Delegate()
    
    var p: String
        set(value) {
            delegate.setValue(thisRef = this, property = delegate, value = value)
        }
        get() = delegate.getValue(thisRef = this, property = delegate)
}
```

当我们使用 p 时调用其`get`方法，给 p 赋值时调用其`set`方法，并且都是通过委托对象 delegate 实现。

## 标准委托

### lazy

函数`lazy`接收一个 lambda 表达式并返回一个 `Lazy <T>` 实例，默认情况下其线程安全

```kotlin
//方法签名
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)


val s: String by lazy {
	println("get")
	"hello"
}

println(s)
println(s)
```

只有在第一次调用 s 时才会执行传递给 `lazy()` 的 lambda 表达式并返回一个记录下来的结果， 后续调用 `get()` 只会返回记录的结果。底层原理在于函数签名中的`SynchronizedLazyImpl`方法，其中会检查变量的值，判断其是否为默认值，如果是则执行初始化函数，否则直接返回结果，具体代码可以查阅`LazyJVM.kt`文件。

控制台输出为

```bash
get
hello
hello
```

### observable

字面意思，用委托的方式来定义一个可观察属性。该函数的方法签名为

```kotlin
public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit):
            ReadWriteProperty<Any?, T>
```

其接收两个参数，第一个为默认值，第二个为 lambda 表达式，位于`Delegates.kt`，具体使用

```kotlin
var name: String by Delegates.observable("initialValue") {
	property, oldValue, newValue ->
	println("$oldValue -> $newValue")
}
name = "newValue0"
name = "newValue1"
```

控制台输出

```bash
initialValue -> newValue0
newValue0 -> newValue1
```

### Storing

将对象的属性委托值 map 中，用于解析 JSON 或者其他动态工作，不过应用较少

```kotlin
val user = User(mapOf("name" to "Jack", "age" to 20))
println(user.name)
println(user.age)

class User(private val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}
```

## 典型应用

封装一个 SharedPreferences（简称 SP） 是 Android 开发中经常要做的事，因为直接调用 SP 足够繁琐。如果是 Java 代码，则代码基本如下

```java
public final class PreferencesUtil {
    private static PreferencesUtil sInstance;

    public static void init(Context context) {
        if (sInstance == null) {
            sInstance = new PreferencesUtil(context);
        }
    }

    public static PreferencesUtil getInstance() {
        if (sInstance == null) throw new RuntimeException("Uninitialized.");

        return sInstance;
    }

    private final SharedPreferences mSp;

    private PreferencesUtil(Context context) {
        mSp = PreferenceManager.getDefaultSharedPreferences(context);
    }

    public String getString(String key, String defValue) {
        return mSp.getString(key, defValue);
    }

    public void putString(String key, String value) {
        mSp.edit().putString(key, value).apply();
    }

    public int getInt(String key, int defValue) {
        return mSp.getInt(key, defValue);
    }

    public void putInt(String key, int value) {
        mSp.edit().putInt(key, value).apply();
    }

    public long getLong(String key, long defValue) {
        return mSp.getLong(key, defValue);
    }

    public void putLong(String key, long value) {
        mSp.edit().putLong(key, value).apply();
    }

    public float getFloat(String key, float defValue) {
        return mSp.getFloat(key, defValue);
    }

    public void putFloat(String key, float value) {
        mSp.edit().putFloat(key, value).apply();
    }

    public boolean getBoolean(String key, boolean defValue) {
        return mSp.getBoolean(key, defValue);
    }

    public void putBoolean(String key, boolean value) {
        mSp.edit().putBoolean(key, value).apply();
    }
}
```

外部调用

```java
if (PreferencesUtil.getInstance().getBoolean(Constant.IS_FIRST_LAUNCH, Constant.DEF_IS_FIRST_LAUNCH)) {
    // Do something first launch, like showing Welcome.
    ...

    PreferencesUtil.getInstance().putBoolean(Constant.IS_FIRST_LAUNCH, false);
}
```

使用 Kotlin 的委托属性之后实现就简洁很多

```kotlin
class PreferenceUtils<T>(val context: Context, val name: String, val default: T): ReadWriteProperty<Any?, T> {

    val prefs: SharedPreferences by lazy { context.defaultSharedPreferences }

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return findPreference(name, default)
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        putPreference(name, value)
    }

    private fun <T> findPreference(name: String, default: T): T = with(prefs) {
        val res: Any = when (default) {
            is Long -> getLong(name, default)
            is Int -> getInt(name, default)
            is String -> getString(name, default)
            is Boolean -> getBoolean(name, default)
            is Float -> getFloat(name, default)
            else -> throw IllegalArgumentException("This type can't be saved into Preferences")
        }
        return@with res as T
    }

    private fun <T> putPreference(name: String, value: T) = with(prefs.edit()) {
        when (value) {
            is Long -> putLong(name, value)
            is Int -> putInt(name, value)
            is String -> putString(name, value)
            is Boolean -> putBoolean(name, value)
            is Float -> putFloat(name, value)
            else -> throw IllegalArgumentException("This type can't be saved into Preferences")
        }.apply()
    }
}
```

其中接口`ReadWriteProperty`为系统提供的规范接口，其中定义了`getValue/setValue`方法。外部调用如下

```kotlin
var themeCode by PreferenceUtils(context, Constant.theme_code, default = 1)
themeCode = 9527
```



