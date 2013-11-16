---
author: admin
comments: true
date: 2011-07-14 16:00:54+00:00
layout: post
slug: usaco-section-4-1-cryptcowgraphy
title: USACO Section 4.1 Cryptcowgraphy
wordpress_id: 328
categories:
- USACO
tags:
- hash
---

题目大意：

一种字符串加密算法，即在原有字符串的任意位置插入C、O、W，C的下标 < O的下标 < W的下标，将C、O之间和O、W之间子串交换，得到新的字符串。给你一个字符串，问其是否是对字符串Begin the Escape execution at the Break of Dawn经过多次加密过后的字符串。

我的思路：

首先，原字符串中并没有COW三个字母，所以不用判断COW是属于原先的字符还是加密时加上的字符，这样简单很多。另外，对于已加密的字符串，只要找到任意一组COW，交换CO和OW之间的子串，再把COW删掉，得到的字符串就是可能的一种原字符串，再这样递归下去，直到不可能的情况或者字符串变为Begin the Escape execution at the Break of Dawn。

在搜索时，有几种减枝可以排除目前的字符串：



	
  * 如果目前字符串的长度不等于原串长度+3×k，排除。（其实只需在开始时判断一次即可）

	
  * 如果目前字符串中COW三个字母的数目不相等，排除。（也可只判断一次）

	
  * 如果任意COW字符之间的子串，没有出现在原串中，排除。（这个减枝很重要，但是没想到，不应该啊）

	
  * 字符串中，所有出现COW的地方，C应该排在第一个，W应该排在最后一个。否则，无法还原成原字符串。

	
  * 用hash来判断目前字符串之前有没有出现过。（hash表的大小选择很关键，选的太大，可能会超时，选的太小，可能会WA）


另外，枚举COW时的搜索方式也很重要。如果每一层都会从i到len，像这样：

    
    	for (i = 0; i < l; i ++)
    	{
    		if ( str[i] != 'C' )
    			continue;
    		for (j = i + 1; j < l; j ++)
    		{
    			if ( str[j] != 'O' )
    				continue;
    			for (k = j + 1; k < l; k ++)
    			{
    				if ( str[k] != 'W' )
    					continue;
    				if ( CheckStr( RestoreString(str, i, j, k)) )
    					return true;
    			}
    		}
    	}


很容易就超时了。

但是，如果先把所有COW的下标先记下来，再在每一层枚举它们的下标，像这样：

    
    	for (i = 0; i < cNum; i ++)
    	{
    		for (j = 0; j < oNum; j ++)
    		{
    			for (k = 0; k < wNum; k ++)
    			{
    				if ( cIdx[i] < oIdx[j] && oIdx[j] < wIdx[k] )
    				{
    					if ( CheckStr( RestoreString(str, cIdx[i], oIdx[j], wIdx[k])) )
    						return true;
    				}
    			}
    		}
    	}


就会快很多。

