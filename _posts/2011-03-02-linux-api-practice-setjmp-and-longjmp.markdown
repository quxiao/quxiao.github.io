---
author: admin
comments: true
date: 2011-03-02 13:09:09+00:00
layout: post
slug: linux-api-practice-setjmp-and-longjmp
title: Linux API 实践：setjmp和longjmp
wordpress_id: 213
categories:
- Development
tags:
- linux
- longjmp
- setjmp
---

我们在平时编程的过程中，编写异常处理的代码是常有的事。但有时，我们的函数调用的比较“深”，同时又想根据较深处的函数的异常处理，使较高层次的调用函数作出相应动作。举个例子吧，main()调用a()，a()调用b()，b()调用c()，如果c()中进行异常处理，要在main()中显示错误信息。我们一般的做法应该都是根据函数返回值将底层的错误返回给调用者，以此类推，直至main()。示意代码如下：

    
    #include <stdio.h>
    
    int a(int);		//-1 means error
    int b(int);		//-1 means error
    int c(int);		//-1 means error
    
    int main ()
    {
    	int i;
    	while ( scanf("%d", &i) != EOF )
    	{
    		if ( a(i) == -1 )
    			printf("error!\n");
    	}
    }
    
    int a (int n)
    {
    	//blablabla
    	if ( b(n) == -1 )
    		return -1;
    }
    
    int b (int n)
    {
    	//blablabla
    	if ( c(n) == -1 )
                    return -1;
    }
    
    int c (int n)
    {
    	//blablabla
    	if ( n < 0 )
    		return -1;
    }


可以看到，a, b, c三个函数都需返回错误信息，有时a和b本来并不需要进行异常处理，而因为c要将异常信息传给main，a和b不得不加入更多的代码。

但是，当你知道有setjmp和longjmp这两个函数后，处理起来就会简单一些。它们可以进行跨函数的跳转，其内在的动作是直接改变程序调用栈的栈顶位置，而不用根据函数调用关系一层一层的返回。我们再来看看经过改进的代码：

    
    #include <setjmp.h>
    #include <stdio.h>
    
    int a(int);
    int b(int);
    int c(int);
    
    jmp_buf jmpbuffer;
    
    int main ()
    {
    	int i;
    	if ( setjmp(jmpbuffer) )
    		printf("error!\n");
    	while ( scanf("%d", &i) != EOF )
    	{
    		a(i);
    	}
    }
    
    int a (int n)
    {
    	//blablabla
    	b(n);
    }
    
    int b (int n)
    {
    	//blablabla
    	c(n);
    }
    
    int c (int n)
    {
    	//blablabla
    	if ( n < 0 )
    		longjmp(jmpbuffer, 1);
    }


在完成功能的情况下，少写了一些代码，结构也清晰了一些。

运行结果如下：


> quxiao@quxiao-laptop:~/learnLinux/setjmp$ ./setjmp_ex.o
0
1
2
3
-1
error!
-2
error!
-3
error!


不过，对于这种加强版的"goto"，我还是觉得应该慎用。
