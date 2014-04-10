---
layout: post
title: "深入ObjectiveC Runtime之(三)NSObject类与NSObject接口"
date: 2014-04-09 20:05:43 +0800
comments: true
categories: ObjectiveC
description: interface NSObject and protocol NSObject
---
在学习ObjectiveC Runtime代码是你会发现，在系统中除了一个名为NSObject的类之外，还存在一个同样名字为NSObject的接口（Protocol），这不禁让我们有疑问，NSObject接口存在的原因是什么，同样的名字难道不会有冲突吗？

我们先看一下接口 NSObject的代码
{% codeblock lang:objc %}
@protocol NSObject

- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;

- (Class)superclass;
- (Class)class;
- (id)self;
- (struct _NSZone *)zone OBJC_ARC_UNAVAILABLE;

- (id)performSelector:(SEL)aSelector;
- (id)performSelector:(SEL)aSelector withObject:(id)object;
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;

- (BOOL)isProxy;

- (BOOL)isKindOfClass:(Class)aClass;
- (BOOL)isMemberOfClass:(Class)aClass;
- (BOOL)conformsToProtocol:(Protocol *)aProtocol;

- (BOOL)respondsToSelector:(SEL)aSelector;

- (id)retain OBJC_ARC_UNAVAILABLE;
- (oneway void)release OBJC_ARC_UNAVAILABLE;
- (id)autorelease OBJC_ARC_UNAVAILABLE;
- (NSUInteger)retainCount OBJC_ARC_UNAVAILABLE;

- (NSString *)description;
@optional
- (NSString *)debugDescription;

@end
{% endcodeblock %}
我们平时使用的runtime类型判断接口，以及`performSelector`等都定义在以NSObject为名的Protocol当中。这些方法在我们平时的编程过程中被大量使用，基本上所有的类都可以respond to这些方法，而我们的类都是继承自NSObject的，他们为什么会有NSObject Protocol当中声明的方法呢，唯一的解释就是`Class NSObject 实现了 NSObject Protocol`，在runtime的代码中我们也可以得到印证
{% codeblock lang:objc %}
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
{% endcodeblock %}
实际上在ObjectiveC中，interface和protocol的名字是可以相同的，因为在语言中不存在任何可以产生歧义的语法，也就是说interface和protocol使用于完全不同的语法环境，因此可以有相同的名字。

我们还可以进一步思考NSObject Protocol存在的必要性。在JAVA中，所有的类都是java.lang.Object的子类（Object是唯一的ROOT Class），而ObjectiveC中并不是所有的类都是NSObject的子类（有些类继承自NSProxy而不是NSObject）。因此在ObjectiveC中，为了给所有的类提供某些通用的接口，就必须保证所有的ROOT Class，都去实现某一个Protocol，NSObject Protocol就因此诞生，即`所有的root class，包括NSObject和NSProxy都会实现NSObject Protocol`， 我想这才是NSObject Protocol存在的根本原因。
