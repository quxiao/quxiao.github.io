---
author: admin
comments: true
date: 2011-04-08 11:03:06+00:00
layout: post
slug: decorator_pattern
title: Decorator模式初探
wordpress_id: 271
categories:
- Development
tags:
- decorator pattern
- io
- java
---

Decorator模式，又称装饰模式。在实际编程中，如果希望能在原有基础上动态的添加新的功能或者扩展已有功能，那么Decorator模式提供了一种很好的解决方案。Decorator模式如下图所示：

[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb5.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image5.png)

具体的实现类ConcreteComponent和装饰类Decorator均实现了Component接口，装饰类含有一个Component类作为其成员。这样，当需要在ConcreteComponent基础上添加新的功能时，只需扩展Decorator形成具有某一特定功能的Decorator即可。由于不同的Decorator都是遵循Component接口，当需要扩展新功能时，在原有的Decorator上再套一层Decorator就可以啦。

来用一个简单的例子解释一下吧。比如现在有个手机类SimplePhone，实现了Phone接口：

    
    public interface Phone {
    	String getDescription();
    	int getCost();
    }
    
    package org.quxiao.phone;
    
    public class SimplePhone implements Phone {
    
    	@Override
    	public String getDescription() {
    		return "simple phone";
    	}
    
    	@Override
    	public int getCost() {
    		return 100;
    	}
    
    }


为了能动态增加功能，做一个抽象的Decorator：

    
    public abstract class AbstractPhoneDecorator implements Phone {
    	protected Phone phone;
    
    	public AbstractPhoneDecorator(Phone phone)
    	{
    		this.phone = phone;
    	}
    }


现在需要给手机添加GPS模块，扩展抽象的Decorator：

    
    public class GPSPhoneDecorator extends AbstractPhoneDecorator {
    
    	public GPSPhoneDecorator(Phone phone) {
    		super(phone);
    	}
    
    	@Override
    	public String getDescription() {
    		// TODO Auto-generated method stub
    		return phone.getDescription() + " and GPS";
    	}
    
    	@Override
    	public int getCost() {
    		// TODO Auto-generated method stub
    		return phone.getCost() + 100;
    	}
    
    }


同理，再添加蓝牙的模块：

    
    public class BlueToothPhoneDecorator extends AbstractPhoneDecorator {
    
    	public BlueToothPhoneDecorator(Phone phone) {
    		super(phone);
    	}
    
    	@Override
    	public String getDescription() {
    		return phone.getDescription() + " and blue tooth";
    	}
    
    	@Override
    	public int getCost() {
    		return phone.getCost() + 200;
    	}
    
    }


这样，我们就可以给SimplePhone类动态的添加模块了

    
    public class Main {
    
    	public static void main(String[] args) {
    		Phone simplePhone = new SimplePhone();
    		Phone GPSPhone = new GPSPhoneDecorator(simplePhone);
    		Phone myPhone = new GPSPhoneDecorator(new BlueToothPhoneDecorator(new SimplePhone()));
    
    		System.out.println("simplePhone description : " + simplePhone.getDescription());
    		System.out.println("simplePhone cost : " + simplePhone.getCost());
    		System.out.println("GPSPhone description : " + GPSPhone.getDescription());
    		System.out.println("GPSPhone cost : " + GPSPhone.getCost());
    		System.out.println("myPhone description : " + myPhone.getDescription());
    		System.out.println("myPhone cost : " + myPhone.getCost());
    	}
    }





> simplePhone description : simple phone
simplePhone cost : 100
GPSPhone description : simple phone and GPS
GPSPhone cost : 200
myPhone description : simple phone and blue tooth and GPS
myPhone cost : 400


其实，Java I/O库中就是用了Decorator模式。

在基于Stream的输入中，所有数据都抽象成InputStream类，其子类FilterInputStream就相当于InputStream的装饰类。[BufferedInputStream](http://download.oracle.com/javase/6/docs/api/java/io/BufferedInputStream.html), [CheckedInputStream](http://download.oracle.com/javase/6/docs/api/java/util/zip/CheckedInputStream.html), [CipherInputStream](http://download.oracle.com/javase/6/docs/api/javax/crypto/CipherInputStream.html), [DataInputStream](http://download.oracle.com/javase/6/docs/api/java/io/DataInputStream.html), [DeflaterInputStream](http://download.oracle.com/javase/6/docs/api/java/util/zip/DeflaterInputStream.html), [DigestInputStream](http://download.oracle.com/javase/6/docs/api/java/security/DigestInputStream.html),[InflaterInputStream](http://download.oracle.com/javase/6/docs/api/java/util/zip/InflaterInputStream.html), [LineNumberInputStream](http://download.oracle.com/javase/6/docs/api/java/io/LineNumberInputStream.html), [ProgressMonitorInputStream](http://download.oracle.com/javase/6/docs/api/javax/swing/ProgressMonitorInputStream.html), [PushbackInputStream](http://download.oracle.com/javase/6/docs/api/java/io/PushbackInputStream.html)都扩展自FilterInputStream，每个都对应于一个特定的功能。比如哦BufferedInputStream具有缓存功能，DataInputStream可将读出基本（primitive）类型的数据。因此，当我们使用Java I/O库进行输入输出时，总会生成一堆的类，就像这样：

    
    DataInputStream dis = new DataInputStream( new BufferedInputStream( new FileInputStream(FILE_LOC)));


之前一直不明白为什么Java的输入输出会这么麻烦，原来是因为用到了Decorator模式。现在明白了其中的原理，也觉得不那么复杂了。
