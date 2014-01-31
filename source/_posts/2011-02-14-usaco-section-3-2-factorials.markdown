---
author: admin
comments: true
date: 2011-02-14 07:09:13+00:00
layout: post
slug: usaco-section-3-2-factorials
title: USACO Section 3.2 Factorials
wordpress_id: 152
categories:
- USACO
---

问你N阶乘的最低非零位上是什么数字。(0 <= N <= 4220)

 

从1一直乘到N，如果能整除10，就除以10，可以吗？不行，因为即使去掉低位的0，高位的非0位仍然很大，无法保存下来。

 

可以将N!这样表示：      
N! = 2^K * 5^L * V(N)       
= 2^(K-L) * V(N) * 10^L ( K >= L 如何证明呢？)

 

10^L不影响N!最低非零位，这个数由(K-L)以及V(N)的个位数所决定。K和L容易得到，V(N)的个位数也好得到，只要枚举i（从1到N），去除因子2和5（因子个数加到K和L），将其个位数乘以中间结果就可以了。

 

关键代码如下：

 
    
    <span style="color: #0000ff">const</span> <span style="color: #0000ff">int</span> f2 [] = {6, 2, 4, 8};
    
    <span style="color: #0000ff">int</span> i, tmp, n2, n5;
    <span style="color: #0000ff">int</span> ans = 1;
    n2 = n5 = 0;
    <span style="color: #0000ff">for</span> ( i = 1; i <= n; i ++)
    {
    	tmp = i;
    	<span style="color: #0000ff">while</span> ( tmp % 2 == 0 )
    	{
    		n2 ++;
    		tmp /= 2;
    	}
    	<span style="color: #0000ff">while</span> ( tmp % 5 == 0 )
    	{
    		n5 ++;
    		tmp /= 5;
    	}
    	ans = (( tmp % 10) * ans) % 10;
    }
    ans = ( ans * f2[( n2- n5)%4] ) % 10;
    printf( "<span style="color: #8b0000">%d\n</span>", ans);
