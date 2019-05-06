---
title: Android View 的工作原理
date: 2019-03-01 13:10:14
tags:
- Android 
- View
categories:
- Android
---

## 引言

在 Android 开发中，View 扮演了很重要的角色，是 Android 在视觉上的呈现。不满足于既有控件的开发者，会自定义 View 实现各种效果。但自定义复杂的 View 具有一定的难度，需要开发者掌握 View 的底层工作原理，比如 View 的测量、布局和绘制等流程。

<!--more-->

## 整体过程

### Window

首先简单认识 Window，其表示一个窗口，属于一个抽象类，具体的实现为 PhoneWindow。通过 WindowManager 向 WindowManagerService 发起请求，WMS 负责 Window 的具体生成。

实际上每一个 Window 对应一个 View 和 ViewRootImpl，  因此 Window 不是实际存在的，它通过 ViewRootImpl 以 View 的形式存在。

### ViewRoot 和 DecorView

ViewRoot 的具体实现就是上一节所提及的 ViewRootImpl，View 的三大流程（measure、layout、draw）都是通过它来实现的。

在 ActivityThread 中，当 Activity 对象被创建后，会将一个 DecorView 添加到 Window，然后创建 ViewRootImpl 对象，并将 DecorView 和 ViewRootImpl 建立关联，这也验证了上一节的观点。

```java
root = new ViewRootImpl(view.getContext(), display);
root.setView(view, vparams, panelParentView);
```

View 的绘制流程是从 ViewRoot 的 `performTraversals` 开始的，经过`measure`（测量 View 的宽高）、`layout`（确定 View 在父容器的位置）、`draw`（将 View 绘制在屏幕上）三个过程，呈现出一个 View。



![performTraversals](Android-View-的工作原理\performTraversals.png)



如图所示，`performTraversals`会依次调用`performMeasure`、`performLayout`、`performDraw`，然后这三个方法分别完成顶级 View 的三大流程。而后，`measure`又会调用`onMeasure`，在其中对所有子元素进行 measure 过程，此时 measure 流程就从父容器传递到子元素了，接着子元素重复上述过程。如此完成整个 View 树的遍历。

measure 过程决定了 View 的宽高，可以通过`getMeasuredWidth`和`getMeasuredHeight`获得 View 测量后的宽高，在几乎所有情况下都可以得到 View 最终的数值。

layout 过程决定了 View 的四个顶点的坐标和实际的 View 的宽高，可以通过`getTop`、`getBottom`、`getLeft`、`getRight`拿到四个顶点的位置，通过`getWidth`、`getHeight`获得 View 的最终宽高。只有 draw 过程完成之后才能呈现 View。

---

![整体布局.webp](Android-View-的工作原理\整体布局.webp.jpg)



由上图我们可以看出，一般 DecorView 会包含一个 LinearLayout，其中上面是标题栏，下面是内容栏。在创建 Activity 的时候需要`setContentView`而不是`setView`的原因便是如此。我们的布局加到了  id 为`android.R.id.content`的 FrameLayout 中。

```java
//得到 content
ViewGroup content = (ViewGroup) findViewById(android.R.id.content)；

//得到开发者设置的 View
View view = content.getChildAt(0);
```

## 具体流程

### MeasureSpec

“测量规格”，是 View 测量过程中非常重要的参数。measure 时系统会将 View 的 LayoutParams 根据父容器施加的规则转换成对应的 MeasureSpec，然后再根据 该 MeasureSpec 测量出 View 的宽高。

MeasureSpec 代表一个32位的 int 值，其中高2位代表 SpecMode（测量模式），低30位代表 SpecSize（某种测量模式下的规格大小）。将两个参数打包成一个 int 值的原因是避免过多的对象内存分配。

SpecMode 有三类

| SpecMode    | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| UNSPECIFIED | 父容器不限制 View，一般用于系统内部                          |
| EXACTLY     | 父容器已检测 VIew 的精确大小，对应参数为`match_parent`和具体的数值 |
| AT_MOST     | 父容器指定了 View 的最大值，具体大小由 View 决定，对应参数为`wrap_content` |

### MeasureSpec 和 LayoutParams 

对于 DecorView，其 MeasureSpec 由窗口尺寸和其自身 LayoutParams 决定，对于普通 View，其 MeasureSpec 由父容器的 MeasureSpec 和 自身的LayoutParams 共同决定。一旦 MeasureSpec 确定，onMeasure 中就可以确定 View 的测量宽高。



