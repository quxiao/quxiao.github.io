---
author: admin
comments: true
date: 2011-07-01 06:01:32+00:00
layout: post
slug: usaco-section-4-1-fence-rails
title: USACO Section 4.1 Fence Rails
wordpress_id: 322
categories:
- USACO
tags:
- dfsid
- 减枝
---

题目大意：

给你N(1 <= N <= 50)段长度不一的木材，另外有R(1 <= R <= 1023)条栏杆需要修复，1 <= ri <= 128。问以这些木材为原材料，能够做出最多多少块栏杆？（做出的栏杆需要是完整的，不能拼接）

思路：

这是一道背包问题，不过，这是高维背包。是把R个栏杆放入N段木材中，传统背包的DP解法在这里貌似就无能为力了，看了hint才知道，原来要用到DFSID算法，也就是限制搜索深度的DFS。是一种类似BFS的求解过程，但是却用DFS来实现的算法。简单的说，每次枚举搜索深度，进行DFS，如果未得到解，则继续增加（减少）搜索深度，直至找到解。

搜索的过程中，有一些重要的减枝：



	
  1. **如果可以组成k块栏杆，那么最小的前k块栏杆一定是其中的一个解。**因为就算存在不连续的解，我们也可以将较大的栏杆替换成较小的栏杆，最终转化为最小的k块栏杆。

	
  2. 根据R和ri ，可以看出栏杆的长度有很多都是**重复**的。将木材和栏杆排序，搜索时记录下当前的栏杆是选择哪块木材。搜索到当前栏杆i时，如果发现与上一个栏杆i+1的长度相同，则我们可以从i+1块栏杆所选择的那块木材开始搜。

	
  3. 假设前k块栏杆是题目的解，那么在这种情况下最大浪费掉的木材maxWaste就是sumBoard – sumRail[k]。在搜索过程中，如果发现某块木材的剩余量比最小的栏杆都小，就可以把他计入curWaste，如果发现curWaste > maxWaste，那么肯定无解。

	
  4. 我们还可以将R的范围减小，如果发现sumRail[k] <= sumBoard并且sumRail[k+1] > sumBoard，那么我们可以将R减小到k。


嗯，差不多就是这些。还有一个减枝，就是如果发现栏杆长度比最大的木材还要长的话，那在输入的时候就略去它（貌似减枝效果不明显-_-）。我们从（经过优化过的）R开始搜索，假设前k个栏杆就是解，对于每个栏杆，枚举木材，基本上就是这么个思路。做完这题的最大感受就是：要善于挖掘题目中的潜在信息。根据R和ri的关系，得知栏杆有许多重复的情况，以及如何推导出如果k为解，则前k个栏杆肯定能构成解，这就是真正的分析问题的能力了！

以下是关键代码：

    
    bool Search (int rIndex)
    {
    	if ( rIndex < 0 )
    		return true;
    	int bound = 0;
    	//如果当前rail和上一个rail的长度相同，则从上一个rail选的board开始搜索
    	if ( rIndex < r - 1 && rail[rIndex] == rail[rIndex+1] )
    		bound = railSelect[rIndex+1];
    	int i;
    	bool ret = false;
    
    	for (i = bound; i < n; i ++)
    	{
    		if ( board[i] >= rail[rIndex] )
    		{
    			board[i] -= rail[rIndex];
    			railSelect[rIndex] = i;
    			if ( board[i] < rail[0] )
    			{
    				curWaste += board[i];
    				//如果当前浪费的board大于最大允许的浪费量，则不选这个board
    				if ( curWaste > maxWaste )
    				{
    					curWaste -= board[i];
    					board[i] += rail[rIndex];
    					railSelect[i] = -1;
    					continue;
    				}
    			}
    			ret = Search(rIndex - 1);
    			//恢复之前状态，顺序很重要！
    			if ( board[i] < rail[0] )
    				curWaste -= board[i];
    			board[i] += rail[rIndex];
    			railSelect[rIndex] = -1;
    			if ( ret )
    				return true;
    		}
    	}
    
    	return false;
    }
    
    void Solve ()
    {
    	int i;
    	memset(railSelect, -1, sizeof(railSelect));
    	sort(rail, rail + r);
    	sort(board, board + n);
    
    	boardSum = 0;
    	for (i = 0; i < n; i ++)
    		boardSum += board[i];
    	//找到可以优化的R
    	//用一个int表示前k个rail之和
    	railSum = 0;
    	for (i = 0; i < r; i ++)
    	{
    		railSum += rail[i];
    		if ( railSum > boardSum )
    		{
    			railSum -= rail[i];
    			break;
    		}
    	}
    
    	-- i;
    	for (; i >= 0; i --)
    	{
    		curWaste = 0;
    		maxWaste = boardSum - railSum;
    		if ( Search(i) )
    		{
    			printf("%d\n", i+1);
    			return;
    		}
    		railSum -= rail[i];
    	}
    	printf("0\n");
    }
