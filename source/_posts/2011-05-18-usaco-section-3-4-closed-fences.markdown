---
author: admin
comments: true
date: 2011-05-18 10:47:14+00:00
layout: post
slug: usaco-section-3-4-closed-fences
title: USACO Section 3.4 Closed Fences
wordpress_id: 307
categories:
- USACO
tags:
- 计算几何
---

题目大意：
    由N（3 < N < 200）个点构成的多边形，问1) 该多边形是否为简单多边形，2) 从某一点(x, y)可以看到哪些边？看到边的任意部分就算可以看到该边，但如果从(x, y)只能看到边的某一顶点不算能看到。

                 几乎没怎么做过计算几何，鸭梨好大啊！第一问比较简单，只要线段之间没有相互相交（不含端点）就可以了。第二问没直接想到思路，参考了N种方法，终于找到一种好理解且实现简单的解法。看到这题，最直观的想法就是从观察点发出射线，找到与其相交且截距最近的线段就可以啦。但是，射线应该有哪些方向呢？360度绕一圈，效率低且精度无法掌握。较好的方法是将观察点(x, y)与每个顶点(xi, yi)连接，再将该射线按顺时针和逆时针转动很小的角度，通过这些射线找到的截距最短的线段集合就是最终的答案（没找到理论基础，只是直觉上感觉是对的）。最终答案需要排序，不过这题的排序规则没什么意思，其实就是在顺序输出的基础上，有两个例外：如果第n-1条线段可以被看到，交换第n-1条线段的两个顶点；如果第n-1和n-2条线段都可以被看到，交换两条线段。

    这道题用到了大量的计算几何模板，不用不知道，一用吓一跳。里面有一些逻辑错误和精度问题。

比如以下两个算法：

    
    /* 判断点p是否在线段l上
    条件：(p在线段l所在的直线上)&& (点p在以线段l为对角线的矩形内) */
    bool online(LINESEG l,POINT p)
    {
         return ((multiply(l.e, p, l.s)==0)
              && ( ( (p.x-l.s.x) * (p.x-l.e.x) <=0 ) && ( (p.y-l.s.y)*(p.y-l.e.y) <=0 ) ) );
    }



    
    // 如果线段l1和l2相交，返回true且交点由(inter)返回，否则返回false
    bool intersection(LINESEG l1,LINESEG l2,POINT &inter)
    {
         LINE ll1,ll2;
         ll1=makeline(l1.s,l1.e);
         ll2=makeline(l2.s,l2.e);
         if(lineintersect(ll1,ll2,inter)) return online(l1,inter);    //!!!!!
         else return false;
    }


第一个算法的精度有问题，应该把精度调低一些，把0换成EP(1e-10)：
    
    bool online(LINESEG l,POINT p)
    {
         return (abs(multiply(l.e, p, l.s) <= EP )          //!!!!!
              && ( ( (p.x-l.s.x) * (p.x-l.e.x) <= EP ) && ( (p.y-l.s.y)*(p.y-l.e.y) <= EP ) ) );
    }


第二个算法逻辑有问题，求两个线段是否相交，它的思想是先求两线段所在的直线是否相交，再看交点是否在一条直线上，但如果是下图中的情况，那答案就是不对的。

[![image](http://www.qxavier.me/wp-content/uploads/2011/05/image_thumb.png)](http://www.qxavier.me/wp-content/uploads/2011/05/image.png)

所以我改成了：

    
    // 如果线段l1和l2相交，返回true且交点由(inter)返回，否则返回false
    bool intersection(LINESEG l1,LINESEG l2,POINT &inter)
    {
         LINE ll1,ll2;
         ll1=makeline(l1.s,l1.e);
         ll2=makeline(l2.s,l2.e);
         if(lineintersect(ll1,ll2,inter)) return online(l1,inter) && online(l2, inter);
         else return false;
    }


经过修改，这题终于AC了！关键代码如下：

    
    int nearestLine (LINESEG ray, LINESEG* lines, int num)
    {
         double curMinDis, tmp;
         int minIdx, i;
         POINT interPoint;
    
         curMinDis = INF;
         minIdx = -1;
         //every fence
         for (i = 0; i < num; i ++)
         {
              if ( intersection(lines[i], ray, interPoint) && (tmp=dist(ray.s, interPoint)) <= curMinDis)
              {
                   curMinDis = tmp;
                   minIdx =  i;
              }
         }
         return minIdx;
    }
    
    void FindFence ()
    {
         int k = 200;          //射线延长倍数
         double delta = 1e-10;     //旋转角度
         LINESEG ray, line1;
         POINT interPoint;
         int i, j;
         double curMinDis, tmp;
         int minIdx;
    
         memset(visible, 0, sizeof(visible));
         ray.s = ob;
         //every point
         for (i = 0; i < n; i ++)
         {
              ray.e = point[i];
    
              ray.e.x = k * (ray.e.x - ray.s.x) + ray.s.x;
              ray.e.y = k * (ray.e.y - ray.s.y) + ray.s.y;
    
              line1.s = ray.s;
              line1.e = rotate(ray.s, delta, ray.e);
              minIdx = nearestLine(line1, fence, n);
              if ( minIdx != -1 )
              {
                   visible[minIdx] = 1;
              }
    
              line1.e = rotate(ray.s, -1 * delta, ray.e);
              minIdx = nearestLine(line1, fence, n);
              if ( minIdx != -1 )
              {
                   visible[minIdx] = 1;
              }
         }
    }
    
    void Output ()
    {
         int i, cnt = 0;
    
         for (i = 0; i < n; i ++)
              if ( visible[i] )
                   ++ cnt;
    
         printf("%dn", cnt);
         if ( visible[n-1] )
         {
              swap(fence[n-1].s, fence[n-1].e);
              if ( visible[n-2] )
                   swap(fence[n-2], fence[n-1]);
         }
         for (i = 0; i < n; i ++)
         {
              if ( visible[i] )
              {
                   printf("%.0lf %.0lf %.0lf %.0lfn", fence[i].s.x, fence[i].s.y, fence[i].e.x, fence[i].e.y);
              }
         }
    }

