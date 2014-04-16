---
layout: post
title: "C++11之Move 构造函数和Move 赋值操作符"
date: 2014-04-13 11:19:04 +0800
comments: true
categories: C_Plus_Plus
---
Move语义是C++11中为了提高性能，减少临时对象的构造和析构而引入的新的语义环境，和Move语义相关的还有右值引用(rvalue reference)，这里我们总结一下，Move语义的引入带来了哪些好处，当我们定义一个类时需要注意什么。

####Move语义及移动构造函数####

我们通过代码来看一下Move语义出现的原因及必要性。

假设我们有如下的类定义，TestClass类中含有一个`Char*`类型的成员变量，为了实现内存的管理，按照`The Big Three`原则，我们需要为这个类定义`拷贝构造函数`，`析构函数`和`赋值操作符`。类的代码如下：
{% codeblock lang:c++ %}
class TestClass
{
public:
	TestClass(char* _array) : array_pointer(_array){
		cout << "Constructor Invoked" << std::endl;
	}
	~TestClass(){
		cout << "Destructor is InVoked" << std::endl;
		delete array_pointer;
	}
	TestClass(const TestClass& rhs){
		cout << "Copy Constructor is Invoked" << std::endl;
		size_t len = strlen(rhs.array_pointer) + 1;
		this->array_pointer = new char[len];
		memcpy(this->array_pointer, rhs.array_pointer, len);
	}

private:
	char* array_pointer;
};
{% endcodeblock %}
其中`拷贝构造函数`的语义为：当我们采用一个对象A去构造新的对象时，拷贝构造函数会被自动调用。拷贝构造函数的参数为`const TestClass& rhs`决定了我们无法改变rhs的值。在下列情况下会使用拷贝构造函数：
{% codeblock lang:c++ %}
TestClass A(x);
TestClass B(x+y);
{% endcodeblock %}
对于对象A的构造，拷贝构造函数完全符合我们的期望，因为我们使用了一个	`lvalue`来构造A，我们不希望改变`x`的内容，以便以后继续保持对`x`的引用。但是对于对象B的构造过程，拷贝构造函数明显做了多余的事情：
>	我们采用`rvalue`即一个临时变量来构造B，在B构造之后，临时变量会被销毁，我们不需要保持对这个临时变量的引用。因此，我们完全可以直接将构造函数中参数`rhs`的`array_pointer`指针直接给B，这样一来就可以省下B中`array_pointer`的初始化以及临时变量中`array_pointer`的释放过程。

我们通过代码来进一步理解：
{% codeblock lang:c++%}
		this->array_pointer = rhs.array_pointer;//directly assign pointer in rhs to the new object
		rhs.array_pointer = nullptr;//do not let rhs refer to the raw pointer any more
{% endcodeblock %}
以上代码就是我们所说的`Move Semantic`，当使用`rvalue`去构造或者给一个对象赋值时,我们可以直接将`rvalue`中的指针直接转移给新的对象，从而节省新的内存分配等的开销,由于`rvalue`在新的对象构造之后会被自动释放，所以我们没有必要保留对它的引用。

在C++11之前我们无法将上面的代码块写入构造函数中，因为我们无法区分`rhs`是不是`rvalue`。C++11中引入了右值引用，允许我们实现针对右值引用的函数重载（只在参数类型 指定右值即可），代码如下：
{% codeblock lang:c++ %}
	//This is move consturctor, transfer ownership
	TestClass(TestClass&& rhs){//notice there are two '&' symbols
		cout << "Move Construct is Invoked" << std::endl;
		this->array_pointer = rhs.array_pointer; 
		rhs.array_pointer = nullptr;
	}
{% endcodeblock%}
上面就是移动构造函数（Move Constructor）,函数参数为`TestClass&&`，表明函数接受一个`rvalue`。

实现移动构造函数应注意两点:

*	参数类型为TestClass&&
*	需要取消rhs中指针对原地址的引用（防止在rhs析构时导致原变量被释放）

####移动赋值运算符####
了解了移动构造函数，很自然的想到在对对象进行赋值运算时同样存在`rvalue`的问题
{% codeblock lang:c++ %}
A = x;
B = x + y;
{% endcodeblock %}
同样我们可以重载移动赋值操作符如下：
{% codeblock lang:c++ %}
	//move assignment
	TestClass& operator=(TestClass&& rhs){//(1)
		cout << "Assign operator and Transfer ownership" << std::endl;
		std::swap(this->array_pointer, rhs.array_pointer);
		return *this;
	}
	//assignment
	TestClass& operator=(TestClass& rhs){//(2)
		cout << "Assign operator" << std::endl;
		TestClass tmp(rhs);
		std::swap(this->array_pointer, tmp.array_pointer);
		return *this;
	}
{% endcodeblock %}
现在我们重载了两个`operator=`操作符，当我们对`A`赋值时,(2)会被调用，当我们对`B`赋值时，(1)会被调用。

在c++11中你可以采用上述做法，重载两个对应复制操作符的函数，也可以使用下面更简洁的方法(unifying assignment operator)：
{% codeblock lang:c++ %}
	TestClass& operator=(TestClass rhs){
		cout << "Assign operator and Transfer ownership" << std::endl;
		std::swap(this->array_pointer, rhs.array_pointer);
		return *this;
	}
{% endcodeblock %}
我们把函数的参数改成按值传递的形式（），这样做的好处有：
*	如果我们是以一个`lvalue`对对象进行赋值，那么系统会自动构建一个对象的拷贝作为函数的参数（因为是按值传递的）
*	如果我们是以一个`rvalue`对对象进行赋值，那么这个函数的表现会像（1）一样。

关于赋值操作符的重载可以参考[这篇文章](http://en.wikibooks.org/wiki/More_C++_Idioms/Copy-and-swap)。

这里有一个[demo project](https://github.com/Charlesjean/C11TestProjects),有兴趣的话可以下来看一下代码。

