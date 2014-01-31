---
author: admin
comments: true
date: 2011-02-16 05:06:10+00:00
layout: post
slug: usaco-section-3-2-stringsobits
title: USACO Section 3.2 Stringsobits
wordpress_id: 158
categories:
- USACO
---

有这样一种集合，集合元素为长度N（1～31）的二进制串，并且每个二进制串中1的个数小于等于L，求这个集合中第I大的元素是多少？

 

最开始很天真的想枚举每个数，计算其中1的个数，结果第8组测试数据开始就超时的不行了。

 

枚举不行，来试试构造可不可以，假设我们有一个长度为n，1个数<=l的二进制串的集合，那么怎么把它们从大到小区分呢？我们一位一位来，根据第n位，可以将集合划为2部分：第n位是0的，第n为是1的。好了，递推式突然就变得很明显了。如何设num[N][L]为长度为N，1个数小于等于L的二进制串的个数，那么：

 

num[N][L] = num[N-1][L] + num[N-1][L-1]     
（第n位是0） （第n位是1）

 

个数有了，那么第I个数是多少怎么求呢？说来也简单，就是用递归的思想，看I落在num[N-1][L]和num[N-1][L-1]的哪一部分，看下面的代码应该就明白了：

 
    
    <span style="color: #0000ff">void</span> Print (<span style="color: #0000ff">int</span> len, <span style="color: #0000ff">int</span> num1, <span style="color: #0000ff">long</span> <span style="color: #0000ff">long</span> idx)
    {
         <span style="color: #0000ff">if</span> ( len == 0 )
              <span style="color: #0000ff">return</span>;
         <span style="color: #0000ff">if</span> ( num[len-1][num1] >= idx )
         {
              putchar('0');
              Print(len-1, num1, idx);
         }
         <span style="color: #0000ff">else</span>
         {
              putchar('1');
              Print(len-1, num1-1, idx-num[len-1][num1]);
         }
    }
