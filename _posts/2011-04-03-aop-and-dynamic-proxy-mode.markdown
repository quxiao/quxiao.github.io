---
author: admin
comments: true
date: 2011-04-03 06:00:53+00:00
layout: post
slug: aop-and-dynamic-proxy-mode
title: AOP以及动态代理模式
wordpress_id: 261
categories:
- Development
tags:
- AOP
- dynamic proxy
---

AOP，即面向方面的编程（面向“切片”的编程应该更合适）。按我的理解，就是对业务逻辑中的某一阶段进行抽取，以达到逻辑模块之前低耦合的编程方法。举个例子，一个系统有许多资源，只有当用户登录系统之后才能访问。那么，验证用户登录就可以抽象成一个“切片”，插在了用户和系统资源之间，编写各个业务模块的人就不用去关心验证用户登录的工作了，验证登录的工作会进行统一配置。

Spring采用的是动态AOP，即通过动态代理模式，在目标对象调用方法前后进行相应处理。

Spring中的动态AOP是基于以下两种方式实现的：

	
  * Java Dynamic Proxy

	
  * CGLIB（code generation library）


Java Dynamic Proxy是面向接口的，而CGLIB是面向类的。两者的关系可以参考这张图：
[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb4.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image4.png)

UserDAOImp是我们需要代理的类，Java动态代理类需要实现了UserDAOImp所实现的接口，（这也要求UserDAOImp必须实现某个接口），而CGLIB则是扩展了UserDAOImp类。

我们先来利用Java Dynamic Proxy实现代理功能。比如目前有一个类RealObject，它实现了接口Interface

Interface

    
    public interface Interface {
    	void funcA ();
    	void funcB (String str);
    	void saveXXX ();
    }


RealObject

    
    public class RealObject implements Interface {
    
    	@Override
    	public void funcA() {
    		// TODO Auto-generated method stub
    		System.out.println("funcA() in RealObject");
    	}
    
    	@Override
    	public void funcB(String str) {
    		// TODO Auto-generated method stub
    		System.out.println("funcB() in RealObject : " + str);
    	}
    
    	@Override
    	public void saveXXX() {
    		// TODO Auto-generated method stub
    		System.out.println("saveXXX() in RealObject");
    	}
    }


假设我们需要代理RealObject，在调用方法前后进行相应处理（输出信息），那么我们就需要一个实现了InvacationHandler接口的类，比如说是ProxyInvacationHandler

ProxyInvacationHandler

    
    public class ProxyInvocationHandler implements InvocationHandler {
    
    	private Object proxied;
    
    	public ProxyInvocationHandler (Object proxied)
    	{
    		this.proxied = proxied;
    	}
    
    	@Override
    	public Object invoke(Object proxy, Method method, Object[] args)
    			throws Throwable {
    		// TODO Auto-generated method stub
    		Object result = null;
    
    		System.out.print("n*******************n");
    		System.out.println(proxy.getClass());	//invoke中第一个参数proxy意义何在？
    		//模拟增加事务处理功能
    		if ( method.getName().startsWith("save") )
    		{
    			System.out.println("transaction begins");
    		}
    		System.out.println("before " + method);
    		//被代理的类调用实际的方法
    		result = method.invoke(proxied, args);
    
    		System.out.println("after " + method);
    		if ( method.getName().startsWith("save") )
    		{
    			System.out.println("transaction ends");
    		}
    		System.out.print("*******************n");
    		return result;
    	}
    
    }


其中的invoke方法就表示当被代理的类调用方法的前后，我们可以做的操作。甚至可以根据方法的不同，选择不同的操作。

最后，再通过Proxy的newProxyInstance静态方法动态生成代理类，因为代理类也是实现Interface接口的，所以操作代理类就像操作被代理类一样。

以下是运行结果：


    funcA() in RealObject
    funcB() in RealObject : 123
    saveXXX() in RealObject

    *******************
    class $Proxy0
    before public abstract void org.quxiao.Interface.funcA()
    funcA() in RealObject
    after public abstract void org.quxiao.Interface.funcA()
    *******************

    *******************
    class $Proxy0
    before public abstract void org.quxiao.Interface.funcB(java.lang.String)
    funcB() in RealObject : 123
    after public abstract void org.quxiao.Interface.funcB(java.lang.String)
    *******************

    *******************
    class $Proxy0
    transaction begins
    before public abstract void org.quxiao.Interface.saveXXX()
    saveXXX() in RealObject
    after public abstract void org.quxiao.Interface.saveXXX()
    transaction ends
    *******************
