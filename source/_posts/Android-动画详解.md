---
title: Android 动画详解
date: 2019-02-18 19:12:22
categories:
- Android
---

## 引言
Android 提供了很多丰富的 API 去实现动画效果，加上 SDK 25.3 时推出的 SpringAnimation，总共有三大类 Animation 供开发者使用。<!--more-->

## View Animation 
视图动画的作用对象是 View，可分为补间动画和帧动画。

### 补间动画
动画开始和结尾的中间过程都是假象，是渲染出来的表象，只是显示的位置变动，View的实际位置未改变，表现为View移动到其他地方，点击事件仍在原处才能响应。利用补间动画，同一个图形在界面上可以进行透明度（AlphaAnimation）、缩放（ScaleAnimation）、旋转（RotateAnimation）、平移（TranslateAnimation）的变化。

#### XML 方式
创建一个 set.xml 文件，通过动画集合标签`<set>`将四种效果结合起来
```XML
<?xml version="1.0" encoding="utf-8"?>

<--动画集合
interpolator 表示所采用的的差值器，其影响动画的速度，可以不指定
shareInterpolator 表示集合中的动画是否和集合共享同一个差值器-->
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fillAfter="true"
    android:shareInterpolator="true"
    android:repeatMode="reverse">
    
    <!-- 缩放动画
    fromXScale 表示水平方向缩放的起始值
    toXScale 表示水平方向的结束值
    fillAfter 表示动画显示结束保持最后一帧-->
    <scale  
        android:duration="1000"
        android:fillAfter="true"
        android:fromXScale="0.5"
        android:fromYScale="0.5"
        android:toXScale="1"
        android:toYScale="1"
        android:repeatCount="infinite"/><!--次数 ,infinite 为无线循环播放-->
        
    <!-- 透明度动画
        fromAlpha 表示起始透明度
        toAlpha 表示结束透明度-->
    <alpha
        android:duration="2000"
        android:fillAfter="true"
        android:fromAlpha="0.7"
        android:toAlpha="1"/>
        
    <!--旋转动画
    fromDegrees 表示起始角度
    toDegrees 表示结束角度-->
    <rotate
        android:fromDegrees="0"
        android:toDegrees="90"
        android:fillAfter="true"
        android:duration="1000"/>
    
    <!--平移动画-->
    <translate
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="100%"
        android:toYDelta="-100%"
        android:fillAfter="true"
        android:duration="2000"
        />
</set>
```
使用以上动画的方式如下
```Java
Animation anim = AnimationUtils.loadAnimation(context, R.anim.my_animation);
view.startAnimation(anim);
```

#### Java 代码方式
使用 Java 代码实现动画的方式如下
```Java
public void startAnimationSet() {
    //创建动画，参数表示他的子动画是否共用一个插值器
    AnimationSet animationSet = new AnimationSet(true);
    //添加动画
    animationSet.addAnimation(new AlphaAnimation(1.0f, 0.0f));
    //设置插值器
    animationSet.setInterpolator(new LinearInterpolator());
    //设置动画持续时长
    animationSet.setDuration(3000);
    //设置动画结束之后是否保持动画的目标状态
    animationSet.setFillAfter(true);
    //设置动画结束之后是否保持动画开始时的状态
    animationSet.setFillBefore(false);
    //设置重复模式
    animationSet.setRepeatMode(AnimationSet.REVERSE);
    //设置重复次数
    animationSet.setRepeatCount(AnimationSet.INFINITE);
    //设置动画延时时间
    animationSet.setStartOffset(2000);
    //取消动画
    animationSet.cancel();
    //释放资源
    animationSet.reset();
    //开始动画
    view.startAnimation(animationSet);
}
```

#### 自定义
除了系统自带的四种补间动画，我们还可以自定义 View 动画。派生一种动画只需要继承抽象类 Animation，然后重写它的`initialize`和`applyTransformation`方法即可，在前一个方法中做初始化工作，后一个方法中进行相应的矩阵变换，本文不再详细介绍。 

#### 特殊使用
View 动画还可以用在控制 ViewGroup 中子元素的出场效果、实现不同 Activity 的切换效果等场景中。

### 帧动画
帧动画是顺序播放一组预先定义好的图片，类似于电影播放。不同于补间动画，系统提供了另外一个类 AnimationDrawable 来使用帧动画。

