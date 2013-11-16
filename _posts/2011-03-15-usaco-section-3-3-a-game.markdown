---
author: admin
comments: true
date: 2011-03-15 03:03:36+00:00
layout: post
slug: usaco-section-3-3-a-game
title: USACO Section 3.3 A Game
wordpress_id: 233
categories:
- USACO
tags:
- DP
- usaco
---

题目大意：有一串长度为N（2<=N<=100）的正整数，两个人按顺序选取最左边或最右边的一个数，将这个数字从数列中去掉，求在双方都进行最优化的选择时，双方选择数字的和分别为多少？

一开始题目的状态为（0，n-1），我们来考虑某个中间状态（i，j）。当前玩家有两种选择：i或者j。选择i，状态就变成（i+1，j）；选择j，状态就变为（i，j-1）；当i==j时，玩家没得选，只能选择i，然后游戏结束。

我们可以用f[i][j]表示在(i, j)情况下，当前玩家的最优解，即从(i, j)到游戏结束所选择的最大数字和。因为双方都是进行最优化选择，所以根据当前玩家选择i，则另一玩家之后的最优解为f[i+1][j]，那么这个情况下当前玩家拿到的数字和是多少呢？应该是sum[i][j] – f[i+1][j]。（sum[i][j]表示i到j的和）。同理，当前玩家选择j时，情况也是类似的。最后我们得到如下递推式：


    f[i][j] = max { sum[i][j] – f[i+1][j], sum[i][j] – f[i][j-1] } ( i < j )

    f[i][j] = v[i] ( i == j )


实现时，用递归加记忆化就可以解决了，算法复杂度为O(N^2)。

关键代码如下：

    
    int CalF(int i, int j)
    {
         if ( f[i][j] != -1 )
              return f[i][j];
         if ( i == j )
              return f[i][j] = v[i];
         return ( f[i][j] = sum[i][j] - min(CalF(i+1, j), CalF(i, j-1)) );
    }
