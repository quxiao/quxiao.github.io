---
layout: post
title: "小试Travis-Ci"
date: 2014-02-24 07:36:20 +0800
comments: true
categories: 
---

前段时间，好友小新给我看了一个工具（或者说是一种服务）——**travis-ci**，它提供了对于Github的项目上的持续集成服务（私有项目是要收费的）。正好最近把一个大学时写的小程序从Google Code迁移到Github上，就拿他来做个实验吧！

## .travis.yml

查看travis-ci的官方文档，其实十分简单。如果需要在项目中使用travis-ci提供的服务，只需要在Repository中添加`.travis.yml`配置文件。进行持续集成嘛，配置文件里面无非就是这么几项吧：

* 使用的编程语言
* 编译器及其版本
* 编译命令
* 跑测试用例的命令
* etc.

是的，不过travis-ci提供更细粒度的操作。当你每次push时，它进行如下操作：

1. `clone`你的repository并`cd`进去
2. 执行`before_install`操作
3. 执行`install`操作
4. 执行`before_script`操作
5. 执行`script`操作
6. 根据是否成功，执行`after_success`和`after_failure`操作
7. 执行`after_script`操作

其中除了步骤1，其它操作的命令都需要在`.travis.yml`中进行配置，具体可以看[doc](http://docs.travis-ci.com/user/build-configuration/) 。其实一般比较简单、独立的项目，并不会用到里面的每一个操作，有时候只需配置`script`就行了。比如我的简单程序，我只需要看是否能编译通过以及跑过测试用例，我就把这些操作写在`build_and_test.sh`里面，然后这么写：

    language: cpp

    script: 
        - ./build_and_test.sh

## 开启服务

配置文件编写好之后，你还需要登录<http://travis-ci.org>来开启对于某个Repository的持续集成服务。

{% img center /images/2014-02-24/20140224-2.png %}


## Push

万事俱备，就差你push来触发服务了！每次你进行push操作，都会在travis-ci的虚拟环境下进行build操作（除非你的comment里面写着`[ci skip]`）。你可以自己看到每次build的结果。

{% img center /images/2014-02-24/20140224-1.png %}

BTW，travis-ci还会为最近一次的build结果生成图片链接（点击上面的`build xxx`小图标即可），你可以把它放到Repository中的README中，这样在Github上面就能随时看到build的结果了。
