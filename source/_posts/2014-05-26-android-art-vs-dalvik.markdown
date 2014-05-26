---
layout: post
title: "Android ART 与 Dalvik虚拟机比较"
date: 2014-05-26 21:39:15 +0800
comments: true
categories: Android
description: 比较Android Runtime 与 Dalvik虚拟机的区别
---
长久以来Android给人们的印象都是反应速度慢，不够流畅，这也是我之前对android的印象，在接触了Android4.4、从事了android的开发后，我觉得Android之所以给人们不够流畅的原因有以下几点：

*	与iOS不同的是，Android手机型号众多，配置更是各种各样，以我之前用的HTC手机为例，2012年出厂的手机装配了Android4.0的系统，但是系统的内存却只有512M，开机后app的可用内存只有可怜的360M，这种配置的手机怎么可能有好的表现，不卡才怪。而iOS的手机型号固定，配置起码都是1G的内存，运行流畅是必然的。
*	Android市场对app的审核不如iOS严格，致使Android App的质量良莠不齐，很多开发者并没有对app进行足够的性能优化，成功如微博客户端，在滑动的时候仍不够流畅，其他的app质量可想而知。
*	最根本的一点是，Dalvik虚拟机在作怪。

大家都知道，Java为了达到跨平台的目的，在编译时将源代码转化为了字节码，Android的Dalvik虚拟机亦是如此，在我们运行app时，系统必须将字节码转换为机器码，然后运行，这种机制就是Dalvik使用的JIT（Just-in-time compilation）机制。在运行时才将字节码转化为机器码，这可以说是android app不够流畅的根本原因，因为我们无法消除将字节码转化为机器码的这部分时间。

Android4.4引入了ART，即Android Runtime，与Dalvik不同的是，ART在我们第一次安装app时，会自动将app转化为机器码，从而确保了app运行时的时间（去除了JIT compile的时间），因此开启ART后，android app流畅度会有一个不错的提升。但是目前，ART有如下问题：

*	某些app在ART下无法运行。这应该是针对一小部分app，因为我目前为止在我的手机上没有发现这个问题
*	由于在首次安装时需要将app转化为机器码，因此app的安装时间会有所增加。我在使用过程中也没有明显的感觉到这一点，可以说这点不会妨碍一般的用户。

可以说ART很好的解决了android不够流畅的问题，并且对于普通用户，并没有带来使用的负面影响，况且现在ART还在进一步的改进和开发当中，相信以后在流畅度，安装时间等各个方面会做的更好。 对于我们开发者，ART也带来了实际的好处。

相信很多开发者会遇到一个问题，就是在Android中，如果你的ViewGroup嵌套层次太多，那么很可能会遇到StackOverflowError这个运行时错误，产生这个错误的原因是由于View Hierarchy在绘制过程中，函数调用的深度超出了系统主线程的Stack frame。在Android4.4中，Dalvik的UI线程的Stack frame只有16K，过多的嵌套很容易造成StackOverflowError，而对于程序，很难从根本上解决这个问题（优化view hierarchy被认为是唯一可行的方法，但实用性却是有限的）。我在试验中发现，ART已经很好的解决了这个问题，即使view group的嵌套达到了60层，仍然不会出现在Dalvik中的crash。

由此可见，ART正在试图解决一些Dalvik中存在的性能等各个方面的问题，或许Android5.0的时候，ART能够成型，那Android必然会向前迈一大步，用户体验可以得到很大的改观（当然像之前说的HTC的那种奇葩配置的机器，永远不可能有好的流畅度）。

由于Android生态的多样性，各个生产商的Android都有很大的不同，我们应该通过比较来区分优劣，Android有自己的特点，包括Android App的设计原则都不同于iOS，作为Android用户，我们需要的不是iOS，而是Android自己的个性，Android的特色。越来越对Android充满信心，相信Android很快可以做的更好。