首先需要定义一个 XML 文件`frame_animation.xml`
```XML
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item
        android:drawable="@mipmap/image1"
        android:duration="100" />
    <item
        android:drawable="@mipmap/image2"
        android:duration="100" />
    <item
        android:drawable="@mipmap/iamge2"
        android:duration="100" />
</animation-list>
```
然后将上述的 Drawable 作为 View 的背景并通过 Drawable 来播放动画即可
```Java
view.setBackgroundResource(R.drawable.frame_animation)；
AnimationDrawable drawable = (AnimationDrawable) view.getBackground();
drawable.start();
```

## Property Animation
属性动画在 API 11 引入，可以看作是增强版的补间动画，与补间动画的不同之处体现在：
- 补间动画只能定义两个关键帧在透明、旋转、位移和倾斜这四个属性的变换，但是属性动画可以定义任何属性的变化。
- 补间动画只能对 UI 组件执行动画，但属性动画可以对任何对象执行动画。

与补间动画类似，属性动画也需要定义几个方面的属性：
- 动画持续时间。默认为 300ms，可以通过 android:duration 属性指定。
- 动画插值方式。通过 android:interploator 指定。
- 动画重复次数。通过 android:repeatCount 指定。
- 重复行为。通过 android:repeatMode 指定。
- 动画集。在属性资源文件中通过 `<set>` 来组合。
- 帧刷新率。指定多长时间播放一帧。默认为 10 ms。

### API
- ValueAnimator：属性动画用到的主要的时间引擎，负责计算各个帧的属性值。
- ObjectAnimator： ValueAnimator 的子类，对指定对象的属性执行动画。
- AnimatorSet：Animator 的子类，用于组合多个 Animator。

属性动画还提供了一个 Evaluator ，用来控制如何计算属性值。
- IntEvaluator：计算 int 类型属性值的计算器。
- FloatEvaluator：用于计算 float 类型属性值的计算器。
- ArgbEvaluator：用于计算十六进制形式表示的颜色值的计算器。
- TypeEvaluator：可以自定义计算器。

### ValueAniamtor
ValueAnimator 类中有3个重要方法：
> ValueAnimator.ofInt(int values)<br>ValueAnimator.ofFloat(float values)<br>ValueAnimator.ofObject(int values)

#### ofInt
将初始值以整型数值的形式过渡到结束值,即估值器是整型估值器 —— IntEvaluator

下面的代码将实现按钮的宽度从 150px 放大到 500px
```Java
    Button mButton = (Button) findViewById(R.id.Button);
    // 设置属性数值的初始值 & 结束值
    // ValueAnimator.ofInt()内置了整型估值器，默认设置了如何从初始值150 过渡到 结束值500
    ValueAnimator valueAnimator = ValueAnimator.ofInt(mButton.getLayoutParams().width, 500);
  
    // 设置动画的播放各种属性
    valueAnimator.setDuration(2000);

    // 将属性数值手动赋值给对象的属性，此处是将值赋给按钮的宽度
    // 设置更新监听器，数值每次变化更新都会调用该方法
    valueAnimator.addUpdateListener(new AnimatorUpdateListener() {
    
        @Override
        public void onAnimationUpdate(ValueAnimator animator) {
            // 获得每次变化后的属性值
            int currentValue = (Integer) animator.getAnimatedValue();
            // 将值手动赋值给对象的属性，实现按钮宽度属性的动态变化
            mButton.getLayoutParams().width = currentValue;
            
            // 刷新视图，即重新绘制
            mButton.requestLayout();
        }
    });
    valueAnimator.start();  // 启动动画
}
```

#### ofFloat
其与 ofInt 的区别仅在于采用了浮点估值器（FloatEvaluator）
```Java
public class FloatEvaluator implements TypeEvaluator {  

    /**
     * 重写evaluate()
     *
     * @param fraction 动画完成度（根据它来计算当前动画的值）
     * @param startValue 动画的初始值
     * @param endValue 动画的结束值
     * @return 
     */
    public Object evaluate(float fraction, Object startValue, Object endValue) {  

        float startFloat = ((Number) startValue).floatValue();  
        
        // 初始值过渡到结束值的算法
        // 1. 用结束值减去初始值，算出它们之间的差值
        // 2. 用上述差值乘以 fraction 系数
        // 3. 加上初始值，得到当前动画的值
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);  
    }  
}  
```

