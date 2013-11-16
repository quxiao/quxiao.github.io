---
author: admin
comments: true
date: 2012-05-13 07:54:10+00:00
layout: post
slug: add-startup-programe-with-expect
title: 利用Expect添加开机启动程序
wordpress_id: 557
categories:
- Development
tags:
- expect
---

都说“懒”是程序员的美德，如果“懒”是指想尽一切办法让计算机来完成重复的事情，那么这句话完全正确！

最近遇到一个问题，让我忍不住想要“懒”一下，情况是这样的：

5、6年前买的笔记本，我现在依然在使用，不过它实在太老，已经沦为“上网本”，我只能在上面装上Ubuntu+Chrome，平时上上网。但是，每次开机，我都要重复的做3件事：



	
  1. 开启ssh，为了能看看墙外的世界

	
  2. 打开含有ssh账户密码的文本文件

	
  3. 打开Chrome


这实在是在麻烦了！我得想个办法，让计算机能自动完成这些工作。关键的问题在于如何在ssh提示你输入密码的时候，能够自动输入，而不用我把密码贴过去。终于，我找到了一个在linux下可以和其他程序“talk”的程序——Expect。

Expect有3个关键的命令：**expect**、**send**、**spawn**

expect——期待程序给我某个信息

send——Expect给程序发送某个信息

spawn——启动某程序

那么，目前我需要Expect的“逻辑”是这样的：



	
  1. 利用spawn启动ssh

	
  2. 期待（expect）ssh程序提示输入密码

	
  3. 把密码send给ssh程序


如果逻辑不复杂的话，Expect的语法还是相对简单的，可以参考《[Exploring Expect](http://oreilly.com/catalog/expect/chapter/ch03.html)》和[man expect](http://linux.die.net/man/1/expect)。

Expect的代码如下：

    
    #!/usr/bin/expect --
    
    #一直等待程序的输入
    set timeout -1
    
    #设置变量
    set username blablabla
    set hostname blablabla.com
    set password blablabla
    
    #不知为何利用-f -n将ssh转入后台就无法成功运行，待解决
    #spawn ssh -qTfnN -D 7070 $username@$hostname
    #启动ssh程序
    spawn ssh -qTN -D 7070 $username@$hostname
    #匹配ssh程序的输出，如果含有“password”，就将密码send给ssh
    expect "password" {send "$password\r"}
    
    #之后将ssh控制权转交给用户
    interact



应该还是比较通俗易懂的吧。

好，现在Expect的工作完成了， 在再之前加上启动gnome-terminal和用bash启动Expect，然后最后加上自启动Chrome，这样就大功告成了！之前寻找各种添加开机启动脚本的方法，都没能成功（怀疑和启动时还未进行gnome桌面环境有关），最后只能通过Ubuntu的"System"->"Preferences"->"Startup Applications"来添加启动命令，分别添加一下两条命令：

    
    #开启gnome模拟终端
    gnome-terminal -e "bash -l -c '~/test_expect.sh'" &
    #开启Chrome浏览器
    /opt/google/chrome/google-chrome



好了，这样我就可以每次开机让计算机替我完成之前重复的工作了，not perfect, but practical !

参考资料：



	
  * 《[Exploring Expect](http://oreilly.com/catalog/expect/chapter/ch03.html)》

	
  * [man expect](http://linux.die.net/man/1/expect)

	
  * [开机启动时快速进入开发环境](http://pastebin.com/Y48pM73C)

	
  * [用于shell脚本无交互的ssh自动登陆](http://zhu8337797.blog.163.com/blog/static/1706175492011101282720246/) （这篇文章出现在许多地方，不知道原出处在哪……）


-EOF-
