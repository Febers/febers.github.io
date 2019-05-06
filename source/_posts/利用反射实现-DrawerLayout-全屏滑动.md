---
title: 利用反射实现 DrawerLayout 全屏滑动
date: 2019-05-06 12:58:30
tags:
- Android
- 反射
categories:
- Android
---

## 引言

在一个项目中需要用到 DrawerLayout，但是其默认实现为边缘滑动打开侧滑界面，只能指定左边缘或者右边缘。想要实现全屏滑动，思路是通过反射的方式修改 DrawerLayout 的相应属性，涉及到枯燥的源码阅读。在完成全屏滑动之后，又发现其默认实现了长按弹出侧滑界面，在全屏滑动下，用户长按任何地方都会跳出侧滑菜单，而且还会出现留白问题。研究半天，还是利用反射的思路一并解决，特此记录。<!--more-->

## DrawerLayout 侧滑

在 DrawerLayout 中定义了两个变量，分别对应 Gravity 为 Left 和 Right 的滑动情景，两者并无实质分别，本文只分析 Left 的情况。此外，DrawerLayout 包含三种状态，STATE_IDLE（已打开或已关闭），STATE_DRAGGING（正在拖动），STATE_SETTLING（执行打开或关闭的动画过程中）。

```java
private final ViewDragHelper mLeftDragger;
private final ViewDragHelper mRightDragger;
```

 构造函数对一些变量做了初始化

```java
mLeftCallback = new ViewDragCallback(Gravity.LEFT);

mLeftDragger = ViewDragHelper.create(this, TOUCH_SLOP_SENSITIVITY, mLeftCallback);
mLeftDragger.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
mLeftDragger.setMinVelocity(minVel);
mLeftCallback.setDragger(mLeftDragger);
```

ViewDraghelper 是官方提供的专门为自定义 ViewGroup 处理拖拽的手势类。此处用到的构造方法为

```java
public static ViewDragHelper create(@NonNull ViewGroup forParent, float sensitivity,
            @NonNull Callback cb)
```

DrawerLayout 中侧滑打开界面正是通过 ViewDragHelper 实现的，查看 DrawerLayout 的`onTouchEvent`方法

```java
@Override
public boolean onTouchEvent(MotionEvent ev) {
	mLeftDragger.processTouchEvent(ev);
	mRightDragger.processTouchEvent(ev);

	final int action = ev.getAction();
	boolean wantTouchEvents = true;

	switch (action & MotionEvent.ACTION_MASK) {
		......
	}

    return wantTouchEvents;
}
```

其明显调用了 ViewDragHelper 的`processTouchEvent`方法处理 Touch 事件

```java
public void processTouchEvent(MotionEvent ev) {
	......
    switch (action) {
        case MotionEvent.ACTION_DOWN: {
            final float x = ev.getX();
            final float y = ev.getY();
            final int pointerId = ev.getPointerId(0);
            //找到当前触摸点的最顶层的子View,作为需要操作的View
            final View toCapture = findTopChildUnder((int) x, (int) y);
            //保存当前Touch点发生的初始状态
            saveInitialMotion(x, y, pointerId);
            //这里是点在一个正在滑动的侧滑栏上，使侧滑栏的状态由正在滑动状态变为正在拖动状态
            tryCaptureViewForDrag(toCapture, pointerId);
            //处理侧滑栏的触摸触发区域是否触摸，如果触摸则通知回调，在DrawerLayout中处理，执行一个侧滑微弹的操作，也就是稍微弹出一点，表示触发了侧滑操作
            final int edgesTouched = mInitialEdgesTouched[pointerId];
            if ((edgesTouched & mTrackingEdges) != 0) {
                mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
            }
            break;
        }
    ......
}

```

重点在`mInitialEdgesTouched[pointerId]`，其为一个保存边缘滑动值的 int 数组。在`saveInitialMotion`方法中发现其赋值过程

