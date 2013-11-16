---
author: admin
comments: true
date: 2011-03-09 05:55:07+00:00
layout: post
slug: usaco-section-3-3-home-on-the-range
title: USACO Section 3.3 Home on the Range
wordpress_id: 231
categories:
- USACO
tags:
- DP
- usaco
---

题目大意：给定包含0和1的边长为N(2<=N<=250)的正方形，统计其中边长>=2并且全部由1组成的子正方形，子正方形可相互覆盖。

 

我的理解是，建立三个数组rightEx[][], downEx[][]和squareSize[][]，分别表示每一点最右和最下能扩展的距离以及可向右下扩展的（目前）最大正方形的边长，然后枚举组成的正方形边长K（2～N），再枚举每一个点（i, j），若满足以下三个条件：

 

  
  * rightEx[i][j] >= k 
   
  * downEx[i][j] >= k 
   
  * squareSize[i+1][j+1] >= k – 1 
 

则squareSzie[i][j] = k，并且更新答案。

 

算法复杂度为O(N^3)，还是可以接受的。