#### ofObject
对于 ValueAnimator.ofInt 和 ValueAnimator.ofFloat 来说，由于使用了系统内置的估值器 —— FloatEvaluator 和 IntEvaluator，所以已经默认实现了从初始值到结束值的逻辑。

但对于 ValueAnimator.ofObject，并没有系统默认实现，因为对象的动画操作复杂多样，系统无法知道如何从初始对象过度到结束对象，因此需要自定义估值器（TypeEvaluator）来告知系统具体的逻辑。

自定义一个估值器 PointEvaluator，实现一个圆从一个点移动到另外一个点。默认已经有了一个点坐标类：Point，其具有 x 和 y 两个浮点属性。

```Java
public class PointEvaluator implements TypeEvaluator {

    @Override
    public Object evaluate(float fraction, Object startValue, Object endValue) {

        // 将动画初始值startValue 和 动画结束值endValue 类型转换成Point对象
        Point startPoint = (Point) startValue;
        Point endPoint = (Point) endValue;

        // 根据fraction来计算当前动画的x和y的值
        float x = startPoint.getX() + fraction * (endPoint.getX() - startPoint.getX());
        float y = startPoint.getY() + fraction * (endPoint.getY() - startPoint.getY());
        
        // 将计算后的坐标封装到一个新的Point对象中并返回
        Point point = new Point(x, y);
        return point;
    }
}
```
将属性动画作用到自定义View当中
```Java
public class MyView extends View {

    public static final float RADIUS = 70f;// 圆的半径
    private Point currentPoint;// 当前点坐标
    private Paint mPaint;// 绘图画笔
    
    // 构造方法，初始化画笔
    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
    }

    // 实现绘制逻辑
    // 先在初始点画圆，监听当前坐标值的变化，每次变化都调用onDraw()重新绘制圆，实现圆的平移动画效果
    @Override
    protected void onDraw(Canvas canvas) {
        // 如果当前点坐标为空(即第一次)
        if (currentPoint == null) {
            currentPoint = new Point(RADIUS, RADIUS);  // 创建一个点对象(坐标是(70,70))

            float x = currentPoint.getX();
            float y = currentPoint.getY();
            canvas.drawCircle(x, y, RADIUS, mPaint);

            // 将属性动画作用到View中
            Point startPoint = new Point(RADIUS, RADIUS);// 初始点为圆心(70,70)
            Point endPoint = new Point(700, 1000);// 结束点为(700,1000)

            ValueAnimator anim = ValueAnimator.ofObject(new PointEvaluator(), startPoint, endPoint);
            anim.setDuration(5000);
    
            anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    currentPoint = (Point) animation.getAnimatedValue();
                    // 每次赋值后就重新绘制，从而实现动画效果
                    // 调用invalidate()后,就会刷新View,即才能看到重新绘制的界面,即onDraw()会被重新调用一次
                    invalidate();  
                }
            });
            anim.start();
        } else {
            // 如果坐标值不为0,则画圆
            float x = currentPoint.getX();
            float y = currentPoint.getY();
            canvas.drawCircle(x, y, RADIUS, mPaint);
        }
    }
}
```

### ObjectAnimator
继承自 ValueAnimator 类，即底层的动画实现机制基于 ValueAnimator。
ObjectAnimator 与 ValueAnimator类的区别在于
- ValueAnimator 类是先改变值，然后**手动赋值**给对象的属性从而实现动画，属于**间接**对对象属性进行操作
- ObjectAnimator 类是先改变值，然后**自动赋值**给对象的属性从而实现动画，属于**直接**对对象属性进行操作

#### 具体使用
对于
> ObjectAnimator animator = ObjectAnimator.ofFloat(Object object, String property, float ....values); 

其中 values 参数不定，表示动画初始值和结束值，如果是两个参数a,b，则动画效果是从属性的a值到b值；如果是三个参数a,b,c，则动画效果是从属性的a值到b值再到c值

对 Button 进行变换
```Java

ObjectAnimator animator = ObjectAnimator.ofFloat(mButton, "alpha", 1f, 0f, 1f);  // 效果:常规->全透明->常规
ObjectAnimator animator = ObjectAnimator.ofFloat(mButton, "rotation", 0f, 360f);
......
animator.setDuration(5000);
animator.start();
```

