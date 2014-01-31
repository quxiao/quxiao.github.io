---
author: admin
comments: true
date: 2011-01-06 14:30:52+00:00
layout: post
slug: usaco-section-2-3-longest-prefix
title: USACO Section 2.3 Longest Prefix
wordpress_id: 133
categories:
- USACO
tags:
- prefix
- tire
---

给定一字符串集合Pn, 1<=n<=200, 1<=strlen(Pn)<=10，再给定一个字符串S，1<=strlen(s)<=200000。问能用字符串集合组成的S的最长前缀的长度。

 

之前想的是用KMP将每一个集合中的Pn与S匹配，找出Pn能构成S子串的位置，组成200×200000的矩阵，再在这个矩阵上进行搜索。可是发现时间和内存的限制都无法满足

 

题目有这样的特点：S的长度很长(200000)，而Pn的长度却很短(10)，而且组成的字符只有A-Z二十六个。那么，Pn中有许多字符串的前缀是重合的，可否利用这一特点避免许多重复的搜索。

 

例如：Pn={A, AB}，S=“ABA”，用P0=A对S[0]=A查找完之后，再查找S[1]=B时，能否直接转到P1=AB的B呢？

 

后来发现，用Trie可以存下所有Pn的信息，而且Pn中的重复前缀会放在一起，这样就避免了重复查找。（以前听过Tire，感觉很神秘，原来就是字典树啊，之前也用过啊）
