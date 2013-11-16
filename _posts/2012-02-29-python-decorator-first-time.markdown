---
author: admin
comments: true
date: 2012-02-29 09:57:15+00:00
layout: post
slug: python-decorator-first-time
title: Python Decorator初体验
wordpress_id: 529
categories:
- Development
- Note
tags:
- decorator
- python
---

前段时间在看一些有关python web framwork的时候，发现在python语言里竟然有“@”符号，一查资料，原来是python装饰器（python装饰器也可以通过除了“@”的其他语法进行定义）。装饰器，是一种设计模式，用于动态地给对象添加行为，之前的一篇[文章](http://www.qxavier.me/2011/04/08/decorator_pattern/)也提到过。python中也有装饰器，不过和普遍意义上的装饰器不同，python中的装饰器实际上是一种“[语法糖](http://en.wikipedia.org/wiki/Syntactic_sugar)”，是一种语句的简便写法。比如，`a[idx]`就是`*(a+idx)`的一种简便写法，也算是一种语法糖。 假设有如下写法：

    
    @dec
    def func():
      pass


它等同于：



    
    func = dec(func)


dec也是一个函数，只不过这个函数比较特殊，它的参数是一个函数（就是原先被装饰的函数func），它的返回值是一个函数。这样，再运行func()时，就是运行经过“装饰”的函数了。 python decorator可以帮助我们轻松地为函数或者类添加行为，而不用像普通的装饰器模式那样基于某个接口，大概这也算是动态语言的优势之一吧。 好，来看一些例子。

**面向切面的编程**

当我们要对许多函数进行相同的测试或者进行其他处理的时候，如果在每个函数都写一遍的话，代码太过重复，不利于统一管理。我们可以写一些统一的函数，然后在需要进行处理的函数前面加上decorator就行了。

    
    def before(f):
    	def wrapper():
    		print 'before function'
    		f()
    	return wrapper
    
    def after(f):
    	def wrapper():
    		f()
    		print 'after function'
    	return wrapper
    
    @before
    @after
    def func():
    	print 'this is function'
    
    if __name__ == '__main__':
    	func()


这样，就可以在函数前后进行相应的处理了。程序输出如下：


    before function

    this is function

    after function


**Singleton模式**

    
    def singleton(cls):
    	instances = {}
    	def wrapper():
    		if cls not in instances:
    			instances[cls] = cls()
    		return instances[cls]
    	return wrapper
    
    @singleton
    class MyClass:
    	def __init__(self):
    		self.num = 0
    
    if __name__ == '__main__':
    	c1 = MyClass()
    	print c1.num
    	c2 = MyClass()
    	c2.num = 1
    	print c1.num
    	print (c1 == c2)


这样，每次“新建”MyClass类型，得到的都会是同一个实例。程序输出如下：


    0

    1

    True


**检验函数参数和返回值的类型**

    
    def accepts(*types):
    	def check_args(f):
            #若去掉该断言，则代码可正常运行
    		assert len(types) == f.func_code.co_argcount, 'type len: %d args len: %d' %(len(types), f.func_code.co_argcount)
    		def new_f(*args, **kwds):
    			for (a, t) in zip(args, types):
    				assert isinstance(a, t), 'arg %r does not match %s' % (a, t)
    			return f(*args, **kwds)
    		new_f.func_name = f.func_name	# why?
    		return new_f
    	return check_args
    
    def returns(rtype):
    	def check_ret(f):
    		def new_f(*args, **kwds):
    			ret = f(*args, **kwds)
    			assert isinstance(ret, rtype), 'return value %r does not match %s' %(ret, rtype)
    			return ret
    		new_f.func_name = f.func_name
    		return new_f
    	return check_ret
    
    @returns((int, float))
    @accepts(int, (int, float))
    def func(arg1, arg2):
    	return arg1 + arg2
    
    if __name__ == '__main__':
    	print func(1, 2.0)
    	print func('1', '2')


在这段代码中，accepts函数用来保证被装饰的函数有两个参数，第一个参数为int型，第二个参数为int或float型，returns函数保证返回值为int或float型。那么，运行func(1, 2.0)，assert就能通过，而func('1', '2')就不能通过。

Todo：这里还有一个问题，就是当@returns和@accepts语句交换顺序之后，accepts中检测函数参数个数的assert就无法通过，输出参数个数为0，还不知道是什么原因，待解决。

若去掉check_args函数中的对于f的参数个数的assert判断，则returns和accepts两个decorator无论什么顺序，代码均可正常运行。是由于`new_f(*args, **kwds)`改变了实际传入的参数的个数？

参考资料：



	
  * [PEP 318 -- Decorators for Functions and Methods](http://www.python.org/dev/peps/pep-0318/)

	
  * [PythonDecorators](http://wiki.python.org/moin/PythonDecorators?action=fullsearch&context=180&value=linkto%3A%22PythonDecorators%22)

	
  * [Syntactic sugar](http://en.wikipedia.org/wiki/Syntactic_sugar)

	
  * [Decorator pattern](http://en.wikipedia.org/wiki/Decorator_pattern)


--EOF--
