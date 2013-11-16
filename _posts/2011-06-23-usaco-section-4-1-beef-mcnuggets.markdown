---
author: admin
comments: true
date: 2011-06-23 06:58:11+00:00
layout: post
slug: usaco-section-4-1-beef-mcnuggets
title: USACO Section 4.1 Beef McNuggets
wordpress_id: 320
categories:
- USACO
tags:
- DP
- 数论
---

题目大意：有n（<=10）个整数a1, a2, …, an（<=256），问是否存在无法用a1～an的线性表达式（k1a1 + k2a2 + … + knan）组成的数，如果有，其中最大的数是多少？（这个数的上限为2,000,000,000！）

思路：本想可以用简单的DP来实现，但一看到2,000,000,000的上限，放弃了DP的想法，结果好几天都没有思路。如果这个上限可以下降的足够小，那该多好！上网查了下资料以及问了下周围同学，得到一个结论：a1~an所无法组成的最大数的上限可以为其中最大两个数的最小公倍数！这样的话，题目的上限就可以一下子缩小到256×256了！DP就变得可行了。

其实，完整的证明我也暂时不能完全理解，这里先给出只有两个数的情况。

当有两个数a1和a2，a1和a2互质，可以证明无法用他们表达的数的上限为a1×a2，即大于a1×a2的数都可以表示成k1×a1+k2×a2。

即，对于任意正整数x，有：

    a1*a2 + x = k1*a1 + k2*a2

    =>  (a2 - k1)*a1 + x = k2*a1

因为k1任意，所以a2-k1也任意，将(a2-k1)*a1对a2取模，因为a1和a2互质，所以余数必然在0～a2-1的范围内，设为x’。另外，因为x也是任意的，所以我们将x表示成k*a2-x’。这样，等式左半边就可以被a2整数，等式得证。

再一次意识到，数学对于计算机科学来说是多么的重要！

好！上限确定下来之后，剩下的就是简单的DP了，以下是关键代码：

    
    void Solve ()
    {
         int i, j, bound, gcdNum;
    
         gcdNum = box[0];
         for (i = 1; i < n; i ++)
              gcdNum = gcd(gcdNum, box[i]);
         if ( gcdNum != 1 )
         {
              printf("0\n");
              return;
         }
    
         canMake[0] = 1;
         sort(box, box+n);
         bound = box[n-1] * box[n-1];
         for (i = 0; i < bound; i ++)
         {
              if ( ! canMake[i] )
                   continue;
              for (j = 0; j < n; j ++)
              {
                   if ( i + box[j] < bound )
                        canMake[i+box[j]] = 1;
              }
         }
    
         for (i = bound-1; i >= 0; i --)
         {
              if ( !canMake[i] )
              {
                   printf("%d\n", i);
                   return;
              }
         }
         printf("0\n");
    }
