---
author: admin
comments: true
date: 2010-05-31 07:50:37+00:00
layout: post
slug: the-meaning-of-the-linux-directory
title: Linux各目录含义
wordpress_id: 94
categories:
- Essay
---

从网上搜集到的一些资料，自己可以时常翻阅，作为参考。

/bin 基础系统所需要的那些命令位于此目录，也是最小系统所需要的命令，比如 ls、cp、mkdir等命令
/sbin 大多是涉及系统管理的命令的存放，是超级权限用户root的可执行命令存放地（凡是目录sbin中包含的都是root权限才能执行的）
/dev 设备特殊文件
/etc 系统配置文件的所在地，一些服务器的配置文件也在这里；比如用户帐号及密码配置文件
/etc/rc.d 启动的配置文件和脚本
/home 用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示
/lib 标准程序设计库，又叫动态链接共享库，作用类似windows里的.dll文件
/sbin 系统管理命令，这里存放的是系统管理员使用的管理程序
/tmp 公用的临时文件存储点
/root 系统管理员的主目录（呵呵，特权阶级）
/mnt 系统提供这个目录是让用户临时挂载其他的文件系统
/media 即插即用型存储设备的挂载点自动在这个目录下创建，比如USB盘系统自动挂载后，会在这个目录下产生一个目录 ；CDROM/DVD自动挂载后，也会在这个目录中创建一个目录，类似cdrom 的目录。
/lost+found 这个目录平时是空的，系统非正常关机而留下“无家可归”的文件就在这里
/proc 虚拟的目录，是系统内存的映射。可直接访问这个目录来获取系统信息。
/var 某些大文件的溢出区，比方说各种服务的日志文件
/usr 最庞大的目录，要用到的应用程序和文件几乎都在这个目录。其中包含：
/usr/x11r6 存放x window的目录
/usr/bin 众多的应用程序
/usr/sbin 超级用户的一些管理程序
/usr/doc linux文档
/usr/include linux下开发和编译应用程序所需要的头文件
/usr/lib 常用的动态链接库和软件包的配置文件
/usr/man 帮助文档
/usr/src 源代码，linux内核的源代码就放在/usr/src/linux里
/usr/local/bin 本地增加的命令
/usr/local/lib 本地增加的库
/opt 表示的是可选择的意思，有些软件包也会被安装在这里，也就是自定义软件包