ViewRootImpl 的源码地址：[ViewRootImpl.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/ViewRootImpl.java)。查阅源码，得到 DecorView 的测量规则如下

- LayoutParams.MATCH_PARENT：精确模式，大小为窗口大小

- LayoutParams.WRAP_CONTENT：最大模式，大小不定，但不能超过窗口

- 固定大小：精确模式，大小为 LayoutParams 指定的大小



ViewGroup  的源码地址：[ViewGroup.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/ViewGroup.java)。查看 ViewGroup 的`measureChildWithMargins`方法

```java
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

可以看到在调用子元素的`measure`之前，会先得到子元素的 MeasureSpec，很显然其和父容器的 MeasureSpec 以及子元素自身的 LayoutParams 包括 margin、padding 参数有关。具体的逻辑在 ViewGroup 中的 `getChildMeasureSpec` 中实现。

`getChildMeasureSpec`方法清楚展示了 普通 View 的 MeasureSpec 的创建规则，下表是该方法的直观展示（表中 parentSize 指父容器中目前可使用的大小）

![MeasureSpec创建规则](Android-View-的工作原理\MeasureSpec创建规则.png)

由此可以看出

- 当 View 采用固定宽高时，View 的 MeasureSpec 与父容器无关，为精确模式、大小为 LayoutParams 设定的值
- 当 View 的宽高为`match_parent`时，如果父容器为精确模式/最大模式，则其也为精确模式/最大模式，且大小为父容器的剩余空间
- 当 View 的宽高为`wrap_content`时，View 的模式总是最大化模式，且大小不超过父容器的剩余空间
-  UNSPECIFIED 模式主要用于系统内部多次 Measure 的情形，一般情况下无需关注 

### measure 过程

#### View

对于 View，measure 完成其自身的测量过程；对于 ViewGroup，除了完成自己的测量过程，还会遍历调用所有子元素的`measure`方法，各子元素再递归执行这一过程。`measure` 是一个`final`方法（[View.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/View.java) 第 23267 行），这意味着子类不能重写该方法。在`measure`中会调用 View 的`onMeasure`，所以开发者只需要重写该方法即可。

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),
                         getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
}
```

`setMeasuredDimension`会设置 View 宽高的测量值，`getDefaultSize`返回 View 测量后的大小。

对于`getSuggestedMinimumWidth`，如果 View 没有设置背景，那么宽度为`mMinWidth`，其对应`android:minWidth`属性，如果不指定属性，则`mMinWidth`默认为0；如果 View 设置了背景，则 View 的宽度为`max(mMinWidth, mBackground.getMinimumWidth())`。那么`mBackground.getMinimumWidth()`所为何物？

观察 Drawable 的`getMinimumWidth`方法（[Drawable.java](https://android.googlesource.com/platform/frameworks/base/+/c80ad99a33ee49d0bac994c1749ff24d243c3862/graphics/java/android/graphics/drawable/Drawable.java) 第 798 行）

```java
public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}
```

可以发现该方法返回 Drawable 的原始宽度（如果存在，否则返回0 —— 比如 ShapeDrawable 就无原始宽高）。

直接继承 View 的自定义控件需要重写`onMeasure`方法并设置`wrap_content`时的自身大小，否则在布局中使用`wrap_content`就相当于使用`match_parent`。原因在于，由上面的表格我们知道，当使用`wrap_content`时，View 的 SpecMode 为 AT_MOST，此时宽高等于 SpecSize，SpecSize 此时又等于 parentSize，效果跟使用`match_parent`是一样的。

解决该问题的方法很简单，给 View 指定一个默认的内部宽高（mWidth 和 mHeight），并在`wrap_content`时设置此宽高即可

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(int widthMeasureSpec, int heightMeasureSpec);
    
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
    
    if (widthSpecMode == MeasureSpec.AT_MOST 
        	&& heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, mHeight);
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, heightSpecSize);
    } else if (heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(widthSpecSize, mHeight);
    }
}
```

#### ViewGroup

对于 ViewGroup 来说，其属于 View 的子类，但同时也是抽象的，它没有重写 View 的`onMeasure`，而是提供了`measureChildren`的方法用于测量子元素

```java
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
```

其中调用了`measureChild`

```java
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

显然该方法的思想是取出子元素的 LayoutParams 然后通过`getChildMeasureSpec`（如 MeasureSpec 小节的分析）创建子元素的 MeasureSpec，接着将 MeasureSpec 传递给 View 的`measure`方法进行测量。

ViewGroup 并没有测量的具体过程，而是交给其子类实现，比如 [LinearLayout](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/widget/LinearLayout.java)、[RelativeLayout](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/widget/RelativeLayout.java) 等。

