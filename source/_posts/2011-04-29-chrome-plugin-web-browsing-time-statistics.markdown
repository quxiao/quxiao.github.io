---
author: admin
comments: true
date: 2011-04-29 14:51:25+00:00
layout: post
slug: chrome-plugin-web-browsing-time-statistics
title: 'Chrome plugin: 网页浏览时间统计'
wordpress_id: 290
categories:
- Other
tags:
- chrome plugin
---

前几天刚经过[实习闹剧](http://www.qxavier.me/2011/04/22/practice-u0026quotfarceu0026quot/)，回到学校，导师暂时还没布置什么任务，比较无聊，于是突发奇想，准备搞个chrome插件玩玩。做个什么功能的插件呢？平时发现许多人（当然也包括我）总会把大把的时间花在一些网站上，做事情的效率变低下不说，还白白浪费了时间、浪费了生命，这样多不好啊！那就做个统计某些网页浏览时间的插件吧。用了三天的晚上空余时间，做了一个雏形。

 

在option页面中，可以添加你想要监控的网站URL，如下图：

 

[![image]({{ site.url }}/images/2011-04-29-chrome-plugin-web-browsing-time-statistics/image_thumb6.png)]({{ site.url }}/images/2011-04-29-chrome-plugin-web-browsing-time-statistics/image_thumb6.png)

 

然后，你就可以向平时一样正常的浏览网页了。当想要查看浏览时间统计的时候，只需点击插件按钮即可。效果如下图：

 

[![image]({{ site.url}}/images/2011-04-29-chrome-plugin-web-browsing-time-statistics/image_thumb7.png)]({{ site.url }}/images/2011-04-29-chrome-plugin-web-browsing-time-statistics/image_thumb7.png)

 

统计信息通过饼图和柱状图展示，显示的是自使用该插件以来，你所花（或者称浪费）在你已监控网站上的时间，以及这些时间占你一生时间（假设你能活100年吧）的比例。

 

Chrome插件开发起来倒还容易，就是javascript+HTML+CSS，熟悉web开发尤其是前端开发的人应该可以很快上手。我是好久没搞web前端开发了，开发时还花了不少时间复习。开发过程中，还用到了[jQuery](http://jquery.com/)和[Highcharts](http://www.highcharts.com/)，节省了不少开发时间啊。

 

其实，我觉得这个插件并没什么技术含量。那为啥要做呢？原因有以下几点：

 

  
  * 我感觉这个点子不错 
   
  * 培养自己的执行力 
   
  * 作为“学术”之外的调剂 
 反正，别让自己闲下来，至少别让自己把大把时间花在没有太多意义的事情上！有些事情，不管你喜欢不喜欢，都是要做的。不过，当你能有自己可以控制的时间，还是多做些更有意义的事情吧！（哎呀，貌似扯远了，最近比较火大 :)）
