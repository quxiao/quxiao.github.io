---
author: admin
comments: true
date: 2011-03-04 07:59:14+00:00
layout: post
slug: usaco-section-3-3-shopping-offers
title: USACO Section 3.3 Shopping Offers
wordpress_id: 222
categories:
- USACO
tags:
- DP
---

题目大意：

买N（0<=N<=5）类商品，每种买K（0<=K<=5）件，再给出S（0<=S<=99）个优惠价格列表，每种优惠给定一种商品及其数量的组合。

优惠列表如下：

    
    
    1 7 3 5
    2 7 1 8 2 10
    


表示第1种优惠包含1类商品：7号商品买3件，优惠价为5；第2种优惠包含2类商品：7号商品买1件，8号商品买2件，优惠价为10。

求买这N类商品，每种K件，购买的最低价格是多少？

思路：

题目中要求买的商品的种类和数量都很少（<=5），这很好，但S的数量级有点高，尽量避免枚举S很重要。商品最多5种，每种商品0～5个，那么商品类型和数目的组合状态就是6^5种。每种组合的最优解（即最低价格）是唯一的，而组合的种类又不多，则很容易想到把子问题的解保存起来，避免重复计算。

对于当前的商品类型价格组合，如果之前已计算出来，则直接返回。否则，枚举每一种优惠，如果可以使用该种优惠，计算优惠价格加上剩下的组合，返回所有可以使用优惠的价格以及不使用优惠的价格中的最小值，作为当前情况的最优解。这样，问题就可以解决了，提交之后发现效率还可以。

关键代码如下：

    
    int getMinCost (int num[PRODUCT_NUM])
    {
    	int curState;
    	curState = getState(num);
    	if ( minCost[curState] != -1 )
    		return minCost[curState];
    
    	int i, j, minPrice = 0, curPrice, tmpNum[PRODUCT_NUM];
    	bool check;
    	for (i = 0; i < proNum; i ++)
    		minPrice += num[i] * price[i];
    	for (i = 0; i < offerNum; i ++)
    	{
    		curPrice = 0;
    		check = true;
    		for (j = 0; j < proNum; j ++)
    		{
    			if ( num[j] - offerProIdxNum[i][j] < 0 )
    			{
    				check = false;
    				break;
    			}
    			tmpNum[j] = num[j] - offerProIdxNum[i][j];
    		}
    		if ( check )
    		{
    			curPrice = offerPrice[i] + getMinCost(tmpNum);
    			if ( curPrice < minPrice )
    				minPrice = curPrice;
    		}
    	}
    	return (minCost[curState] = minPrice);
    }
