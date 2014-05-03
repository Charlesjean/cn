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

当我们在屏幕上滑动时，view controller会移除当前view（currentView），并添加新的view（nextView），并通过动画来达到比较优美的效果。Demo中含有两种动画效果，当我们像左滑动时，触发了folder animation，当我们向右滑时触发了accordion animation。

####Render tree 和 Presentation tree
CALayer与UIView类似，对于一个有UIView组成的View Hierarchy，同样也有一个对应的Layer Hierarchy。但是Layer Hierarchy又对应两种不同的树结构，我们称作Render tree 和 Presentation tree，理解了render tree和presentation tree后，有利于我们解决动画过程当中的某些问题（在动画开始和结束时候的可能出现的闪烁问题）。

我们以一个例子来解释这两个概念。
在CALayer中很多属性是可以触发动画的，我们看如下简单代码：
{% codeblock lang:objc %}
	layer.backgroundColor = [UIColor redColor].CGColor;
{% endcodeblock %}
在layer的背景色被改变后，我们可以看到动画中layer的颜色会由原来的颜色，慢慢过渡为红色。在动画中我们看到的layer的状态（颜色随时间而变化）就是presentation tree中layer的状态，而当动画结束后，layer最终呈现的状态就是render tree中的状态。换句话说，动画中我们改变的是presentation tree中layer的状态，而不是layer最终呈现的状态，如果我们将改变颜色的一个CABasicAnimation对象作用到layer上，动画会呈现出来（presentation tree）而layer在动画结束的瞬间仍然会变回原来的颜色（render tree）。

####Folder Animation
Folder动画给我们的感觉是类似翻书的效果，要实现这个动画我们首先需要考虑如何将动画分解，及动画中我们将涉及到CALayer的布局如何，我们实现动画所需要改变的属性是什么。按照folder动画的效果，我们可以将这个动画分解为下图所示：

![](https://raw.githubusercontent.com/Charlesjean/cn/master/source/_posts/folder.png)

直观上我们可以看到图中包含了三个CALayer，黄色的leftLayer，绿色的rightLayer，已经处于旋转状态的rotateLayer。动画的整个过程就是rotateLayer绕Y轴旋转180度的过程。

folder动画应该满足的要求为：
*	rotateLayer在旋转的时候应该有透视（Perspective）的效果
*	rotateLayer似乎有两个面，正面显示的内容为currentView的右半部分，反面显示的为nextView的左半部分

查看代码，在FolderAnimation.mm中，我们可以看到以下的变量声明
{% codeblock lang:objc %}
@interface FolderAnimation()
{
    CATransformLayer* animationLayer;//rotateLayer
    CALayer* frontAnimationLayer;
    CALayer* backAnimationLayer;
    
    CALayer* currentViewHalfLayer;//leftLayer
    CALayer* nextViewHalfLayer;//rightLayer
    
}
{% endcodeblock %}
我们实际需要使用5个Layer来完成这个动画，其中需要旋转并带有透视效果的layer我们选择的是CATransformLayer，在CATransformLayer上，我们需要放置两个CALayer，作为它的"正面"和"反面"。

CALayer的绘制是二维的，也就是说在将CALayer的整个Layer Hierarchy绘制到屏幕上之前，CALayer当中所有的layer都会被先绘制到一张bitmap上面，然后系统再将这张bitmap显示出来。这种绘制方式决定了，我们的要求2无法满足，也就是说一个layer不可能有两个不同的面，这种情况下我们就需要使用CATransformLayer。CATransformLayer允许我们建立三维的Layer Hierarchy，也就是说它的所有的Sub Layer存在于一个三维的空间内，我们可以对他们进行旋转、平移等操作，也正是由于CATransformLayer模拟了三维的空间，所以它可以提供给我们透视（perspective）的效果（具体可以参见CATransformLayer的文档）。

建立好了用于动画的Layer Hierarchy之后，下一步就是要向rotateLayer添加动画，因为本例中只涉及了一个转动的动画，所以我们选择使用CABasicAnimation
{% codeblock lang:objc %}
    CABasicAnimation* animation = [CABasicAnimation animationWithKeyPath:@"transform"];
    animation.delegate = self;
    animation.removedOnCompletion = YES;
    CATransform3D transform = CATransform3DIdentity;
    transform = CATransform3DRotate(transform,  direction * M_PI, 0, 1, 0);
    transform.m34 = direction * 1.0/1000;
    animation.toValue = [NSValue valueWithCATransform3D:transform];
    animation.duration = self.duration;
    [animationLayer addAnimation:animation forKey:Nil];
{% endcodeblock %}
动画的代码比较简单，我们将图中的rotateLayer旋转了180度，需要注意的有两点：

*	1.transform.m34的设置是为了产生透视效果，至于transform的意义涉及到图形学中的视图投影矩阵的内容，会在另一篇文章中阐述
*	2.animation.delegate的设置是为了监听动画的结束，以便做一些动画结束时的工作（保证presentation tree的最后状态与render tree一致）。

####Accordion Animation
相对于folder animation来说accordion animation的实现要复杂许多。