#### 自定义
在上面的例子中，我们给`ObjectAnimator.ofFloat`的第二个参数`String property`传入`alpha`、`rotation`、`translationX` 和`scaleY`等值，实际上，我们可以传任意属性值，因为 ObjectAnimator 类实现动画效果的本质是：不断控制值的变化，再不断自动赋给对象的属性，赋值的过程是通过调用对象的`get/set`方法进行的。

所以自定义属性就可以通过为对象设置需要操作属性的`set/get`方法，再实现 TypeEvaluator 定义属性变化的逻辑完成。

还是对一个球做变换
```Java
public class MyView2 extends View {
    public static final float RADIUS = 100f;
    private Paint mPaint;

    private String color;  // 设置背景颜色属性

    // 设置背景颜色的get() & set()方法
    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
        mPaint.setColor(Color.parseColor(color));  // 将画笔的颜色设置成方法参数传入的颜色
        invalidate();
    }

    public MyView2(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawCircle(500, 500, RADIUS, mPaint);
    }
}
```
实现自定义估值器，完成颜色过渡的逻辑
```Java
public class ColorEvaluator implements TypeEvaluator {

    private int mCurrentRed;

    private int mCurrentGreen ;

    private int mCurrentBlue ;

    // 重写evaluate，写入对象动画过渡的逻辑，此处为颜色过渡
    @Override
    public Object evaluate(float fraction, Object startValue, Object endValue) {

        String startColor = (String) startValue;
        String endColor = (String) endValue;

        // 通过字符串截取的方式将初始化颜色分为RGB三个部分，并将RGB的值转换成十进制数字
        // 那么每个颜色的取值范围是0-255
        int startRed = Integer.parseInt(startColor.substring(1, 3), 16);
        int startGreen = Integer.parseInt(startColor.substring(3, 5), 16);
        int startBlue = Integer.parseInt(startColor.substring(5, 7), 16);

        int endRed = Integer.parseInt(endColor.substring(1, 3), 16);
        int endGreen = Integer.parseInt(endColor.substring(3, 5), 16);
        int endBlue = Integer.parseInt(endColor.substring(5, 7), 16);

        // 将初始化颜色的值定义为当前需要操作的颜色值
        mCurrentRed = startRed;
        mCurrentGreen = startGreen;
        mCurrentBlue = startBlue;


        // 计算初始颜色和结束颜色之间的差值
        // 该差值决定着颜色变化的快慢，如果初始颜色值和结束颜色值相近，变化会比较缓慢
        int redDiff = Math.abs(startRed - endRed);
        int greenDiff = Math.abs(startGreen - endGreen);
        int blueDiff = Math.abs(startBlue - endBlue);
        int colorDiff = redDiff + greenDiff + blueDiff;
        
        if (mCurrentRed != endRed) {
            // getCurrentColor()决定如何根据差值来决定颜色变化的快慢 ->>关注1
            mCurrentRed = getCurrentColor(startRed, endRed, colorDiff, 0, fraction);
        
        } else if (mCurrentGreen != endGreen) {
            mCurrentGreen = getCurrentColor(startGreen, endGreen, colorDiff, redDiff, fraction);
            
        } else if (mCurrentBlue != endBlue) {
            mCurrentBlue = getCurrentColor(startBlue, endBlue, colorDiff, redDiff + greenDiff, fraction);
        }
        // 将计算出的当前颜色的值组装返回
        String currentColor = "#" + getHexString(mCurrentRed)
                + getHexString(mCurrentGreen) + getHexString(mCurrentBlue);
        return currentColor;
    }



    // 根据fraction值来计算当前的颜色。
    private int getCurrentColor(int startColor, int endColor, int colorDiff,
                                int offset, float fraction) {
        int currentColor;
        if (startColor > endColor) {
            currentColor = (int) (startColor - (fraction * colorDiff - offset));
            if (currentColor < endColor) {
                currentColor = endColor;
            }
        } else {
            currentColor = (int) (startColor + (fraction * colorDiff - offset));
            if (currentColor > endColor) {
                currentColor = endColor;
            }
        }
        return currentColor;
    }

    // 将10进制颜色值转换成16进制。
    private String getHexString(int value) {
        String hexString = Integer.toHexString(value);
        if (hexString.length() == 1) {
            hexString = "0" + hexString;
        }
        return hexString;
    }
}
```
具体调用
```Java
ObjectAnimator anim = ObjectAnimator.ofObject(myView2, "color", new ColorEvaluator(), "#0000FF", "#FF0000");
anim.setDuration(2000);
anim.start();
```