```java
private void saveInitialMotion(float x, float y, int pointerId) {
	ensureMotionHistorySizeForId(pointerId);
	mInitialMotionX[pointerId] = mLastMotionX[pointerId] = x;
	mInitialMotionY[pointerId] = mLastMotionY[pointerId] = y;
	mInitialEdgesTouched[pointerId] = getEdgesTouched((int) x, (int) y);
	mPointersDown |= 1 << pointerId;
}
```

原来是调用了`getEdgesTouched`方法

```java
private int getEdgesTouched(int x, int y) {
	int result = 0;

	if (x < mParentView.getLeft() + mEdgeSize) result |= EDGE_LEFT;
	if (y < mParentView.getTop() + mEdgeSize) result |= EDGE_TOP;
	if (x > mParentView.getRight() - mEdgeSize) result |= EDGE_RIGHT;
	if (y > mParentView.getBottom() - mEdgeSize) result |= EDGE_BOTTOM;

	return result;
}
```

可以看到，该方法将判断`x < mParentView.get*() + mEdgeSize`，然后将对应的 result 返回。`mEdgeSize`即为边缘滑动的临界值，其初始化值为

```java
final float density = context.getResources().getDisplayMetrics().density;
mEdgeSize = (int) (EDGE_SIZE * density + 0.5f);
```

因此，要让 DrawerLayout 支持全屏滑动打开侧滑菜单而不是边缘滑动，重点便是要修改该值，将其设为屏幕宽度。

具体的反射代码（kotlin）

```kotlin
//获取 ViewDragHelper，更改 edgeSizeField
val leftDraggerField = drawerLayout.javaClass.getDeclaredField("mLeftDragger")
leftDraggerField.isAccessible = true
val leftDragger = leftDraggerField.get(drawerLayout) as ViewDragHelper

val edgeSizeField = leftDragger.javaClass.getDeclaredField("mEdgeSize")
edgeSizeField.isAccessible = true
val edgeSize = edgeSizeField.getInt(leftDragger)

val displaySize = Point()
activity.windowManager.defaultDisplay.getSize(displaySize)
edgeSizeField.setInt(leftDragger, displaySize.x)
```



## DrawerLayout 长按弹出

在 DrawerLayout 中，用户在非侧滑界面的 mEdgeSize 范围内长按，侧滑界面将弹出。当我们修改 mEdgeSize 为屏幕宽度之后，用户所有的长按动作都将触发原来的弹出逻辑，而且触发范围为屏幕宽度，侧滑菜单将过度右移，造成左侧边缘有空白。

原来是 DrawerLayout 的私有内部类 ViewDragCallback 重写了`onEdgeTouched`方法

```java
private class ViewDragCallback extends ViewDragHelper.Callback
```

```java
@Override
public void onEdgeTouched(int edgeFlags, int pointerId) {
	postDelayed(mPeekRunnable, PEEK_DELAY);
}
```

该方法会执行一个 mPeekRunnable，其为内部类的私有 Runnable 类型的属性，其`run`方法执行了`peekDrawer`方法

```java
void peekDrawer() {
	final View toCapture;
	final int childLeft;
	final int peekDistance = mDragger.getEdgeSize();
	final boolean leftEdge = mAbsGravity == Gravity.LEFT;
	if (leftEdge) {
		toCapture = findDrawerWithGravity(Gravity.LEFT);
		childLeft = (toCapture != null ? -toCapture.getWidth() : 0) + peekDistance;
	} else {
		toCapture = findDrawerWithGravity(Gravity.RIGHT);
		childLeft = getWidth() - peekDistance;
	}
	// Only peek if it would mean making the drawer more visible and the drawer isn't locked
	if (toCapture != null && ((leftEdge && toCapture.getLeft() < childLeft)
		|| (!leftEdge && toCapture.getLeft() > childLeft))
		&& getDrawerLockMode(toCapture) == LOCK_MODE_UNLOCKED) {
		final LayoutParams lp = (LayoutParams) toCapture.getLayoutParams();
		mDragger.smoothSlideViewTo(toCapture, childLeft, toCapture.getTop());
		lp.isPeeking = true;
		invalidate();

		closeOtherDrawer();
		cancelChildViewTouch();
	}
}
```

