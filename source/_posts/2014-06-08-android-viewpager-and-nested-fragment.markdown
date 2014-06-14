---
layout: post
title: "Android ViewPager and Nested Fragment"
date: 2014-06-08 23:28:49 +0800
comments: true
categories: Android
---

范佩西的头球，老罗的速度，我这个伪球迷也被震撼了!

Fragment在android的设计中变得越来越重要，它比Activity轻量，又具有自己的生存周期，加上默认的Fragment的backstack的支持，使的Fragment使用起来十分得心应手。

MVC模式是Mobile app遵循的基本设计模式，Model负责数据的表示，View负责数据的显示，Controller负责协调Model和View之间的工作。在Mobile App中，每一屏一般就是一个MVC，而具体到Android中，Fragment可以允许我们很好地将每一屏的逻辑包含在内，它负责内容（View）的生成，协调View和Model间的协作，基本上就是MVC中Controller的逻辑。下面情况 我们都可以用fragment：

*	界面的分屏显示，每一屏都是一个fragment来表示
*	与ViewPager配合使用，每一个page都作为一个Fragment来表示
*	在tablet上的布局中，对master-detail模式的app，导航与具体的信息显示可以由不同的fragment来表示。
*	在Navigation Drawer导航模式中，每一个导航都推荐设计为一个fragment

在Android4.2之前，Fragment不支持嵌套的布局，就是说Fragment内部不能再有子Fragment。在使用viewpager与fragment结合的时候，我们可以发现，如果viewpager放在一个fragment当中，而viewpager当中的每一个page又以一个fragment来实现，那么内部的fragment（每一个page）的生命周期就会发生混乱。

在Android4.2之后，系统添加了推[nested Fragment](http://developer.android.com/about/versions/android-4.2.html#NestedFragments)的支持。如果要兼容之前的版本，建议使用Support Library v4。

好记性不如烂笔头，just for mark。



