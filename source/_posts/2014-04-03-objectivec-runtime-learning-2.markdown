---
layout: post
title: "深入ObjectiveC Runtime之（二）安装Clang"
date: 2014-04-03 21:20:48 +0800
comments: true
categories: ObjectiveC
description: 深入学习ObjectiveC Runtime的实现与技术
---
在[上一篇](http://dj-chen.com/cn/blog/2014/03/26/objectivec-runtime-learning-1/)中我们通过阅读代码，了解了ObjectiveC的内部对象模型的实现，接下来我们继续通过代码来了解ObjectiveC是如何加载一个类，以及ObjectiveC是如何实现对Category的支持的。

我们需要利用Clang来生成一部分中间代码，帮助我们了解runtime的运行机制，因此我们首先需要安装Clang。

Clang是LLVM的前端，可以用来编译C，C++，ObjectiveC等语言。传统的编译器通常分为三个部分，前端(frontEnd)，优化器(Optimizer)和后端(backEnd)。在编译过程中，前端主要负责词法和语法分析，将源代码转化为抽象语法树；优化器则是在前端的基础上，对得到的中间代码进行优化，使代码更加高效；后端则是将已经优化的中间代码转化为针对各自平台的机器代码。Clang则是以LLVM为后端的一款高效易用，并且与IDE结合很好的编译前端。

我们需要自己下载LLVM与Clang源代码进行编译，步骤如下：

{% codeblock lang=shell %}
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm 

cd llvm/tools 
svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
cd ../..

cd llvm/projects
svn co http://llvm.org/svn/llvm-project/compiler-rt/trunk compiler-rt
cd ../..

mkdir build 
cd build
../llvm/configure
make

{% endcodeblock %}
至此，Clang 安装完毕，你可以使用`clang --help`来验证是否正确安装。

工欲善其事必先利其器，安装Clang主要是为我们今后的学习做准备，我本身对编译器没有特别深入的学习，如果你对Clang感兴趣可以[参考这里](http://clang.llvm.org/)。