#### 宽高的获取

一般情况下，measure 完成后即可通过`getMeasuredWidth/height`获得 View 的测量宽高，但是极端情况下系统可能需要多次 measure 才能确定最终的宽高，所以最好是在`onLayout`中获取宽高，而不是在`onMeasure`中。

考虑 View 外部，比如当我们在 Activity 的`onCreate`或者`onResume`中获取 View 的宽高时，会发现结果是不正确的，这是因为 View 的 measure 过程和 Activity 的生命周期方法不是同步的，因此无法保证在 Activity 执行`onCreate`、`onResume`时某个 View 已经测量完毕。有四种方法解决该问题：

- Activity/View.onWindowFocusChanged，此时 View 已经初始化完毕。注意该方法可能会被调用多次，比如每次 Activity 的窗口获得/失去焦点

  ```java
  public void onWindowFocusChanged(boolean hasFocus) {
      super.onWindowFocusChanged(hasFocus);
      if (hasFocus) {
          int width = view.getMeasuredWidth();
          int height = view.getMeasuredHeigth();
      }
  }
  ```

  

- view.post(Runnable runnable)，通过 post 将一个 Runnable 投递到消息队列尾部，当 Looper 调用此 Runnable 时，View 已经被初始化

  ```java
  view.post(() -> {
      int width = view.getMeasuredWidth();
      int height = view.getMeasuredHeigth();
  })
  ```

  

- ViewTreeObserver，该类拥有一系列回调方法，比如使用 OnGlobalLayoutListener 接口监听 View 树的状态，同样接口方法会被调用多次

  ```java
  ViewTreeObserver observer = view.getViewTreeObserver();
  observer.addOnGlobalLayoutListener(() -> {
      int width = view.getMeasuredWidth();
      int height = view.getMeasuredHeigth();
  })
  ```

  

- view.measure(int widthMeasureSpec, int heightMeasureSpec)，手动 measure，较为复杂，不再赘述。

### layout 过程

View / ViewGroup 使用`layout`过程确定自身位置，然后在`onLayout`中遍历所有的子元素并调用其`layout`方法，重复上述过程。

View 中的`layout`方法

```java
public void layout(int l, int t, int r, int b) {
        ......
        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
            ......
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }
    ......
}
```

首先通过`setFrame`设定 View 的四个顶点（mLeft、mTop、mBottom、mRight）的值，以此确定 View 在父容器中的位置。接着调用`onLayout`确定子元素的位置，和`onMeasure`类似，View / ViewGroup 都没有提供该方法的实现，而是交给具体的布局。

在 **ViewRoot 和 DecorView** 小节，提及了在 View 的 layout 之后通过`getWidth`、`getHeight`获得 View 的“最终宽高”，那么`getMeasuredWidth`和`getWidth`的区别到底是什么？

```java
    public final int getWidth() {
        return mRight - mLeft;
    }
    
    public final int getHeight() {
        return mBottom - mTop;
    }
```

从上面的代码可以看出，`getWidth`的返回值刚好就是 View 的测量宽度，也就是说，View 的测量宽高等于最终宽高，只不过测量宽高形成与 measure 过程，而最终宽高形成与 layout 过程 —— 赋值时机不同。但是如果重写 View 的`layout`方法，改变了`super`的参数值，比如

```java
public void layout(int l, int t, int r, int b) {
    super.layout(l, t, r + 100, b + 100);
}
```

就会导致 View 的最终宽高总是比测量宽高大 100px。

### draw 过程

将 View 绘制到屏幕上，具体步骤

- 绘制背景 background.draw(canvas)
- 绘制自身 onDraw
- 绘制子元素 dispatchDraw
- 绘制装饰 onDrawScrollBars



## 其他

Android 提供了一些 API 供开发调用，实现对 View 绘制过程的操纵

- `requestLayout`

  调用此方法会导致 View 树调用 layout 和 measure 过程，但不会触发 draw 流程

- `invalidate`

  请求重绘 View 树，即 draw 过程。在子线程中可以通过`postInvalidate`实现

当开发者调用 View 的`setVisibility`方法实现 VISIBLE / INVISIBLE -> GONE 时，相当于间接调用 `requestLayout` 和 `invalidate`。 

当开发者调用 View 的`setVisibility`方法实现  INVISIBLE -> VISIBLE 时，相当于间接调用 `invalidate`。 

---

*本文主要参考了《Android 开发艺术探索》——任玉刚 著*