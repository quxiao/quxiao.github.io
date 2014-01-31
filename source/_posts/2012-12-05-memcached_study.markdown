---
author: admin
comments: true
date: 2012-12-05 16:28:32+00:00
layout: post
slug: memcached_study
title: memcached调研
wordpress_id: 678
categories:
- Note
tags:
- memcached
---

最近因开发需要，调研了下memcached，share一下！


# 什么是memcached


Memcached是一个开源的高性能，分布式的内存对象缓存系统，通过键值对的形式来对数据进行存取。


# 特性




## 简单的Key Value存储


|---Memcached只把key value当作一组字符串来组织，不关心数据的格式
|---没有持久化，所有数据都存储在内存中，当内存用满时，丢弃最旧的数据
\---数据只会保存一份，不支持replication，也不支持容错（一些客户端有自己的实现）


## Server和client共同决定操作的行为


|---Server只关心如何获取数据、存储数据、丢弃数据
\---Client只关心如何通过key定位server，以及如何和server通信


## Server相互独立


|---Server之间不会有通信，也不知道其他server的存在，但是Client知道所有server的存在。


## 所有操作都是常数时间，所有操作也都是原子操作




# 数据组织




## 内部结构


Memcached将内存分为不同的page（默认为1 Mb）、每个page被（永久）赋给某一个slab。Memcached中有多个级别的slab，每一种slab决定了内存的粒度。举例说明：
假设memcached可以操作1 Mb的内存，每个page为1Kb，有3类slab：class1、class2、class3。Class1的内存分配粒度为96字节，class2的内存分配粒度为288字节，class3的内存分配粒度为1024字节。那么，memcached的内存结构为下图：

[![](http://qxavier.me/wp-content/uploads/2012/12/memcached_internal_struct.png)](http://qxavier.me/wp-content/uploads/2012/12/memcached_internal_struct.png)


实际运行效果：





> 

> 
> ./bin/memcached -vv -m 1 -I 1k -f 3 -M -p 11225
> 
> 

> 
> slab class   1: chunk size        96 perslab      10
> 
> 

> 
> slab class   2: chunk size       288 perslab       3
> 
> 

> 
> slab class   3: chunk size      1024 perslab       1
> 
> 





class 1中，chunk的大小为96字节，共有10个chunk；
class 2中，chunk的大小为288字节，共有3个chunk；
class 3中，chunk的大小为1024字节，共有1个chunk；





# 数据处理




## 存储


数据（item）以key、value的形式进行存储，key最大为250 byte，value最大为1 Mb。
当存储item时，memcached会找到最“适合”大小的chunk用于保存这个item。例如item大小为200字节，对于上面的例子，这个item会被保存至class 2。


## 删除


Memcached不会主动删除数据，删除数据的情况有两种：



	
  1. 当新数据无法找到合适的位置进行存储时，会删除数据，删除数据的原则是LRU算法。

	
  2. 当client查找一个数据，server发现其已经过期时，就会删除该数据。


对于情况1，具体来说，当对一个item进行存储时，如果在适合的slab class中，既找不到空闲的chunk，又找不着这个slab class中空闲的page，memcached就会选择数据进行删除。Memcached首先会在处于LRU队列尾部的几个数据中，查找已经过期的数据；如果找不到过期的数据，就在队列尾部选择一个未过期的数据进行删除。


# 线程工作方式


Memcached采用libevent库进行多线程请求处理。
（libevent是一个事件出发的网络库，使用与 windows, linux, mac os 等的高性能跨平台网络库 ,支持的多种I/O 网络模型 epoll poll dev/poll select kqueue 等）
Memcached中，每个线程拥有一个自己的libevent实例。
如果memcached采用TCP/IP连接，将采用单线程监听端口，监听线程获取到请求之后，将其分配给其中一个处理线程，选取处理线程的算法为round-robin。
如果memcached采用UDP连接，所有（空闲的）处理线程都有监听UDP socket，其中一个线程监听到请求之后，进行相应处理。


# 操作


Memcached中，所有操作都是原子操作，操作时间都是常数级别。Memcached的操作有以下几类：


## 存储


Set
如有没有key对应的value，添加value；如果已存在该key对应的value，更新value。

Add
如有没有key对应的value，添加value；如果已存在该key对应的value，则操作失败。

Replace
如果已存在该key对应的value，更新value；如果没有key对应的value，则操作失败。

Append / Prepend
在key对应的value前/后添加数据

Cas（check and set）
Client会事先获取一个关于key的64位cas值，更新时，只有当server的cas值与client的cas一致时，才会执行value更新操作，否则操作失败。


## 查询


Get（Mget）
根据一个key或者一系列key，获取对应的value(s)。

Gets（get with cas）
获取key所对应的value的同时，也会返回cas值。

Exist
查询某个key是否存在（不是官方标准，部分client会实现该功能）


## 删除


Del
删除对应key的value


## 自增/自减


Incr / Decr
如果value的形式是为64位整数的字符串，Incr / Decr将对该value进行自增/自减操作。


## Flush


将memcached中的所有item都设置为过期。




# 参考





	
  * [http://memcached.org/](http://memcached.org/)

	
  * [Memcached for Dummies](http://work.tinou.com/2011/04/memcached-for-dummies.html)

	
  * [http://docs.oracle.com/cd/E17952_01/refman-5.5-en/ha-memcached-using.html](http://docs.oracle.com/cd/E17952_01/refman-5.5-en/ha-memcached-using.html)

	
  * [http://libmemcached.org/](http://libmemcached.org/)


--EOF--
