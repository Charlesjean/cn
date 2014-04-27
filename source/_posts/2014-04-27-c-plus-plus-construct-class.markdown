---
layout: post
title: "在设计C++的类时我们需要考虑哪些内容"
date: 2014-04-27 11:32:33 +0800
comments: true
categories: C_Plus_Plus
---
作为一种面向对象的编程语言，类是我们使用C++时频繁使用的单位，由于C++语言的某些特性，与java、Objectivec等语言相比，C++在对多态的实现上有一些不同。java与ObjectiveC是通过runtime来实现多态的，即系统在运行时根据对象的类型来决定，而C++是通过编译时对象的类型来决定调用的函数（C++中的多态是通过指针与virtual 函数来是实现的）。我们先通过例子来看C++与ObjectiveC在多态方面的不同。

{% codeblock lang:objc %}
//base class
@interface Shape : NSObject
- (void)draw;
@end

@implementation Shape
- (void)draw
{
    NSLog(@"Shape Draw");
}
@end

//derived class
@interface Rectangle : Shape
@end
@implementation Rectangle
- (void)draw
{
    NSLog(@"Rectangle Draw");
}
@end

//test
Shape* s = [[Shape alloc] init];
[s draw];//will call draw defined in shape
Shape* r = [[Rectangle alloc] init];
[r draw];//will call draw definied in rectangle
{% endcodeblock%}
对应的类在C++中定义如下：
{% codeblock lang:c++%}
#include<iostream>
class Shape
{
public:
	Shape();
	~Shape();
	void draw(){ std::cout<<"Shape Draw"; }
};

class Rectangle :public Shape
{
public:
	void draw(){ std::cout << "Rectangle Draw"; }
};

//test
	Shape* s = new Shape;
	s->draw();//will call draw definied by Shape
	Shape* r = new Rectangle;
	r->draw();//will also call draw defined by Shape
{% endcodeblock%}

这个例子可以看出来，在C++中，如果函数没有被声明为Virtual，那么通过指针是无法实现多态的。这种机制，带来了代码运行的效率，但是却给我们编码人员提出了更大的要求，我们不能单纯的凭感觉使用c++，使用c++需要我们同时进行思考，将一些必须考虑的内容时刻放在脑子里。

在设计我们自己的类时，我们通常需要审视以下方面的内容（来自C++沉思录）：

*	类中是否需要构造函数
*	类中成员变量的作用域（private or public）
*	是否需要定义默认构造函数
*	是否每一个成员变量都需要在每一个构造函数中初始化
*	是否需要定义析构函数（*）
*	是否需要将析构函数定义为Virtual
*	是否需要定义拷贝构造函数，移动构造函数（*）
*	是否需要重载赋值操作符（*）
*	赋值操作符的重载是否正确（是否有效的处理了自赋值情况）（*）
*	拷贝构造函数和赋值操作符的参数是否确保使用了const限制符（*）
*	传入引用参数时，是否应该使用const修饰符

以上这些条目，应该是我们实现类时需要注意的内容，其中标记`*`的我们需要进一步解释。

####析构函数、拷贝构造函数、赋值操作符####
析构函数、拷贝构造函数和赋值操作符被称作C++的`Big Three`,因为通常情况下，如果我们定义了其中一个，我们同样需要定义其他两个。

当我们的类中，具有不能自动释放的资源（通常是一些指针），那么我们需要定义析构函数，在我们的对象析构时，确保资源能够还给系统，从而避免资源的泄露。在这种情况下，`我们同样不能够使用系统自己提供的默认的拷贝和赋值的定义`。下面的例子可以告诉我们原因：
{% codeblock lang:c++ %}
class Shape
{
public:
	Shape();
	~Shape();
	void draw(){ std::cout << "Shape Draw"; }
	void changeStr(){ *str = "I am changed"; }
	void printStr(){ std::cout << *str << std::endl; }
private:
	std::string* str;
};
//test
Shape s;
Shape b(s);
b.changeStr();
s.printStr();//result "I am changed"
b.printStr();//result "I am changed"
{% endcodeblock %}

通过结果我们可以看出，系统默认提供的拷贝构造函数和赋值操作符实现了`bit-wise`的拷贝，拷贝后，例子中的两个对象中的指针成员指向了同样的内存地址，当我们改变一个的内容时，另一个对象的内容也被改变了（基本上这不会是我们希望的）。

因此，在我们实现类时，`析构函数`，`拷贝构造函数`，`赋值操作符`，`移动构造函数`这四者是息息相关的，我们必须谨慎思考如何适当的实现他们，并且需要确保能够正确和高效的实现，关于他们的实现方法，可以参考之前的一篇[文章](http://dj-chen.com/cn/blog/2014/04/13/c-plus-plus-11-move-semantic/)。

关于赋值操作符的重载方式，以及传入引用或者const引用的问题，同样可以参考上面文章中的实现。