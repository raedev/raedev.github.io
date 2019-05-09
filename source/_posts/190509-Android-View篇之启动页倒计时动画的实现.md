---
title: Android-View篇之启动页倒计时动画的实现
date: 2019-05-09 10:22:54
tags: View
---

Hello，小伙伴们大家好，今天来实现一个很简单的倒计时动画，仿酷狗音乐的启动页倒计时效果，也是大多数APP在用的一个动画，来看看效果图：

![倒计时动画](https://upload-images.jianshu.io/upload_images/2706530-3a013b585cff7f08.gif?imageMogr2/auto-orient/strip)

<!-- more -->

>  实现思路

看看是不是很简单，画个圈圈动起来，整体的思路就是用一个平滑的帧动画来画圆弧就行了。

>  这篇文章学到什么？

-  了解属性动画`ValueAnimator`的用法
-  了解动画属性插值`Interpolator`，让动画过度得更自然
-  如何画圆弧

> 开始准备

新建一个类继承`TextView`，因为中间还有`跳过`的文本，所以选择用`TextView`来画个动起来的背景图。

```java
/**
 * 倒计时文本
 * Created by ChenRui on 2017/10/31 0031 23:01.
 */
public class CountDownTextView extends RaeTextView {
    // 倒计时动画时间
    private int duration = 5000;
    // 动画扫过的角度
    private int mSweepAngle = 360;
    // 属性动画
    private ValueAnimator animator;
    // 矩形用来保存位置大小信息
    private final RectF mRect = new RectF();
    // 圆弧的画笔
    private Paint mBackgroundPaint;

    public CountDownTextView(Context context) {
        super(context);
    }

    public CountDownTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CountDownTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    void init() {
        super.init();
        // 设置画笔平滑
        mBackgroundPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        // 设置画笔颜色
        mBackgroundPaint.setColor(Color.WHITE);
        // 设置画笔边框宽度
        mBackgroundPaint.setStrokeWidth(dp2px(2));
        // 设置画笔样式为边框类型
        mBackgroundPaint.setStyle(Paint.Style.STROKE);
    }
```  

> 开始动画

原理： 利用圆的360度角来做属性动画，让它平滑的分配做每帧动画的角度值，然后调用`invalidate() `来重绘自己本身，从而进入到本身的`onDraw()`方法来画图。

```java
  /**
     * 开始倒计时
     */
    public void start() {
        // 在动画中
        if (mSweepAngle != 360) return;
        //  初始化属性动画
        animator = ValueAnimator.ofInt(mSweepAngle).setDuration(duration);
        // 设置插值
        animator.setInterpolator(new LinearInterpolator());
        // 设置动画监听
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                // 获取属性动画返回的动画值
                mSweepAngle = (int) animation.getAnimatedValue();
                // 重绘自己
                invalidate();
            }
        });
        // 开始动画
        animator.start();
    }
```

> 画圆弧

画圆弧比较简单， 从效果图来看，有的同学可能刚开始以为要画两个圆，一个背景的内圆和一个白色边框的大圆，其实这里可以利用画笔设置画笔样式`paint.setStyle()`和宽度大小`paint.setStrokeWidth()`的特性来实现。代码很简单，开始的角度选择-90，从头顶开始画。这样实现的是一个顺时针的倒计时效果。如果你想实现酷狗的逆时针效果，就控制`mSweepAngle `的值用`mSweepAngle  = 360 - mSweepAngle `开始就可以了。

```java
 @Override
    protected void onDraw(Canvas canvas) {
        int padding = dp2px(4);
        mRect.top = padding;
        mRect.left = padding;
        mRect.right = getWidth() - padding;
        mRect.bottom = getHeight() - padding;

        // 画倒计时线内圆
        canvas.drawArc(mRect, //弧线所使用的矩形区域大小
                -90,  //开始角度
                mSweepAngle, //扫过的角度
                false, //是否使用中心
                mBackgroundPaint); // 设置画笔

        super.onDraw(canvas);
    }
```

> 什么是插值动画？

为了让动画过度的更加自然或者添加一些动画效果，比如匀速运动、加速运动、减速运动、弹跳运动等等，这些的动画的效果就是靠插值来实现的。在Android中系统内置了一些插值，这里做下搬运工记录一下。推荐一个能在线运行Interpolator的效果以及数学公式定义的网站 [http://inloop.github.io/interpolator/](http://inloop.github.io/interpolator/) 更加直观的展示下面介绍的动画效果。


| 插值        |  说明   | 
| :--------   | :-----  |
| LinearInterpolator     | 以常量速率改变 | 
| BounceInterpolator     | 动画结束的时候弹起 | 
| CycleInterpolator     | 动画循环播放特定的次数，速率改变沿着正弦曲线 | 
| DecelerateInterpolator     | 在动画开始的地方快然后慢 | 
| OvershootInterpolator     | 向前甩一定值后再回到原来位置 | 
| AccelerateInterpolator     | 在动画开始的地方速率改变比较慢，然后开始加速 | 
| AnticipateInterpolator     | 开始的时候向后然后向前甩 | 
| AccelerateDecelerateInterpolator     | 在动画开始与介绍的地方速率改变比较慢，在中间的时候加速 | 
| AnticipateOvershootInterpolator     | 开始的时候向后然后向前甩一定值后返回最后的值| 


> 项目使用

这里要定义文本的宽高，因为没有画底部的黑色圆背景，还要设置一下背景图。

```xml
  <com.rae.cnblogs.widget.CountDownTextView
            android:id="@+id/tv_skip"
            style="@style/Widget.AppCompat.Button.Borderless"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="@drawable/bg_count_down"
            android:text="跳过"
            android:textColor="#ffffff"
            android:textSize="12sp" />
```

背景图

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <shape android:shape="oval">
            <solid android:color="#302d2d2d" />
        </shape>
    </item>
    <item>
        <shape android:shape="oval">
            <solid android:color="#7F2d2d2d" />
        </shape>
    </item>
</selector>
```

到这里结束啦，希望对你有帮助，[本篇文章的源码都在开源的博客园Android客户端这里。](https://github.com/raedev/android-cnblogs/blob/master/module-widget/src/main/java/com/rae/cnblogs/widget/CountDownTextView.java)
