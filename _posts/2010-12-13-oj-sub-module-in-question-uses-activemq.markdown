---
author: admin
comments: true
date: 2010-12-13 04:04:41+00:00
layout: post
slug: oj-sub-module-in-question-uses-activemq
title: 在OJ判题模块中使用ActiveMQ
wordpress_id: 125
categories:
- Tool
tags:
- activemq
---

这两天在考虑如何在OJ中实现分布式判题，之前想到了JMS，但觉得不是很好入门。随后发现了[ActiveMQ](http://activemq.apache.org/)这个东西，其实就是在JMS上面有包装了一层，但是使用起来很方便。于是就做了一个样例模拟程序，以验证其可行性和效率。下图是样例程序大致思想：

 

[![image](http://www.qxavier.me/wp-content/uploads/2010/12/image_thumb.png)](http://www.qxavier.me/wp-content/uploads/2010/12/image.png)

 

 

只要开启ActiveMQ的服务，Server和Client再做一些简单的配置，就可以共享ActiveMQ中的某一消息队列（比如图中的pending-queue）。Server程序无需管理Client Node，相应工作是由ActiveMQ来完成的。queue中的消息是具有事务性的，所以Client既不会获取重复的消息，也不会漏读消息（至少按目前的观察是这样的）。这样，就实现了Server和Client之间的异步消息传递，达到分布式的效果。

 

样例程序的Server每隔1秒发送一个带有id的消息到pending queue；多个Client端从queue中读取消息，接着sleep一段时间。下图是样例程序的运行效果：

 

[![activemq_sample_screen](http://www.qxavier.me/wp-content/uploads/2010/12/activemq_sample_screen_thumb.jpg)](http://www.qxavier.me/wp-content/uploads/2010/12/activemq_sample_screen.jpg)
