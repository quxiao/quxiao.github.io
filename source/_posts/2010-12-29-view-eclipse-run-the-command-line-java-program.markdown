---
author: admin
comments: true
date: 2010-12-29 06:45:48+00:00
layout: post
slug: view-eclipse-run-the-command-line-java-program
title: 查看Eclipse运行Java程序的命令行
wordpress_id: 127
tags:
- eclipse
- java
---

这些天一直试图用命令行运行Java程序，结果一直说找不到jar包中的类，改来改去，很是麻烦。在网上查找方法时，无意间在[这里](http://http://pire-cao.spaces.live.com/blog/cns!C2130CDC4173DBDA!1584.entry)发现了一种途径，既然Java程序在Eclipse里面是可以正常运行的，那就看看Eclipse运行Java程序的命令到底是什么。

 

只要在Eclipse里面Debug相应Java程序，在Debug视图中查看主线程的Properties，就可以看到命令行了！
