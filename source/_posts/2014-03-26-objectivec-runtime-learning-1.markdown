---
layout: post
title: "ObjectiveC Runtime之对象模型"
date: 2014-03-26 07:36:12 +0800
comments: true
categories: ObjectiveC
description: ObjectiveC Runtime的对象模型及各种runtime技术的实现原理
---
ObjectiveC与C++同样是在C的基础上对C的扩展，但与C++相比，我认为ObjectiveC提供了更加强大的功能，比如Category，forward message, method-swiazzing, KVO以及一些runtime接口，而所有这些技术都是以ObjectiveC强大的Runtime为基础的。因此只有深入学习了runtime的机制，才能使我们对OC的使用更加得心应手， 并且对于大多数runtime技术，我们必须在熟悉它的底层实现机制后才能够达到正确使用的目的，否则的话你将陷入痛苦的debug之中。

对象模型是程序运行时对象在内存中的组织方式，由于ObjectiveC是OOP的编程语言，对象模型就涉及到`类的组织方式`,`继承关系的表示方法`，`成员变量与函数的存储方式`，`如何实现RTTI`等，因此对象模型是我们了解ObjectiveC runtime技术实现机制的基础。

###ObjectiveC中对象的数据结构###
我们都知道，ObjectiveC中的大多数类都是NSObject的子类，因此我们先从NSObject的结构开始,在NSObject.h中我们可以看到如下代码：
{% codeblock lang:objc %}
NS_ROOT_CLASS
@interface NSObject <NSObject> {
    Class	isa;
}
{% endcodeblock %}
NSObject类中只有一个名为`isa`的`Class`类型的变量，跳转到`Class`的定义我们可以看到：
{% codeblock lang:objc %}
typedef struct objc_class *Class;
typedef struct objc_object {
    Class isa;
} *id;
{% endcodeblock %}
`Class`为指向`objc_class`的指针，而`objc_class`就代表了在Objectivec中一个类的定义。其源代码定义如下：
{% codeblock lang:objc %}
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;
{% endcodeblock %}
我们看到`objc_class`继承自`objc_object`,而`objc_object`的定义如下：
{% codeblock lang:objc %}
struct objc_object {
private:
    uintptr_t isa;
{% endcodeblock %}
objc_object只保存了一个`isa`指针，在原代码中我们还可以看到

	typedef struct objc_object *id;
看到这里一切应该都已经明了了。我们知道在ObjectiveC中每一个对象都是一个`id`,所以`objc_object`就是我们每一个对象在内存中的结构，它保存了一个指向`objc_class`的指针，这个指针就指向对象所对应的类在内存中的位置。

既然`objc_class`代表了ObjectiveC中的类，那么结构中必然需要存储类的相关信息，包括类的名字，成员函数的位置等信息，在ObjectiveC2.0中，这些信息被封装在了`class_rw_t`和`class_ro_t`当中，他们代码定义如下：
{% codeblock lang:objc %}
struct class_rw_t {
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    union {
        method_list_t **method_lists;  // RW_METHOD_ARRAY == 1
        method_list_t *method_list;    // RW_METHOD_ARRAY == 0
    };
    struct chained_property_list *properties;
    const protocol_list_t ** protocols;

    Class firstSubclass;
    Class nextSiblingClass;
};

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;
    const method_list_t * baseMethods;
    const protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    const property_list_t *baseProperties;
};
{% endcodeblock %}
至此我们基本上了解了ObjectiveC中对象的组织形式，总结如下：

> 每一个ObjectiveC对象都保存了一个`isa`指针指向其所对应的类的实现`objc_class`，而`objc_class`中保存了类父类，名称，函数列表，以及Protocol等信息。

但是有些内容还不是特别清晰：

> + 对象的成员变量应该是局部于对象的，不应该保存在`objc_class`中，对于成员变量这部分的实现还不清晰
+ 对于大部分文章中提到的`元类`在代码中是如何体现的？ 目前我在代码中没有看到

以上ObjectiveC基本的对象模型，在这个对象模型的基础上ObjectiveC提供给了我们丰富的Runtime特性以及OO的支持，以后我们会进一步深入了解一些重要特性的实现。

我阅读的代码版本是Objc4-551，是官方开源的最新代码，如果文章中有错误或者有更详尽的看法请留言，共同学习进步。
  