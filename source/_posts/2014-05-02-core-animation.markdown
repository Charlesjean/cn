---
layout: post
title: iOS中Core Animation的使用
date: 2014-05-02 21:26:21 +0800
comments: true
categories: iOS
---
用户体验是用户在使用一款软件和系统时，对软件和系统能得到的最直观的感受，iOS一直因为良好的用户体验而受到大家的赞赏，流畅的操作和丰富的动画无疑给iOS的用户体验加了不少分。对于我们编程人员来说，iOS中提供了强大的Core Animation库，并封装了友好的API，使我们能够实现复杂的动画效果。

Core Animation提供了两种动画的类型，并封装成类CABasicAnimation和CAKeyframeAnimation，我们需要针对自己的目标来选择使用这两种类。动画的实现过程实际上就是在一定时间内连续播放一系列的视频帧，在iOS中要求60帧/秒，通常情况下，我们实现动画的做法是给出动画在一定时间内的`关键帧`，然后利用`差值`的方法得到关键帧之间的所有帧序列的状态。CABasicAnimation 和 CAKeyframeAnimation的最大区别就是
>		CABasicAnimation要求我们只能提供动画的第一帧和最后一帧，然后系统利用我们指定的插值方法，计算出整个动画的所有序列；
		而利用CAKeyframeAnimation我们可以给出动画过程中的多个关键帧，然后同样系统利用我们制定的插值方法，计算出连续的两个关键帧之间的所有序列，从而实现动画。 
		可以看到，利用CAKeyframeAnimation，我们对动画有更多的控制权，因此大多数比较复杂的动画也需要选用CAKeyframeAnimation。

我在[demo](https://github.com/Charlesjean/CoreAnimationDemo)中实现了两种动画效果，动画并不算复杂，但是却能很好的了解这两种动画类的使用，并且由于动画是作用在CALayer之上，因此CALayer的相关知识也是我们能够正确的使用Core Animation的基础，在例子中你也可以看到一些CALayer的属性，如anchor point，transform等是如何使用的。demo的效果如下：
{% video 720 480 https://raw.githubusercontent.com/Charlesjean/CoreAnimationDemo/master/demo.mp4 %}

Demo中含有两种动画效果，当我们像左滑动时，触发了folder animation，当我们向右滑时触发了accordion animation。
####Folder Animation
Folder动画给我们的感觉是类似翻书的效果，要实现这个动画我们首先需要考虑如何将动画分解，及动画中我们将涉及到CALayer的布局如何，我们实现动画所需要改变的属性是什么。按照folder动画的效果，我们可以将这个动画分解为下图所示：
![]()

####Accordion Animation


