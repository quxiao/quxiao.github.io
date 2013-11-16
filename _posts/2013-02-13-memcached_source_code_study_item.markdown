---
author: admin
comments: true
date: 2013-02-13 14:03:13+00:00
layout: post
slug: memcached_source_code_study_item
title: memcached源码学习——item
wordpress_id: 701
categories:
- Source Reading
tags:
- memcached
---

item是memcached操作的最小单位，一个item包括以下数据：









	
  * struct item 本身大小

	
  * key (`uint8_t`)

	
  * suffix (`uint8_t`)，内容是" %d %d\r\n", flag, nbytes-2

	
  * value

	
  * CAS（optional, `uint64_t`）





(参考`do_item_alloc` && `item_make_header`)

与item相关的操作有以下几种


# 分配item


流程如下：






	
  1. 确定item大小——ntotal

	
  2. 找到合适的slab

	
  3. 看该slab的（LRU）tails中是否有item
    3.1 如果tails没有，
        在该slab上申请ntotal大小的内存
    3.2 如果a) tails上面有item，并且b)其引用计数（refcount）为0，c)item是过期的（time和exptime），那么
        重新计算这个slab上面分配的总item大小，
        更新关联数组（do_item_unlink_noblock），就是链表的删除节点操作。
        （注意：_hashitem_before()函数中，使用了&引用操作）
    3.3 如果slab上面试图分配但是分配不到，并且最tails上的item没有过期（expire）
        3.3.1 如果不允许将item给evict
                return NULL
        3.3.2 否则
                只好把这个tails上的item给evict掉
                重新计算这个slab上面分配的总item大小，
                删除关联数组中的该item节点

	
  4. 初始化item各个字段




#  free item
	
  1. 当一个item的引用计数为0时，就将这个item“释放”（其实没有真正的释放）

	
  2. 计算item的大小ntotal

	
  3. 判断是在哪个slab_class上

	
  4. 在该slab_class上，释放ntotal大小的空间（之前的slab上释放空间的操作）


# hashtable && heads && tails


当对item进行创建、删除等操作的时候，除了在slab上进行申请、释放空间之外，还需要更新以下数据：



	
  * item的time和flag

	
  * 全局stats

	
  * primary_hashtable

	
  * 与item对应的slab的heads和tails数组


其中，

time就是当前创建时间，flag表示item目前的状态（TODO）

`primary_hashtable`是一个`item**`结构
`primary_hashtable`的大小是2 ^ hashpower，hashpower默认大小为16
item与`primary_hashtable`的对应关系为：
    hash_key(item) & (2^hashporwer - 1)

heads和tails数组用于记录每一个`slab_class`的`item*`的链表，用于进行LRU淘汰，所以说，memcached的LRU算法是每个`slab_class`独立的。

--EOF--











