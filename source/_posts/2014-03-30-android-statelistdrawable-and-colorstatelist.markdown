---
layout: post
title: "Android通过代码设置控件点击时颜色变化"
date: 2014-03-30 12:54:00 +0800
comments: true
categories: Android
description: android通过代码设置View 不同状态时背景颜色和字体颜色
---
在UI设计时，我们通常希望用户得到及时的交互反馈，例如在按钮上发生点击事件，或者textView上发生点击时，我们通常采用highlight一个控件的方法，提醒用户点击事件的发生。我们通常会通过改变控件的背景颜色或者字体的颜色来达到高亮的效果，Android提供了两个类，StateListDrawable和ColorStateList来支持这种操作。

在Android的[官方文档](http://developer.android.com/guide/topics/resources/color-list-resource.html)以及网上很多资料中，我们可以看到这种高亮的支持都会通过编写xml来实现的。个人认为xml的实现方式虽然简单、可复用，但是如果我们有很多种状态，或者要动态的产生不同的颜色效果，那么XML的方式会使我们的工程中增加很多的resource文件，作为一个Programmer我更喜欢通过代码来达到相同的目的。本文所用的例子比较简单，仅仅是对一个textview做了高亮设置，当用户点击textiew时，会改变其背景色和字体颜色，可以在[这里](https://github.com/Charlesjean/FragmentTest)下载代码。关键代码如下:
{% codeblock lang:java %}

    /*
    *Add statedrawable for targetView to display different drawable when
    * view is in different state
     */
    public void addStateDrawable(View targetView)
    {
        StateListDrawable listDrawable = new StateListDrawable();
        //add drawable used when view is pressed
        listDrawable.addState(new int[]{android.R.attr.state_pressed}, new ColorDrawable(Color.rgb(106,170,234)));
        //show gray background for all other state
        listDrawable.addState(StateSet.WILD_CARD, new ColorDrawable(Color.GRAY));
        targetView.setBackgroundDrawable(listDrawable);

    }

    /**
     * set different text color for different state
     */
    public void addColorState(TextView targetView)
    {
        int[][] states = {
                new int[]{android.R.attr.state_pressed},
                StateSet.WILD_CARD};
        int[] colors = {Color.RED, Color.BLACK};
        ColorStateList coloList = new ColorStateList(states, colors);
        targetView.setTextColor(coloList);
    }
{% endcodeblock %}

StateListDrawable 提供了`addState`接口，使我们能够对应每一种状态，设置一个drawable；ColorStateList同样使我们为每一种状态指定一种字体颜色。 其中`StateSet.WILD_CARD`表示去除当前已有状态外的其他所有状态。 

所谓状态就是指随着用户的交互，View所出的不同的状态。 其中`state_pressed`表示` when the user is pressing down in a view. `,更多对状态的描述可以参考[android 文档](http://developer.android.com/reference/android/graphics/drawable/StateListDrawable.html) (个人认为android的很多文档写得很烂，完全是对字面意思的解释，这一点与iOS根本无法相比)。

###使用StateListDrawable和ColorStateList需要注意的事项###

* 首先由于控件的状态是随着用户的点击等事件而改变的，所以要想使之前的代码能起作用，我们必须确保view能够接受用户的点击事件，最基本的必须用`setClickable(true)`,使控件可以点击
* 由于android对点击等事件的传递有一定的规则，要使`StateListDrawable`和`ColorStateList`起作用，最根本上要保证view的**onTouchEvent**函数能够被调用到，即TouchEvent不能被其他的对象 如`ParentView`和`TouchListener`所截断。(如果view注册了EventListener那么事件就会在传给onTouchEvent()函数之前，被EventListener截断)。

关于Android中事件的传递机制，可以参考[这篇文章]()。
