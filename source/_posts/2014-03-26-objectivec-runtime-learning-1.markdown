---
layout: post
title: "ObjectiveC Runtime之对象模型"
date: 2014-03-26 07:36:12 +0800
comments: true
categories: ObjectiveC
description: ObjectiveC Runtime的对象模型及各种runtime技术的实现原理
---
ObjectiveC与C++同样是在C的基础上对C的扩展，但与C++相比，我认为ObjectiveC提供了更加强大的功能，比如Category，forward message, method-swiazzing, KVO以及一些runtime接口，而所有这些技术都是以ObjectiveC强大的Runtime为基础的，只有深入学习了runtime的机制，才能使我们对OC的使用更加得心应手， 并且对于大多数runtime技术，我们必须在熟悉它的底层实现机制后才能够达到正确使用的目的，否则的话你将陷入痛苦的debug之中。

对象模型是程序运行时对象在内存中的组织方式，由于ObjectiveC是OOP的编程语言，对象模型就涉及到`类的组织方式`,`继承关系的表示方法`，`成员变量与函数的存储方式`，`如何实现RTTI`等，因此对象模型是我们了解ObjectiveC runtime技术实现机制的基础。