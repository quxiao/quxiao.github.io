---
author: admin
comments: true
date: 2011-04-05 09:30:52+00:00
layout: post
slug: cglib-dynamic-proxy-mode
title: CGLib动态代理模式
wordpress_id: 264
categories:
- Development
tags:
- cglib
- java
---

CGLib，即code generation library，原理是通过动态生成类以实现代理的功能。[上一篇](http://www.qxavier.me/2011/04/03/aop-and-dynamic-proxy-mode/)文章中，我们提到了AOP（面向切片编程）以及AOP的一种实现方法——Java Dynamic Proxy。需要注意的是，Java动态代理是面向接口的，即被代理的类必须实现某个接口，代理类以该接口的形式出现，而使用CGLib，则没有这方面的限制，任意一个类都是可以的。

简单的说，使用CGLib代理某个类，需要在Enhancer对象中设置好基类（也就是被代理类），以及一系列回调函数Callback。Callback是一个接口，CGLib提供了6个它的子接口：


**Callback子接口**


**用途（有待确认）**

[Dispatcher](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/Dispatcher.html)


分发给其他Callback


[FixedValue](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/FixedValue.html)


仅仅返回被代理类方法的返回值，对于限定某一些特定方法很有用（因为返回值必须和被代理类方法的返回值类型相匹配）






[InvocationHandler](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/InvocationHandler.html)


主要用于Proxy（替代Java动态代理），也可以用户Enhancer






[LazyLoader](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/LazyLoader.html)


与Dispatcher类似，当代理类的第一个lazily-load方法调用时才会被调用






[MethodInterceptor](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)


普通用途的回调方法，在处理逻辑（advice）前后进行处理






[NoOp](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/NoOp.html)


直接调用基类（被代理类）的方法调用




好，那我们来假设一个场景吧。

有这样一个类RealObject，它可以查询、保存资源，比如是这样：

    
    public class RealObject {
    	public void queryA ()
    	{
    		System.out.println("queryA");
    	}
    	public void queryB ()
    	{
    		System.out.println("queryB");
    	}
    	public void saveA ()
    	{
    		System.out.println("saveA");
    	}
    	public void saveB ()
    	{
    		System.out.println("saveB");
    	}
    }


我们有这样的需求：保存资源时，需要加入事务功能。那么可以实现自己的MethodInterceptor，实现其中的intercept方法：

    
    public class MyMethodInterceptor implements MethodInterceptor {
    
    	@Override
    	public Object intercept(Object obj, Method method, Object[] args,
    			MethodProxy proxy) throws Throwable {
    		// TODO Auto-generated method stub
    		System.out.println(obj.getClass());
    		//模拟事务功能
    		if ( method.getName().startsWith("save") )
    		{
    			System.out.println("begin transaction");
    		}
    		Object result = proxy.invokeSuper(obj, args);
    		if ( method.getName().startsWith("save") )
    		{
    			System.out.println("end transaction");
    		}
    		System.out.println("************************\n");
    		return result;
    	}
    }


再来写主入口：

    
    public class Main {
    
    	/**
    	 * @param args
    	 */
    	public static void main(String[] args) {
    		// TODO Auto-generated method stub
    		Enhancer enhancer = new Enhancer();
    		//将需要代理的类作为基类
    		enhancer.setSuperclass(RealObject.class);
    		//设置回调功能，这里使用的是拦截器，
    		//当被代理类调用方法时，会执行拦截器的intercept方法
    		enhancer.setCallback(new MyMethodInterceptor());
    		RealObject realObj = (RealObject) enhancer.create();
    		consume(realObj);
    	}
    
    	private static void consume(RealObject realObj)
    	{
    		realObj.queryA();
    		realObj.queryB();
    		realObj.saveA();
    		realObj.saveB();
    	}
    }


运行一下，看看效果吧！


    class org.quxiao.RealObject$$EnhancerByCGLIB$$91eddf0b
    queryA
    ************************

    class org.quxiao.RealObject$$EnhancerByCGLIB$$91eddf0b
    queryB
    ************************

    class org.quxiao.RealObject$$EnhancerByCGLIB$$91eddf0b
    begin transaction
    saveA
    end transaction
    ************************

    class org.quxiao.RealObject$$EnhancerByCGLIB$$91eddf0b
    begin transaction
    saveB
    end transaction
    ************************


我们可以看到，CGLib动态生成了一个叫RealObject$$EnhancerByCGLIB$$91eddf0b的类，这个类就是RealObject的代理类。以后有时间来反编译一下，研究研究CGLib动态生成类的原理。
