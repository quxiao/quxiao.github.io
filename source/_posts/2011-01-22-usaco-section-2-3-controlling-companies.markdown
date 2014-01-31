---
author: admin
comments: true
date: 2011-01-22 08:22:28+00:00
layout: post
slug: usaco-section-2-3-controlling-companies
title: USACO Section 2.3 Controlling Companies
wordpress_id: 139
categories:
- USACO
---

明明是一道难度不大的题，我却做了N天才做出来，惭愧惭愧！看到题的第一想法就是如果A控制了B，就把B所占有的股份传给A以及A的母公司，并且这样递归下去。可自己编程会发生股份重复计算的情况，解决方法是当将B的股份更新到A上时（通过母公司的关系找到A），如果之前A已经直接或间接控制了B，那么如果再加就算是重复计算了。但在判断A是否直接或间接控制B时，又要判断是否有环。

 

后来在网上找到了一种解法：当发现A控制B时，将B的股份传给A，如果发现增加股份之后，A中的股份[A][i] > 50 并且A还没有控制i，则将(A, i)加入队列。一直循环操作，直至队列为空为止。

 

官方的解题报告中，其实就是我一开始想的递归的方法，他在更新（A, B）时：

 

  
  1. 如果A已经控制B，退出 
   
  2. 若没有，control[A][B] = 1 
   
  3. 将B的股份传给A 
   
  4. 枚举已控制了A的i，递归更新（i, B） 
   
  5. 枚举A的股份[A][k]，如果大于50，递归更新（A, k） 