好了，废话不多说，直接上代码：

    
    unsigned int elf_hash(string str)
    {
    	unsigned long h = 0, g, i, l;
    	l = str.length();
    	for (i = 0; i < l; i ++)
    	{
    		h = (h << 4) + str[i];
    		if (g = h & 0xf0000000l)
    			h ^= g >> 24;
    		h &= ~g;
    	}
    	return h % HASH_NUM;
    }
    
    //COW的数目是否相等，以及数目是多少
    //其实跟CheckStr中的判断有重复，还可以优化！
    int CheckCOWNum (string str)
    {
    	int i, l;
    	int nC, nO, nW;
    	nC= nO = nW = 0;
    	l = str.length();
    
    	for (i = 0; i < l; i ++)
    	{
    		if ( str[i] == 'C' )
    			++ nC;
    		else if ( str[i] == 'O' )
    		{
    			++ nO;
    		}
    		else if ( str[i] == 'W' )
    		{
    			++ nW;
    		}
    	}
    	if ( nC == nO && nO == nW )
    		return nC;
    	else
    		return -1;
    }
    
    //还原加密的字符串
    //idx1 < idx2 < idx3
    string RestoreString (string now, int idx1, int idx2, int idx3)
    {
    	string pre = "";
    	int i, l;
    	l = now.length();
    	pre = now.substr(0, idx1);
    	pre += now.substr(idx2+1, idx3-idx2-1);
    	pre += now.substr(idx1+1, idx2-idx1-1);
    	pre += now.substr(idx3+1, l-idx3-1);
    
    	return pre;
    }
    
    bool CheckBetweenCOW (string str)
    {
    	int lastIdx;
    	int i, l;
    	lastIdx = -1;
    	l = str.length();
    	for (i = 0; i < l; i ++)
    	{
    		if ( str[i] == 'C' || str[i] == 'O' || str[i] == 'W' )
    		{
    			if ( finalStr.find(str.substr(lastIdx + 1, i - lastIdx - 1)) == string::npos )
    			{
    				return false;
    			}
    			lastIdx = i;
    		}
    	}
    	return true;
    }
    
    bool CheckStr (string str)
    {
    	if ( str == finalStr )
    		return true;
    
    	int hash;
    	hash = elf_hash(str);
    	if ( hashVisited[hash] )
    		return false;
    	hashVisited[hash] = true;
    
    	int i, j, k, l;
    	int cNum, oNum, wNum, allNum;
    	int cIdx[NUM], oIdx[NUM], wIdx[NUM], allIdx[NUM];
    	cNum = oNum = wNum = allNum = 0;
    	l = str.length();
    
    	//记录C O W的下标
    	allIdx[allNum++] = -1;
    	for (i = 0; i < l; i ++)
    	{
    		if ( str[i] == 'C' )
    		{
    			cIdx[cNum++] = i;
    			allIdx[allNum++] = i;
    		}
    		else if ( str[i] == 'O' )
    		{
    			oIdx[oNum++] = i;
    			allIdx[allNum++] = i;
    		}
    		else if ( str[i] == 'W' )
    		{
    			wIdx[wNum++] = i;
    			allIdx[allNum++] = i;
    		}
    	}
    	if ( allIdx[allNum-1] != l - 1 )
    		allIdx[allNum++] = l;
    
    	//所有出现COW的地方，C应该排在第一个，W应该排在最后一个。否则，无法还原成原字符串
    	if ( cNum )
    	{
    		if ( cIdx[0] > oIdx[0] || cIdx[0] > wIdx[0] )
    			return false;
    		if ( wIdx[wNum-1] < cIdx[cNum-1] || wIdx[wNum-1] < oIdx[oNum-1] )
    			return false;
    	}
    
    	//任意COW之间的子串都应该在原串中出现过
    	for (i = 0; i < allNum-1; i ++)
    	{
    		if ( finalStr.find(str.substr(allIdx[i]+1, allIdx[i+1]-allIdx[i]-1)) == string::npos )
    			return false;
    	}
    
    	for (i = 0; i < cNum; i ++)
    	{
    		for (j = 0; j < oNum; j ++)
    		{
    			for (k = 0; k < wNum; k ++)
    			{
    				if ( cIdx[i] < oIdx[j] && oIdx[j] < wIdx[k] )
    				{
    					if ( CheckStr( RestoreString(str, cIdx[i], oIdx[j], wIdx[k])) )
    						return true;
    				}
    			}
    		}
    	}
    
    	//这样就会很慢。。。
    	// 	for (i = 0; i < l; i ++)
    	// 	{
    	// 		if ( str[i] != 'C' )
    	// 			continue;
    	// 		for (j = i + 1; j < l; j ++)
    	// 		{
    	// 			if ( str[j] != 'O' )
    	// 				continue;
    	// 			for (k = j + 1; k < l; k ++)
    	// 			{
    	// 				if ( str[k] != 'W' )
    	// 					continue;
    	// 				if ( CheckStr( RestoreString(str, i, j, k)) )
    	// 					return true;
    	// 			}
    	// 		}
    	// 	}
    
    	return false;
    }
    
    void Solve ()
    {
    	if ( (str.length() - finalStr.length()) % 3 != 0 )
    	{
    		printf("0 0\n");
    		return;
    	}
    
    	int nCOW = CheckCOWNum(str);
    	if ( nCOW == -1 || !CheckStr(str) )
    	{
    		printf("0 0\n");
    		return;
    	}
    	printf("1 %d\n", nCOW);
    }