此时还有一个问题需要我们解决：如果需要对 view 控件（比如 button）的宽高做变换，但是由于因为 View 中`setWidth`并不是设置 View 的宽度，而是设置控件的最大和最小宽度，所以通过`get/set`无法改变控件的宽度，也就无法实现动画效果。

解决方案是使用装饰器模式，包装原始动画对象，间接给对象加上该属性的`get/set`方法。
```Java
ButtonWrapper wrapper = new ViewWrapper(button);
ObjectAnimator.ofInt(wrapper, "width", 500)
        .setDuration(3000)
        .start();

private static class ViewWrapper {
    private View mTarget;

    public ViewWrapper(View target) {
        mTarget = target;
    }
    
    // 为宽度设置get/set
    public int getWidth() {
            return mTarget.getLayoutParams().width;
    }
    
    public void setWidth(int width) {
        mTarget.getLayoutParams().width = width;
        mTarget.requestLayout();
    }
}
```

### AnimatorSet
最后介绍组合动画类，仅展示用法
```Java

ObjectAnimator translation = ObjectAnimator.ofFloat(mButton, "translationX", curTranslationX, 300,curTranslationX);  
ObjectAnimator rotate = ObjectAnimator.ofFloat(mButton, "rotation", 0f, 360f);  
ObjectAnimator alpha = ObjectAnimator.ofFloat(mButton, "alpha", 1f, 0f, 1f);  

AnimatorSet animSet = new AnimatorSet();  

animSet.play(translation).with(rotate).before(alpha);  
animSet.setDuration(5000);  
animSet.start();  
```
## Spring Animation
SpringAnimation，弹簧动画，位于`android.support.animation`包中，属性动画位于`android.animation.Animator`包中，其实通过 [BounceInterpolator](https://developer.android.com/reference/android/view/animation/BounceInterpolator.html) 或者 [OvershootInterpolator](https://developer.android.com/reference/android/view/animation/OvershootInterpolator.html) 作为插值器，同样可以实现弹性动画效果，引入 SpringAnimation 是因为它使用更简单，而且上两个差值器实现的轨迹并不符合物理学上的弹跳效果。

使用之前需要导入`com.android.support:support-dynamic-animation`包

### API
```Java
public SpringAnimation(View v, ViewProperty property)
public SpringAnimation(View v, ViewProperty property, float finalPosition)
```
参数分别是操作对应的View，对应的变化属性及最终的位置。

ViewProperty 包括(Z轴支持需要API >= 21)：
>TRANSLATION_X
>TRANSLATION_Y
>TRANSLATION_Z
>SCALE_X
>SCALE_Y
>ROTATION
>ROTATION_X
>ROTATION_Y
>X
>Y
>Z
>ALPHA
>SCROLL_X
>SCROLL_Y

在 SpringAnimation 中有一个 SpringForce 对象，负责对应的变量设置及位置计算。其中包括两个个关键变量
- Stiffness 刚度(劲度/弹性)，刚度越大，形变产生的里也就越大，体现在效果上就是运动越快
- DampingRatio 阻尼系数，系数越大，动画停止的越快。从理论上讲分为三种情况 Overdamped过阻尼（ζ > 1）、Critically damped临界阻尼(ζ = 1)、Underdamped欠阻尼状态(0 < ζ <1)。

### 简单使用
```Java
SpringAnimation btnAnim = new SpringAnimation(mButton, SpringAnimation.TRANSLATION_Y, 0);

btnAnim.getSpring().setStiffness(SpringForce.STIFFNESS_VERY_LOW);
btnAnim.getSpring().setDampingRatio(SpringForce.DAMPING_RATIO_LOW_BOUNCY);

btnAnim.setStartVelocity(10000);  //开始速度，单位是px/second. 正数是弹簧收缩的方向，负数相反
btnAnim.start();
```