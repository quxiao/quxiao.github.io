---
author: admin
comments: true
date: 2013-10-27 16:29:06+00:00
layout: post
slug: effective-go-learning-note
title: 《Effective Go》学习笔记
wordpress_id: 791
categories:
- Note
tags:
- go
---

最近抽时间把Go语言看了一下，把Go playground玩了一遍，把官方文档上的Effective Go也学习了一番。把一些重点或者说自己觉得比较有特点的地方记录下来，用作备忘和分享。


# 未使用的package或者变量是编译错误


这相当于就在代码层面对开发人员进行了规范，不像C++等其它语言，你多include一个.h文件，或者声明了一个变量但没有使用，这对你编译程序都不会有任何影响，挺多会打些编译warning。这样很容易造成程序编译了没有使用的库和变量，使程序变得臃肿。


# **变量的可见性有变量名决定**


一个package中的变量名如果首字母是大写，则对于package外部是可见的，反之则不可见。Go里面没有public, private这样的关键字（至少我目前没看到），直接约定首字母大写的变量是public的，不是大写的是private的，简便并且规范。


# 函数可以返回多值


像Python一样，Go的函数可以返回多个值，这样就很方便的把正常情况下需要返回的数据以及发生错误时error一并返回了。


# 关键字defer——return之前执行


Go里面可以将语句前声明defer关键字，表示目前先不执行这条语句，等待函数return前再执行。这实在是太方便了！特别是对于一些需要管理资源的场景。

    
    // Contents returns the file's contents as a string.
    func Contents(filename string) (string, error) {
        f, err := os.Open(filename)
        if err != nil {
            return "", err
        }
        defer f.Close()  // f.Close will run when we're finished.
    
        var result []byte
        buf := make([]byte, 100)
        for {
            n, err := f.Read(buf[0:])
            result = append(result, buf[0:n]...) // append is discussed later.
            if err != nil {
                if err == io.EOF {
                    break
                }
                return "", err  // f will be closed if we return here.
            }
        }
        return string(result), nil // f will be closed if we return here.
    }


试想一下，在C++中，如果想要在每个分支都能释放资源，就得采用1）`goto`或`do-while`，例如以下代码，或者 2）`scoped pointer`。

    
    int f1(const char* fname) {
        int ret = 0;
        FILE* fp = fopen(fname, "r");
        if ( NULL == fp ) {
            return -1;
        }
        if (....) {
            ret = 1;
            goto FINISH;
        } else if (....) {
            ret = 2;
            goto FINISH;
        }
    
    FINISH:
        fclose(fp);
        return ret;
    }
    
    int f2(const char* fname) {
        int ret = 0;
        FILE* fp = fopen(fname, "r");
        if ( NULL == fp ) {
            return -1;
        }
        do {
            if (....) {
                ret = 1;
                break;
            } else if (....) {
                ret = 2;
                break;
            }
        } while(0);
    
        fclose(fp);
        return ret;
    }




# new & make


Go语言中的new和C++中的不太一样，它只负责分配一段全为`'\0'`的内存，不会进行任何其它初始化，需要你自己再做一些工作。
而make关键字是用来新建slice, map, channel这三中类型的，因为它们内部必须做一些特殊的初始化才能使用。


# 结构体可以直接print


默认情况下，struct就可以直接打印出来，而且还会打印出每个字段的名称和值。这个对于开发人员排查问题，也是极其方便的。

    
    type T struct {
        a int
        b float64
        c string
    }
    t := &T{ 7, -2.35, "abc\tdef" }
    fmt.Printf("%v\n", t)
    fmt.Printf("%+v\n", t)
    fmt.Printf("%#v\n", t)


会打印出

    
    &{7 -2.35 abc   def}
    &{a:7 b:-2.35 c:abc     def}
    &main.T{a:7, b:-2.35, c:"abc\tdef"}


当然，你也可以定义某个类型T的**String()**方法，用来改变输出的字符串，就像Python中实现`__str__`方法一样。


# 接口“嵌入”


如果想使用某套接口，在C++中一般都是将满足这套接口的实例作为某个类的成员，再调用这个成员的接口。在Go中，可以在某个接口里面直接“嵌入”其它已经实现的接口，比如需要一个ReadWriter接口，里面有Read()和Write()，但是已经有一个接口Reader里面有Read()，一个接口Writer里面有Write()，就直接使用以下代码就可以了，这样Read()和Writer()就是ReadWriter接口的方法了。

    
    type Reader interface {
        Read(p []byte) (n int, err error)
    }
    
    type Writer interface {
        Write(p []byte) (n int, err error)
    }
    
    // ReadWriter is the interface that combines the Reader and Writer interfaces.
    type ReadWriter interface {
        Reader
        Writer
    }




# goroutine && channel


goroutine以及channel可以说是Go语言最重要的特性了。goroutine有一个简单的思想，它就是一个通其它goroutine运行在同一份内存空间的函数。而channel顾名思义，就是一个管道，任何数据可以通过管道进行传输，可以往channel里面放数据，可以从channel里面获取数据。goroutine和channel的配合使用，很好的解释了Go语言“**Do not communicate by sharing memory; instead, share memory by communicating**”的思想，还有一个很好的例子在[这里](http://talks.golang.org/2012/waza.slide#12)，这里例子同时也讲解了并发（concurrency）和并行（parallelism）之间的关系和区别。

我觉得，并发是一种工作方式，它把一个整体的复杂的任务分解为小型的逻辑简单的任务；而并行就是真是的同时在做许多事情。就那例子中的任务来说，把一堆书送到另一端的火堆烧掉。如果是并行的话，就是原来是一个人“拿书-运书-烧书”，现在同时有多个人“拿书-运书-烧书”。如果是并发的话：，就是把任务可以分解为：1）一个人把一部分书从书堆中放到车子上；2）一个人把装有书的车子推到火堆旁；3）一个人把火堆旁的书烧掉。每个人各司其职，只需要从一个源头（channel或者其它地方）获取原始数据，自己进行简单的加工，然后堆到另外一边（另一个channel），自己不用管其他人是怎么工作、或者什么时候工作的，只需要把自己分内的事情完成就OK了。Divide and conquer真是一种简单精妙的思想！

好了，暂时就先整理到这里吧~

参考资料：

[Effetive Go](http://golang.org/doc/effective_go.html)
[Concurrency is not Parallelism](http://talks.golang.org/2012/waza.slide)
[Go Playground](http://play.golang.org/)

--EOF--
