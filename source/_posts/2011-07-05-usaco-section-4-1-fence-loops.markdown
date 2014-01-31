---
author: admin
comments: true
date: 2011-07-05 13:56:12+00:00
layout: post
slug: usaco-section-4-1-fence-loops
title: USACO Section 4.1 Fence Loops
wordpress_id: 325
categories:
- USACO
tags:
- 图论
- 深搜
---

题目大意：

 

有N个栏杆，告诉你它们的长度以及它们各自的连接情况，求出围成的最短的封闭空间的周长。其中，1 <= N <= 100，栏杆长度Ls，1 <= Ls <= 255，每个栏杆至多与另外8跟栏杆相连。

 

思路：

 

就是在图中找最小环。因为N（即边数）最大为100，即使通过枚举每个点进行搜索，复杂度也不过为100×100，可以采用搜索。枚举每个栏杆的一端顶点，如果搜索到这根栏杆的另一端顶点，那么就意味着找到环了，更新答案。

 

这题思路倒很简单，但是由于输入是以边为中心的，所以实现起来稍稍有点困难。我将每个栏杆的两个顶点定义为s端和e端，如果从某栏杆a的s端出发，那么枚举与a的e端相连的栏杆b，再判断是b的哪一端连接着a，枚举b的另一端，以此类推。

 

还有一些减枝和优化，比如：

 

  
  1. 收到[上一题](http://www.qxavier.me/2011/07/01/usaco-section-4-1-fence-rails/)的启发，可以限定搜索深度，因为组成环的边数少，周长小的概率就比较大。 
   
  2. 当前的长度和大于等于目前的答案，退出。 
   
  3. 当前栏杆之前访问过，退出。 
   
  4. 只需枚举栏杆的一端即可，因为s->e和e->s的效果是一样的。 
 

看了USACO的官方分析，貌似处理的比较复杂，先把输入转化成标准的图，再对于每一条边，先删除它，然后用Dijkstras算法求最短路。这种方案相交于我的思路，没有太多优越性，但是编码的复杂度却也高。-_-

 

关键代码如下：

 
    
    const int N = 101;
    
    int n;
    int len[N];
    vector<int> childSeg[N][2];		//[0] s end; [1] e end
    short int connect[N][N];		//connect[a][b] = 0/1 means the s/e end of a connects with seg b
    int curMax = 1<<20;
    int visited[N];
    
    /*
    seg为负，表示当前节点为seg的s端
    seg为正，表示当前节点为seg的e端
    */
    void Search(int seg/*negtive s end; positive e end*/, int curLen, int destSeg, int depth)
    {
    	if ( seg == destSeg && curLen != 0)
    	{
    		if ( curLen < curMax )
    			curMax = curLen;
    		return;
    	}
    	if ( !depth )
    		return;
    	if ( curLen >= curMax )
    		return;
    
    	int segId = abs(seg);
    	if ( visited[segId] )
    		return;
    	visited[segId] = 1;
    	int childSegId;
    	//whichEnd, segId的另一端
    	int whichEnd = 0;
    	if ( seg < 0 )
    		whichEnd = 1;
    	int i;
    	for (i = 0; i < childSeg[segId][whichEnd].size(); i ++)
    	{
    		childSegId = childSeg[segId][whichEnd][i];
    		//从childSegId的另一端继续搜索
     		if ( connect[childSegId][segId] == 0 )
    		{
    			Search(childSegId * -1, curLen + len[segId], destSeg, depth-1);
    		}
    		else if ( connect[childSegId][segId] == 1 )
    		{
    			Search(childSegId, curLen + len[segId], destSeg, depth-1);
    		}
    	}
    	visited[segId] = 0;
    }
    
    void Solve ()
    {
    	int i, j;
    	for (i = 3; i <= n; i ++)
    	{
    		for (j = 1; j <= n; j ++)
    		{
    			Search(j, 0, j, i);
    //			不需要枚举另一端，因为s->e和e->s的效果是一样的
    // 			Search(-1*j, 0, -1*j, i);
    		}
    	}
    	printf("%d\n", curMax);
    }