注意`mDragger.smoothSlideViewTo(toCapture, childLeft, toCapture.getTop())`就是长按屏幕时，侧滑菜单会自动滑出来的原因。

解决这个问题着实费了一番脑筋，因为 ViewDragCallback 为私有内部类，外部无法直接得到其引用。幸好观察之后发现其实现了 ViewDragHelper.Callback 接口，从而让我们可以利用多态的方式，获取其反射实例

```kotlin
//获取 Layout 的 ViewDragCallBack 实例“mLeftCallback”
//更改其属性 mPeekRunnable
val leftCallbackField = drawerLayout.javaClass.getDeclaredField("mLeftCallback")
leftCallbackField.isAccessible = true

//因为无法直接访问私有内部类，所以该私有内部类实现的接口非常重要，通过多态的方式获取实例
val leftCallback = leftCallbackField.get(drawerLayout) as ViewDragHelper.Callback

val peekRunnableField = leftCallback.javaClass.getDeclaredField("mPeekRunnable")
peekRunnableField.isAccessible = true
val nullRunnable = Runnable {  }
peekRunnableField.set(leftCallback, nullRunnable)
```

完美解决问题！

最后便是构建一个工具类

```kotlin
object DrawerLayoutHelper {

    /**
     * 通过反射的方式将 DrawerLayout 的侧滑范围设为全屏
     * 该方法存在一个问题，在侧滑范围内长按，也会划出菜单
     * 通过查看 DrawerLayout 的源码分析，其内部类 ViewDragCallback
     * 重写了 onEdgeTouched 方法，然后调用一个 Runnable 属性的变量 “mPeekRunnable”
     * 该变量调用了 peekDraw 方法，实现了长按划出侧滑菜单的功能
     * 同样使用反射将该 Runnable 更改为空实现
     *
     * @param activity
     * @param drawerLayout
     * @param displayWidthPercentage
     */
    fun setDrawerLeftEdgeSize(activity: Activity?,
                              drawerLayout: DrawerLayout?,
                              displayWidthPercentage: Float) {
        if (activity == null || drawerLayout == null) return
        try {
            //获取 ViewDragHelper，更改其 edgeSizeField 为 displayWidthPercentage*屏幕大小
            val leftDraggerField = drawerLayout.javaClass.getDeclaredField("mLeftDragger")
            leftDraggerField.isAccessible = true
            val leftDragger = leftDraggerField.get(drawerLayout) as ViewDragHelper

            val edgeSizeField = leftDragger.javaClass.getDeclaredField("mEdgeSize")
            edgeSizeField.isAccessible = true
            val edgeSize = edgeSizeField.getInt(leftDragger)

            val displaySize = Point()
            activity.windowManager.defaultDisplay.getSize(displaySize)
            edgeSizeField.setInt(leftDragger, Math.max(edgeSize, (displaySize.x * displayWidthPercentage).toInt()))

            //获取 Layout 的 ViewDragCallBack 实例“mLeftCallback”
            //更改其属性 mPeekRunnable
            val leftCallbackField = drawerLayout.javaClass.getDeclaredField("mLeftCallback")
            leftCallbackField.isAccessible = true

            //因为无法直接访问私有内部类，所以该私有内部类实现的接口非常重要，通过多态的方式获取实例
            val leftCallback = leftCallbackField.get(drawerLayout) as ViewDragHelper.Callback

            val peekRunnableField = leftCallback.javaClass.getDeclaredField("mPeekRunnable")
            peekRunnableField.isAccessible = true
            val nullRunnable = Runnable {  }
            peekRunnableField.set(leftCallback, nullRunnable)

        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    fun setDrawerLeftEdgeFullScreen(activity: Activity?, drawerLayout: DrawerLayout?) {
        setDrawerLeftEdgeSize(activity, drawerLayout, 1.0f)
    }
}
```

