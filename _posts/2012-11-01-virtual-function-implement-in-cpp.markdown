---
author: admin
comments: true
date: 2012-11-01 14:47:02+00:00
layout: post
slug: virtual-function-implement-in-cpp
title: C++虚函数的实现
wordpress_id: 658
categories:
- Development
tags:
- c++
- virtual function
- virtual table
---

前段时间公司进行校招，大家那段时间也顺带看看一些面试题目。其中一道经典的题目就是C++中的虚函数是如何实现的？要是直接问我，我还真不记得了。（现在让我再去面试，肯定完蛋）

所谓虚函数，就是函数声明前使用virtual关键字，作用是当基类指针指向子类对象时，通过基类指针调用虚函数，运行的就是实际子类的对应函数，而不是基类的。这种特性在像Java这种“纯”面向对象的语言中，是默认实现的，在C++中就要进行显示声明。如果不显示声明的话，运行的就会是基类的函数。

C++中的虚函数是通过虚表（virtual table）实现的。声明类时，编译器会放一个隐藏的指针，这个指针放在了类的最开始位置，并且指向的就是虚表，虚表里面存放着这个类声明的各个虚函数的地址。假设如果有基类Base：

    
    class Base {
    	virtual void f1();
    	virtual void f2();
    }


那么Base类的虚表的结构就是：

    
    Base
        __vptr    -->    Base::f1(), Base::f2()
        .....（其他成员）


假设还有类D：

    
    class D: public Base {
    	virtual void f1();
    }


那么D的虚表就是下面这个样子的：

    
    D
        __vptr    -->    D::f1(), Base::f2()
        .....


这样的话，当Base类的指针p_base= D()并且p_base->f1()的时候，调用的就是D的f1()了。注意，因为Base和D的虚表的结构一致，所以才能保证调用了正确的函数。

那么，如果是多重继承，即有多个基类时，结果如何呢？C++中，一个类的每一个基类，都有一个单独的虚表与之对应。假设类结构如下：

    
    class B1 {
    	virtual void f1();
    	virtual void f3();
    }
    
    class B2 {
    	virtual void f2();
    	virtual void f3();
    }
    
    class D: public B1, public B2 {
    	virtual void f1();
    	virtual void f2();
    }


那么D类的虚表结构就是：

    
    D
    	__vptr	-->	D::f1(), B1::f3()
    			-->	D::f2(), B2::f3()
    	.....


还有种情况，那就是在多重继承的情况下，子类声明了不属于任何父类的虚函数。例如：

    
    class B1 {
    	virtual void f1();
    	virtual void f3();
    }
    
    class B2 {
    	virtual void f2();
    	virtual void f3();
    }
    
    class D: public B1, public B2 {
    	virtual void f1();
    	virtual void f2();
    	virtual void f4();
    }


子类声明的不属于任何父类的虚函数地址就会继续向第一个虚表的结尾放。

    
    D
    	__vptr	-->	D::f1(), B1::f3(), D::f4()
    			-->	D::f2(), B2::f3()
    	.....


这样的话，不管你用什么类的声明，声明类的虚表结构和实际类的就会是一致的（准确的说应该是父类虚表的结构一定是子类对应的虚表的前缀），这样就实现了基类指针调用子类函数的功能。



参考文章：

[C++ 虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051)

[The virtual table](http://www.learncpp.com/cpp-tutorial/125-the-virtual-table/)

[Busting C++ myths: virtual function calls are slow](http://coldattic.info/shvedsky/s/blogs/a-foo-walks-into-a-bar/posts/3)

--EOF--
