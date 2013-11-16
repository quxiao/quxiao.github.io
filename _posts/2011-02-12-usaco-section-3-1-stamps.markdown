---
author: admin
comments: true
date: 2011-02-12 03:58:43+00:00
layout: post
slug: usaco-section-3-1-stamps
title: USACO Section 3.1 Stamps
wordpress_id: 149
categories:
- USACO
---

有N(1<=N<=50)种不同面值邮票，由这些邮票组成面值1～M，1、2、……、M每种面值均由不超过K(1<=K<=200)数目的邮票组成，求最大的M为多少？邮票最大面值为10000

 

一开始想到DP，数组canComprise[10000\*200][200]，canComprise[i][j]表示用j张邮票是否可以组成面值i，但数组太大，放弃。

 

后来改用深搜，优化了许久，最后几组数组仍然超时，放弃。

 

又回头想DP，如果canComprise[i][j1]和canComprise[i][j2]均为true，j1 < j2，那么canComprise[i][j1]肯定是更优的解，因为j1可以扩展更多i+stamps[x]。所以，只要用一维数组保存答案就可以了，比如minStamp[i] = j就表示组成i所用到的最少邮票数为j，递推式很容易想到：

 

minStamp[i] = Min{ minStamp[i-stamp[x]] + 1 } ( i – stamp[x] >= 0 )

 

 

**同一种情况，表达解的方式可能有多种，尽量使用最精简的方式，已达到降维的效果。**
