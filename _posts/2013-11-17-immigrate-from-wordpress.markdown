---
layout: post
title: "从Wordpres迁移至Jekyll(@github.io)"
description: ""
category: "Note"
tags: [jekyll]
---
{% include JB/setup %}

用上Wordpress之后，总体感觉还是很不错的，安装、设置、撰写文章都很方便，但仍然有几点不足：

1. 许多功能都需要插件支持，比如代码语法高亮，首页截取文章摘要
2. 图片和文章的存储方式，图片需要单独找图床、或者上传到服务器，而文章又是保存
   mysql数据库中的
3. 数据备份比较麻烦，每次都得使用插件或者人工备份数据

因此决定将博客迁移到github上面，即免费又能使用[Jekyll](http://jekyllrb.com/)这种静态博客，不需要配置DB、使用git发布、markdown语法。简单、方便、
很极客！


## 建立github.io初始环境

首先，你需要一个github帐号，然后使用[JekyllBootstrap](http://jekyllbootstrap.com/)建立一个初始化的环境，参照[教程](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)一步一步来，环境很快就可以搞定。你就可以在USERNAME.github.io上面看到环境了

PS: 教程上面说需要建立的是USERNAME.github.com的repository，需要改为USERNAME.github.io才可以

## 备份Wordpress数据

迁移之前，先备份下数据，如果是使用VPN的话，SSH登录到服务器上，备份数据库
    
    mysqldump -u root -p -h localhost BLOG_TABLE > BLOG_TABLE.sql

另外一些静态文件，比如图片什么的，也备份一份

## 选择迁移工具

迁移工具方面，基本上是基于下面两种方式：

1. 使用WordPress导出的xml文件，导出md文档
2. 连接WordPress的Mysql数据库导出md文档

Jekyll有自带的`jekyll-import`工具，很想使用官方的迁移工具，但我一直“未遂”，说是无法load到`jekyll-import`模块……

另外，Jekyll官网上还推荐了三个第三方的工具：

1. [Exitwp](https://github.com/thomasf/exitwp)
2. [A great
   article](http://vitobotta.com/how-to-migrate-from-wordpress-to-jekyll/) ，
   其实是一篇详细的迁移教程
3. [wpXml2Jekyll](https://github.com/theaob/wpXml2Jekyll)

第二个太过详细，第三个只能运行在Windows下，所以看来只能选择第一个工具了。


## 导出WordPress XML文件

在WordPress的`export.php`页面上可以轻松导出XML文件。但这一步遇到一个小插曲：blog上面太多spam评论了，结果也会一并export出来，这些spam
评论会导致后面工具导出markdown文档失败。因为blog上面真实的评论并不多，所以就考
虑把评论都删除，网上查了资料，只需要将WordPress Mysql表中的`wp_comments`表删除即可

    DROP TABLE wp_comments;

## 使用exitwp

这个工具带有鲜明的“反WordPress”风格，使用起来还是比较简单的，按照项目首页的
README操作即可，你的WordPress文章就变成一个个markdown文档了！

不过使用途中发现有几个问题：

1. 在Mac上使用pip进行依赖安装后，那些依赖的库还是会import失败（是我打开方式不对吗……），还是得自己将一个个依赖下载后安装
2. Exitwp虽然支持下载图片，但是文章中的图片链接却没有换过来
3. 导出后的md文档，对于正文中的特殊字符没有做处理，比如`*`，会导致`jekyll build`失败，需要手动修改
4. 由于系统编码原因，会导致使用exitwp以及jekyll失败，参考这篇[文章](http://www.webplay.pro/linux/set-locale-terminal-settings-mac-os-x.html)
   ，需要在`/etc/profile`添加`export LANG=zh_CN.UTF-8`以及`export
   LC_ALL=zh_CN.UTF-8`


## 发布！
好了，下面将生成的markdown文档以及图片，push到你的github空间即可！Just enjoy it! :)
