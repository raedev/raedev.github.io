---
title: Android-View篇之自定义验证码输入框
date: 2019-05-09 09:22:54
tags: View
---

首先，我们来看看实现的是怎么样的效果：

![验证码输入框效果图](http://upload-images.jianshu.io/upload_images/2706530-587acc1084e20dcf?imageMogr2/auto-orient/strip)

如果我们拿到这样的UI，想到的布局应该是用4个`EditText`包在横向的`LinearLayout`里面，但今天要讲的View，所以我们决定用一个自定义的`EditText` 画出来。

<!-- more -->

> 学到什么？

- 基本理解画布概念
- 画布的状态、平移
- 布局测量
- 画图片

> 功能需求

- 高亮当前输入框
- 输入满4个数字自动调用方法


> 思路

完全重画一个`EditText`，就包含了`测量布局`和`重新绘制`这两个关键步骤。好了，到这里理一下整体的思路：

- 根据验证码个数以及边框大小来计算输入框显示的宽度
- 覆盖原来的`EditText`画布，重新绘制方框
- 根据输入的索引来确定高亮的方框
- 重写`onTextChanged` 但满足验证码个数的时候调用自动完成方法


> 开始动手

准备开始了，果断继承一个`AppCompatEditText` 来初始化基本参数先：

- 验证码个数
- 输入方框的大小
- 边框的大小及间距

```java 


/**
 * 验证码输入框,重写EditText的绘制方法实现。
 * @author RAE
 */
public class CodeEditText extends AppCompatEditText {
    
   //  验证码文本颜色
    private int mTextColor;
    // 输入的最大长度
    private int mMaxLength = 4;
    // 边框宽度
    private int mStrokeWidth;
    // 边框高度
    private int mStrokeHeight;
    // 边框之间的距离
    private int mStrokePadding = 20;
    // 用矩形来保存方框的位置、大小信息
    private final Rect mRect = new Rect();
    // 方框的背景
    private Drawable mStrokeDrawable;

   /**
     * 构造方法
     *
     */
    public CodeEditText(Context context, AttributeSet attrs) {
        super(context, attrs);
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CodeEditText);
        int indexCount = typedArray.getIndexCount();
        for (int i = 0; i < indexCount; i++) {
            int index = typedArray.getIndex(i);
            if (index == R.styleable.CodeEditText_strokeHeight) {
                this.mStrokeHeight = (int) typedArray.getDimension(index, 60);
            } else if (index == R.styleable.CodeEditText_strokeWidth) {
                this.mStrokeWidth = (int) typedArray.getDimension(index, 60);

            } else if (index == R.styleable.CodeEditText_strokePadding) {
                this.mStrokePadding = (int) typedArray.getDimension(index, 20);

            } else if (index == R.styleable.CodeEditText_strokeBackground) {
                this.mStrokeDrawable = typedArray.getDrawable(index);

            } else if (index == R.styleable.CodeEditText_strokeLength) {
                this.mMaxLength = typedArray.getInteger(index, 4);
            }
        }
        typedArray.recycle();

        if (mStrokeDrawable == null) {
            throw new NullPointerException("stroke drawable not allowed to be null!");
        }

        setMaxLength(mMaxLength);
        setLongClickable(false);
        // 去掉背景颜色
        setBackgroundColor(Color.TRANSPARENT);
        // 不显示光标
        setCursorVisible(false);
    }

    @Override
    public boolean onTextContextMenuItem(int id) {
        return false;
    }

   /**
     * 设置最大长度
     */
    private void setMaxLength(int maxLength) {
        if (maxLength >= 0) {
            setFilters(new InputFilter[]{new InputFilter.LengthFilter(maxLength)});
        } else {
            setFilters(new InputFilter[0]);
        }
    }
}
```

> 开始测量布局

初始化完了就要开始测量布局了，计算公式为：

`输入框宽度 = 边框宽度 * 数量 + 边框间距 *（数量-1）`


```java

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        // 当前输入框的宽高信息
        int width = getMeasuredWidth();
        int height = getMeasuredHeight();
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        // 判断高度是否小于推荐高度
        if (height < mStrokeHeight) {
            height = mStrokeHeight;
        }

        // 输入框宽度 = 边框宽度 * 数量 + 边框间距 *(数量-1)
        int recommendWidth = mStrokeWidth * mMaxLength + mStrokePadding * (mMaxLength - 1);
        // 判断宽度是否小于推荐宽度
        if (width < recommendWidth) {
            width = recommendWidth;
        }

        widthMeasureSpec = MeasureSpec.makeMeasureSpec(width, widthMode);
        heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, heightMode);
        // 设置测量布局
        setMeasuredDimension(widthMeasureSpec, heightMeasureSpec);
    }

```

> 画家登场

来到最重要的步骤了，重画输入框！来一步步看代码注释：

```java
    @Override
    protected void onDraw(Canvas canvas) {
        // 在画支持设置文本颜色，把系统化的文本透明掉，相当于覆盖
        mTextColor = getCurrentTextColor();
        setTextColor(Color.TRANSPARENT);
        //  系统画的方法
        super.onDraw(canvas);
        // 重新设置文本颜色
        setTextColor(mTextColor);
        // 重绘背景颜色
        drawStrokeBackground(canvas);
        // 重绘文本
        drawText(canvas);
    }
```

**绘制背景方框**

```java

    /**
     * 绘制方框
     */
    private void drawStrokeBackground(Canvas canvas) {
        // 下面绘制方框背景颜色
        // 确定反馈位置
        mRect.left = 0;
        mRect.top = 0;
        mRect.right = mStrokeWidth;
        mRect.bottom = mStrokeHeight;
        int count = canvas.getSaveCount(); //  当前画布保存的状态
        canvas.save(); // 保存画布
        for (int i = 0; i < mMaxLength; i++) {
            mStrokeDrawable.setBounds(mRect); // 设置位置
            mStrokeDrawable.setState(new int[]{android.R.attr.state_enabled}); // 设置图像状态
            mStrokeDrawable.draw(canvas); //  画到画布上
            //  确定下一个方框的位置
            float dx = mRect.right + mStrokePadding; // X坐标位置
            // 保存画布
            canvas.save();
            // [注意细节] 移动画布到下一个位置
            canvas.translate(dx, 0);
        }
        // [注意细节] 把画布还原到画反馈之前的状态，这样就还原到最初位置了
        canvas.restoreToCount(count);
        // 画布归位
        canvas.translate(0, 0);

        // 下面绘制高亮状态的边框
        // 当前高亮的索引
        int activatedIndex = Math.max(0, getEditableText().length());
        mRect.left = mStrokeWidth * activatedIndex + mStrokePadding * activatedIndex;
        mRect.right = mRect.left + mStrokeWidth;
        mStrokeDrawable.setState(new int[]{android.R.attr.state_focused});
        mStrokeDrawable.setBounds(mRect);
        mStrokeDrawable.draw(canvas);

    }

```

一般画布的移动`canvas.translate(x,y)`会结合`canvas.save();`来使用。
1、调用`canvas.save();`保存当前画布的状态，用PS来解析就是按下`ctrl +s`键，然后帮你新建一个新的图层。你之后画的内容不会影响到之前画的内容，要回到之前的状态就调用`canvas.restoreToCount(count)`来还原。
2、把画布的位置移到下一个位置`canvas.translate(x,y)`，下图所示，你会发现方框在画布中的位置没有发生变化而是画布距离发生了变化。这就是画布平移的效果了。

![画布平移](http://upload-images.jianshu.io/upload_images/2706530-d5aa961032802f4e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**画验证码文字**

```java
    /**
     * 重绘文本
     */
    private void drawText(Canvas canvas) {
        int count = canvas.getSaveCount();
        canvas.translate(0, 0);
        int length = getEditableText().length();
        for (int i = 0; i < length; i++) {
            String text = String.valueOf(getEditableText().charAt(i));
            TextPaint textPaint = getPaint();
            textPaint.setColor(mTextColor);
            // 获取文本大小
            textPaint.getTextBounds(text, 0, 1, mRect);
            // 计算(x,y) 坐标
            int x = mStrokeWidth / 2 + (mStrokeWidth + mStrokePadding) * i - (mRect.centerX());
            int y = canvas.getHeight() / 2 + mRect.height() / 2;
            canvas.drawText(text, x, y, textPaint);
        }
        canvas.restoreToCount(count);
    }
```

**监听文本变化回调自动完成方法**

```java
    @Override
    protected void onTextChanged(CharSequence text, int start,
                                 int lengthBefore, int lengthAfter) {
        super.onTextChanged(text, start, lengthBefore, lengthAfter);

        // 当前文本长度
        int textLength = getEditableText().length();

        if (textLength == mMaxLength) {
            hideSoftInput();
            if (mOnInputFinishListener != null) {
                mOnInputFinishListener.onTextFinish(getEditableText().toString(), mMaxLength);
            }
        }

    }
```


[查看完整的源码](https://github.com/raedev/android-cnblogs/blob/master/module-widget/src/main/java/com/rae/cnblogs/widget/CodeEditText.java)

到这里你能大概理解画布的概念了，本文完。
