---
author: admin
comments: true
date: 2011-01-10 12:00:23+00:00
layout: post
slug: usaco-section-2-3-cow-pedigrees
title: USACO Section 2.3 Cow Pedigrees
wordpress_id: 135
categories:
- USACO
---

一棵树，每个节点有0或2个孩子，共N个节点，高度为K，问可以组成多少种不同的结构？

 

假设，当前树的节点问n，高度为k，那么子树可分为3种情况：

 

  
  1. 左子树高度为k-1，右子树高度为1～k-2
   
  2. 右子树高度为k-1，左子树高度为1～k-2
   
  3. 左右子树均为k-1
 

并且，满足题目要求的树的节点与高度有这样的关系：2\*k-1 <= n <= 2^k-1，可是根据这个关系枚举左右子树的节点数

 

于是就可以用递归+DP的方法解出这道题了。

 

（在对n <= 2^k-1进行转化时，自己居然写成了k >= log(n+1.0)/log(2.0)，其实应该是k >= floor (log(n+1.0)/log(2.0))，还是太粗心啦）
