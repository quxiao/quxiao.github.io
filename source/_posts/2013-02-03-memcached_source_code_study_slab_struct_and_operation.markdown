---
author: admin
comments: true
date: 2013-02-03 15:15:55+00:00
layout: post
slug: memcached_source_code_study_slab_struct_and_operation
title: memcached源码学习——Slab结构及操作
wordpress_id: 692
categories:
- Source Reading
tags:
- memcached
- slab
---

memcached的内存结构中，内存被分为一个一个的页（slab），每一个slab会被分到不同的slab_class。不同的slab_class，申请的item大小就不一样。


slab_class所对应的数据结构如下：



    
    typedef struct {
    
        //当前slab_class的最大item大小
        unsigned int size;      /* sizes of items */
        //当前slab_class每一个slab的item数
        unsigned int perslab;   /* how many items per slab */
        //被释放过的item的list
        void **slots;           /* list of item ptrs */
        //free list大小
        unsigned int sl_total;  /* size of previous array */
        //free list下标 
        unsigned int sl_curr;   /* first free slot */
        //最近申请的page(slab)的指针
        void *end_page_ptr;         /* pointer to next free item at end of page, or 0 */
        //最近申请的page(slab)所剩的item个数
        unsigned int end_page_free; /* number of items remaining at end of last alloced page */
        //当前slab_class申请了多少slab(page)
        unsigned int slabs;     /* how many slabs were allocated for this class */
        //申请的slab的指针数组
        void **slab_list;       /* array of slab pointers */
        unsigned int list_size; /* size of prev array */
        //没用到？
        unsigned int killing;  /* index+1 of dying slab, or zero if none */
        size_t requested; /* The number of requested bytes */
    
    } slabclass_t;




# slab初始化


memcached的slab初始化比较简单，主要是设定一些字段的大小，逻辑如下：

下标i从0开始遍历每一个slab class：



	
  * 对齐当前slab class的size，使其是`CHUNK_ALIGN_BYTES`字节对齐的。

	
  * 设置slabclass[i] 为size

	
  * 设置当前slab class中，每一页能存放的iterm数据slabclass[i].perslab

	
  * 更新size为下一个slab class的size

	
  * 设置最大的一个slab class，其中`size = setting.item_size_max, perslab = 1`




#  slab分配内存






如果是采用系统的malloc，就是直接申请；
如果是预先申请了所有的内存的话，依次通过以下手段获取内存：



	
  * 之前被“free”掉的item的地址会被标记在freelist中，如果freelist中有，则直接从freelist中拿；

	
  * 如果最近申请过的页中还有剩余空间，则从页的末尾申请item

	
  * 申请一个新的page







# 释放slab上的item空间


如果是系统分配，free指针，mem_malloc -= size；
否则，将地址设置到free_list末尾，如果free_list满了，double一下长度







