---
author: admin
comments: true
date: 2011-04-02 09:48:09+00:00
layout: post
slug: webx3-sample-project-login-webx3-tutorial-study-notes-a
title: Webx3样例工程login-webx3-tutorial学习笔记（一）
wordpress_id: 258
categories:
- Essay
tags:
- webx
---

在eclipse中部署好工程，然后再通过jetty启动server，工程的功能相当简单，就是将表单中的字符串提交后显示。通过运行和调试，初步了解了webx的运行流程。

 

首先看一下WEB-INF中的配置文件

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image.png)

 

web.xml是J2EE工程必须的配置文件，webx.xml以及common中的xml都是webx框架公用的配置文件，而app1文件夹以及webx-app1.xml就是名为"app1"的模块配置文件。

 

配置文件的详细设置我们稍后再谈，我们直接通过运行实例来讲解流程吧。

 

打开首页，内容是通过Velocity生成的：

 

index.vm

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb1.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image1.png)

 

提交表单之后，即会运行com.alibaba.webx.tutorial.app1.module.action.SimpleAction中的doGreeting方法。那为什么会跳转到这边呢？其实都是写在配置文件中的。

 

来看看webx-app1.xml

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb2.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image2.png)

 

 

 

它指定了app1模块所在的包。再看index.vm，“simple_action”表示查找SimpleAction类，“event_submit_do_greeting”表示执行SimlieAction类中的doGreeting方法。（name是否一定设为“action”我还不确定）

 

试着修改一下，我们将index.vm中的simple_action修改为my_action、event_submit_do_greeting改为event_submit_do_my_greeting，再在com.alibaba.webx.tutorial.app1.module.action包中加入MyAction类，并添加doMyGreeting方法。重启服务器，就可看到效果。输入111，结果就会变成my111。

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/04/image_thumb3.png)](http://www.qxavier.me/wp-content/uploads/2011/04/image3.png)

 

好，基本知道了解流程了，但理解得还是很凌乱，尤其是配置文件加载以及动态定位到某个类的原理。
