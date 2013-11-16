---
author: admin
comments: true
date: 2011-05-31 02:52:29+00:00
layout: post
slug: usaco-section-3-4-raucous-rockers
title: USACO Section 3.4 Raucous Rockers
wordpress_id: 317
categories:
- USACO
tags:
- DP
---

题目大意：有N(<=20)首歌，打算放在M(<=20)张CD中，每张CD可存储T(<=20)分钟的音乐，给定每首歌的时长，问如何将歌曲按照日期（也就是输入）的顺序，存在这M张CD中，并且每首歌不可以分开存在多张CD上，使得存储的歌曲的数目最多。

来看给的样例：

    
    4 5 2
    4 3 4 2


答案是3，可以直接看出，存放的方案是将第1首4分钟的歌存入第1张CD，然后将第2首3分钟和第4首2分钟的歌存入第2张CD。

思路：
    一看到题，就冲动的想到搜索。后来理智还是打败了冲动，20层的搜索还是有些接受不了。于是试着看这题能不能DP。DP的关键就是能否准确的记录子问题，我们来想一想子问题应该如何表示。很自然的，子问题可以表示为：前n首歌考虑放入前m张CD，所能放入的最大歌曲数。然后我们就可以将问题逐渐扩展，直至N、M的规模。但是，子问题如何扩展呢？那就是将[n][m]的情况下，考虑第n+1首歌。如果不放这首歌，就转化为[n+1][m]；如果放这首歌，可以分两种情况：将第n+1首歌放在第m张CD上，将第n+1首歌放在第m+1张CD上。那怎么判断第n+1首歌可以放入第m张CD上呢？因此，还需要一个维度，那就是在[n][m]的规模下，第m张CD还剩余的时长。最后，子问题就可以表示成如下的形式：

`maxSong[songNum][diskNum][remain]`


	
  * songNum表示已考虑到的前songNum首歌

	
  * diskNum表示已考虑到的前diskNum张CD

	
  * remain表示在第diskNum张CD上剩余的时长


`maxSong[songNum][diskNum][remain]`（假设值为n）可以扩展为以下状态：

	
  1. `maxSong[songNum+1][diskNum][remain]`( = n ) 不放第songNum+1首歌

	
  2. `maxSong[songNum+1][diskNum][remain-songTime[songNum+1]]`( = n+1 ) 将第songNum+1首歌放入第diskNum张CD

	
  3. `maxSong[songNum+1][diskNum+1][T-songTime[songNum+1]]`( = n + 1 ) 将第songNum+1首歌放入第diskNum+1张CD


这样，按n、m和t的规模从小到大一次进行扩展，最终maxSong中最大的值即为答案。

关键代码如下：

    
    void Expand(int songNum, int diskNum, int remain)
    {
         if ( maxSong[songNum][diskNum][remain] == -1 )
               return;
         if ( songNum == N && diskNum == M )
               return;
         int curMax = maxSong[songNum][diskNum][remain];
         if ( remain >= songTime[songNum+1] )
         {
              maxSong[songNum+1][diskNum][remain-songTime[songNum+1]] =
                    max( maxSong[songNum+1][diskNum][remain-songTime[songNum+1]], curMax+1);
         }
         if ( T >= songTime[songNum+1] )
         {
               maxSong[songNum+1][diskNum+1][T-songTime[songNum+1]] =
                    max( maxSong[songNum+1][diskNum+1][T-songTime[songNum+1]], curMax+1);
         }
         maxSong[songNum+1][diskNum][remain] = max( maxSong[songNum+1][diskNum][remain], curMax);
    }
    
    void Solve ()
    {
         int i, j, k, ans;
         ans = -1;
         memset(maxSong, -1, sizeof(maxSong));
         maxSong[0][0][0] = 0;
         for (i = 0; i <= M; i ++)
         {
               for (j = 0; j <= N; j ++)
               {
                    for (k = 0; k <= T; k ++)
                    {
                         Expand(j, i, k);
                    }
               }
         }
    
         for (i = 0; i <= M; i ++)
         {
               for (j = 0; j <= N; j ++)
               {
                    for (k = 0; k <= T; k ++)
                    {
                         if ( maxSong[j][i][k] > ans )
                         {
                               ans = maxSong[j][i][k];
                         }
                    }
               }
         }
         printf("%d\n", ans);
    }
