---
author: admin
comments: true
date: 2011-01-30 08:59:01+00:00
layout: post
slug: usaco-section-3-1-humble-numbers
title: USACO Section 3.1 Humble Numbers
wordpress_id: 145
categories:
- USACO
---

最直接的想法是枚举每个数，看是否能用S中的元素将其分解，但1<=N<=100000，第N个数肯定会很大，这样做肯定超时，放弃。

 

后来想利用STL中的set来解决，枚举某一个数，如果属于set，将其与S中各元素相乘的数放入set，如此循环，直至找到第N个数，提交后还是超时。看来即便是set，毕竟存取的效率不是O(1)，性能还是有影响。

 

突然想到，这题不是跟poj的[Ugly Number](http://poj.org/problem?id=1338)挺像的嘛，是Ugly Number的加强版。具体思想是：对于S中的每个元素p[i]，设置一个下标pIdx[i]，pIdx[i]指向humble number数组。进行N次循环，每次找出最小的p[i] * humble[pIdx[i]]，将该数加入humble数组，然后pIdx[minIdx]++。这样就能由小到大找出第N个humble number了。

 

PS：其实这种方法生成的humble number只能保证非降序，比如2×3和3×2就会生成相同的humble number，这种情况要排除。
