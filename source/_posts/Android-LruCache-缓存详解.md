---
title: Android LruCache 缓存详解
date: 2019-03-17 10:49:27
tags:
- Android
- LRU
- 缓存
categories:
- Android 
---

![](https://upload-images.jianshu.io/upload_images/3392635-584d9b129fd4132c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
<!--more-->
## Android 缓存策略
一般来说，缓存策略主要包含缓存的添加、获取和删除。如何添加和获取缓存这个比较好理解，那为什么还要删除缓存呢？这是因为不管是内存缓存还是硬盘缓存，它们的缓存大小都是有限的。当缓存满了之后，再向其添加缓存，就需要先删除旧的缓存。因此 LRU 缓存算法应运而生。

LRU（Least Recently Used），最近最少使用算法，核心思想是当缓存满时，优先淘汰那些近期最少使用的缓存对象，有效的避免了 OOM 的出现。Android 中采用 LRU 算法的常用缓存有两种：LruCache 和 DisLruCache，分别用于实现内存缓存和硬盘缓存。

LRU 缓存的实现类似于一个特殊的栈，把访问过的元素放置到栈顶（若栈中存在，则更新至栈顶；若栈中不存在则直接入栈），然后如果栈中元素数量超过限定值，则删除栈底元素（即最近最少使用的元素）。如下图：

![](https://upload-images.jianshu.io/upload_images/3392635-bb6c6461e8d01701.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/291/format/webp)

## 使用
LruCache 是Android 3.1所提供的一个缓存类，可以直接使用。而  DisLruCache  目前还不是 Android SDK的一部分，但 Android 官方文档推荐使用该算法来实现硬盘缓存。

讲到 LruCache 不得不提一下 LinkedHashMap，因为 LruCache 中 Lru 算法就是通过 LinkedHashMap 来实现的。

LinkedHashMap 继承于 HashMap，使用了一个双向链表来存储 Map 中的 Entry 顺序关系，这种顺序有两种，一种是 LRU 顺序，一种是插入顺序，由其构造函数

```java
public LinkedHashMap(int initialCapacity,float loadFactor, boolean accessOrder)
```

中的最后一个参数 accessOrder 来指定。

对于`get`、`put`、`remove`等操作，LinkedHashMap 除了要做 HashMap 做的事情，还会做些调整 Entry 顺序链表的工作。LruCache 中将 LinkedHashMap 的顺序设置为 LRU 顺序来实现 LRU 缓存，每次调用 `get`(也就是从内存缓存中取图片)，则将该对象移到链表的尾端。调用`put` 插入新的对象，则存储在链表尾端，这样当内存缓存达到设定的最大值时，将链表头部的对象（近期最少用到的）移除。

![](https://upload-images.jianshu.io/upload_images/3392635-af28ceea733149ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/650/format/webp)

LruCache 的使用非常简单，以图片缓存为例：
```Java
int maxMemory = (int) (Runtime.getRuntime().totalMemory()/1024);
int cacheSize = maxMemory/8;

mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight() / 1024;
    }
};
```

1. 设置LruCache缓存的大小，一般为当前进程可用容量的1/8。
2. 重写sizeOf方法，计算出要缓存的每张图片的大小。

注意：缓存的总容量和每个缓存对象的大小所用单位要一致。

## 原理
LruCache 的核心思想很好理解，就是要维护一个缓存对象列表，其中对象列表的排列方式是按照访问顺序实现的，即一直没访问的对象会放在队尾，即将被淘汰。而最近访问的对象会放在队头，最后被淘汰。
这个队列由 LinkedHashMap 来维护。LinkedHashMap 由数组+双向链表的数据结构实现，其中双向链表的结构可以实现访问顺序和插入顺序，使得 LinkedHashMap 中的`<key,value>`对按照一定顺序排列起来。

通过下面的构造函数来指定 LinkedHashMap 中双向链表的结构是访问顺序还是插入顺序。

```Java
public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```
accessOrder 设置为 true 则为访问顺序；为 false，则为插入顺序。

以具体例子解释，当设置为true时

```Java
public static final void main(String[] args) {
    LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(0, 0.75f, true);
    map.put(0, 0);
    map.put(1, 1);
    map.put(2, 2);
    map.put(3, 3);
    map.put(4, 4);
    map.put(5, 5);
    map.put(6, 6);
    map.get(1);
    map.get(2);
    for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```
输出结果为:
```Java
0:0
3:3
4:4
5:5
6:6
1:1
2:2
```
即最近访问的最后输出，正好满足 LRU 缓存算法的思想。可见 LruCache 的巧妙实现，就是利用了LinkedHashMap 的这种数据结构。

下面我们在 LruCache 源码中具体看看，怎样应用 LinkedHashMap 来实现缓存的添加，获得和删除

构造方法

````Java
public LruCache(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }
    this.maxSize = maxSize;
    this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
}
```
`put`方法

```Java
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }
    
    V previous;
    synchronized (this) {
        //插入的缓存对象值加1
        putCount++;
        //增加已有缓存的大小
        size += safeSizeOf(key, value);
        //向map中加入缓存对象
        previous = map.put(key, value);
        //如果已有缓存对象，则缓存大小恢复到之前
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }
    
    //entryRemoved()是个空方法，可以自行实现
    if (previous != null) {
        entryRemoved(false, key, previous, value);
    }
    
    //调整缓存大小(关键方法)
    trimToSize(maxSize);
    return previous;
}
```
可以看到，添加过缓存对象后会调用`trimToSize`方法，来判断缓存是否已满，如果满了就删除近期最少使用的对象

`trimToSize`方法

```Java
public void trimToSize(int maxSize) {
    //死循环
    while (true) {
        K key;
        V value;
        synchronized (this) {
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }
            
            //如果缓存大小size小于最大缓存，或者map为空，不需要再删除缓存对象，跳出循环
            if (size <= maxSize || map.isEmpty()) {
                break;
            }
            
            //迭代器获取第一个对象，即队尾的元素，近期最少访问的元素
            Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
            key = toEvict.getKey();
            value = toEvict.getValue();
            
            //删除该对象，并更新缓存大小
            map.remove(key);
            size -= safeSizeOf(key, value);
            evictionCount++;
        }
        entryRemoved(true, key, value, null);
    }
}
```
该方法不断地删除 LinkedHashMap 中队尾的元素，直到缓存大小小于最大值。

当调用 LruCache 的`get`方法获取集合中的缓存对象时，就代表访问了一次该元素，队列将会更新，保持其按照访问顺序的排序的规则。这个更新过程是在 LinkedHashMap 中的`get`方法中完成的。

先看 LruCache 的`get`方法

```Java
public final V get(K key) {
    //key为空抛出异常
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
    //获取对应的缓存对象
    //get()方法会实现将访问的元素更新到队列头部的功能
    mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;
            return mapValue;
        }
        missCount++;
    }
}
```


LinkedHashMap 的`get`方法如下：

```Java
public V get(Object key) {
    LinkedHashMapEntry<K,V> e = (LinkedHashMapEntry<K,V>)getEntry(key);
    if (e == null)
        return null;
    //实现排序的关键方法
    e.recordAccess(this);
    return e.value;
}
```

```Java
 void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    //判断是否是访问排序
    if (lm.accessOrder) {
        lm.modCount++;
        //删除此元素
        remove();
        //将此元素移动到队列的头部
        addBefore(lm.header);
    }
}
```



**由此可见 LruCache 中维护了一个集合 LinkedHashMap，该 LinkedHashMap 是以访问顺序排序的。当调用`put`方法时，就会在队列中添加元素，并调用`trimToSize`判断缓存是否已满，如果满了就用 LinkedHashMap 的迭代器删除队尾元素，即近期最少访问的元素。当调用`get(`方法访问缓存对象时，就会调用 LinkedHashMap 的`get`方法获得对应集合元素，同时更新该元素到队头。**

以上便是 LruCache 实现的原理，理解了 LinkedHashMap 的数据结构就能理解整个原理。

本文转自 [彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)