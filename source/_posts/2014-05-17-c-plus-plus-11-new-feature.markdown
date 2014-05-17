---
layout: post
title: "C++11 常用新语言特性总结"
date: 2014-05-17 14:07:31 +0800
comments: true
categories: C_Plus_Plus
description: c++11 new feature summary, c++11新语言特性
---
C++11给我们提供了很多新的特性，相比于C++98，C++11在很多地方使用起来更加顺手，很多新的特性让C++变得更加让人喜欢，对编码人员更加友好，很多规则更加统一。在这些新的特性当中，有很多高级特性，比如关于模板的特性（个人真的很少在开发中用到）；也有很多确确实实方便了开发，而且使c++代码更加整洁、安全、快速的特性。在学习中总结一下，往往可以在使用过程中更加得心应手。

####auto####
c++11中auto关键字和c++98中有完全的不同，在c++11中，如果我们把变量声明为auto，那么编译器会自动根据上下文推断出变量的类型，因此在很多情况下，特别是类型比较长的时候，我们可以使用auto来使代码更加整洁：
{% codeblock lang:c++ %}
vector<String> vec;
.....

auto iter = begin(v);//get iterator of vector,do not need to use vector<String>::iterator
{% endcodeblock%}
乍一看，这仅仅是提供了一些编程上的方便，但是实际上在很多类型不确定的时候，auto提供了更大的帮助
{% codeblock lang:c++ %}
	template<class T, class U> 
void multiply(const vector<T>& vt, const vector<U>& vu)
{
	auto tmp = vt[i]*vu[i];//here it's hard to decide type of tmp, auto saves us a lot
}
{% endcodeblock %}
####智能指针####
在c++11中，智能指针使用起来更加顺手，相比于之前的auto_ptr, unique_ptr和shared_ptr更加容易的和stl容器一起使用。unique_ptr顾名思义为独有指针，被unique_ptr包含的指针在任意时刻只能有一个unique_ptr拥有，任何对unique_ptr的赋值和拷贝都会产生指针的移动；shared_ptr是实现了引用计数的智能指针，相对于unique_ptr，shared_ptr需要维护一个计数对象，因此在赋值和拷贝时，较unique_ptr会产生更大的开销。weak_ptr是用来避免循环引用。（在另一篇文章中，我们会比较这几种智能指针的使用）
{% codeblock lang:c++ %}
class gadget;
 
class widget {
private:
    shared_ptr<gadget> g; // if shared ownership
};
 
class gadget {
private:
    weak_ptr<widget> w;//avoid cycle reference
};
{% endcodeblock %}

####nullptr####
在C++11中我们可以使用nullptr来指定一个指针为空，而不再使用0 或者NULL。
{% codeblock lang:c++ %}
char* p = nullptr;
int* q = nullptr;
char* p2 = 0;//do not use this any more, if you use, p==p2 is still ok

//for function
void f(int);
void f(char*);
f(0);//will call f(int)
f(nullptr);//will call f(char*)
{% endcodeblock %}

####Range for####
c++11中也引入了更加有好的for循环语法
{% codeblock lang:c++ %}
//c++98 
for (vector<int>::iterator i = v.begin(); i != v.end(); ++i)
{
	...
}
//c++11
for (audo d : v)
{
	...
}
{% endcodeblock %}
Range-for语法不仅适用于stl容器，还可以用于string, array。
####不需使用容器的begin和end成员函数####
begin()和end()方法不仅仅可以用于stl 容器，还可以用于数组当中，看如下的对比代码：
{% codeblock lang:c++ %}
	vector<int> v;
	int a[100];
//c++98
std::sort(v.begin(), b.end());
std::sort(&a[0], &a[0] + sizeof(a) / sizeof(a[0]));

//c++11
std::sort(begin(v), end(v));
std::sort(begin(a), end(a));
{% endcodeblock %}
对比如此明显，你还会用原来的语法吗？

####Move Semantic####
可以参考[这篇文章](http://dj-chen.com/cn/blog/2014/04/13/c-plus-plus-11-move-semantic/)

####类成员变量的初始化####
在c++11中允许我们在声明成员变量时给出初始化，避免了需要在所有的构造函数中提供变量的初始化：
{% codeblock lang:c++ %}
class A {
	public:
		A() {}
		A(int a_val) : a(a_val) {}//a_aval will replace 7
		A(D d) : b(g(d)) {}
		int a = 7;//this is ok
		int b = 5;//this is ok
	private:
		HashingFunction hash_algorithm{"MD5"};  // Cryptographic hash to be applied to all A instances
		std::string s{"Constructor run"};       // String indicating state in object lifecycle
    };
{% endcodeblock %}

####托管构造####
C++11允许我们托管构造，即在一个构造函数中调用另一个构造函数，实现类状态的初始化，避免了我们之前经常需要自己实现一个Init函数，然后在各个构造函数中调用:
{% codeblock lang:c++ %}
	class X {
		int a;
	public:
		X(int x) { if (0<x && x<=max) a=x; else throw bad_X(x); }
		X() :X{42} { }
		X(string s) :X{lexical_cast<int>(s)} { }
	};
{% endcodeblock %}
C++11还有许多其他的新的特性或者语法，这里只是自己平时用到的，觉得比较实用的，欢迎大家提出自己喜欢的新特性。对于C++11中的lambda语法，我个人不是太喜欢，因为不是特别容易调试:)
