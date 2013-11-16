---
author: admin
comments: true
date: 2011-05-23 07:20:05+00:00
layout: post
slug: usaco-section-3-4-american-heritage
title: USACO Section 3.4 American Heritage
wordpress_id: 314
categories:
- USACO
---

题目大意：一句话！已知二叉树的前序和中序，求二叉树的后序，树的节点数不超过26。

二叉树的前序的节点处理顺序为：根节点、左子树、右子树；
中序为：左子树、根节点、右子树；
后序为：左子树、右子树、根节点。

因为前序的第一个元素是根节点，则可以找到根节点在中序中的位置idx，这样idx左边的序列为左子树、idx右边的就为右子树。根据左右子树的个数，就可以知道其在前序中的相应位置，然后就可以递归的处理左右子树了并得到它们的后序排列，最终的答案就为：左子树的后序 + 右子树的后序 + 根节点。当子问题足够小的时候，即前序、中序长度为1或0时，后序就是前序或者中序。

题目中例子的解题顺序如下图：

[![image](http://www.qxavier.me/wp-content/uploads/2011/05/image_thumb1.png)](http://www.qxavier.me/wp-content/uploads/2011/05/image1.png)

关键代码如下：

    
    string getPostOrder(string preOrder, string inOrder)
    {
         assert(preOrder.length() == inOrder.length());
         if ( preOrder.length() == 1 || preOrder.length() == 0 )
              return preOrder;
         int rootIdx;
         rootIdx = inOrder.find_first_of(preOrder[0]);
         assert(rootIdx != string::npos);
         string leftTree, rightTree;
         leftTree = getPostOrder(preOrder.substr(1, rootIdx), inOrder.substr(0, rootIdx));
         rightTree = getPostOrder(preOrder.substr(rootIdx+1, preOrder.length()-rootIdx-1), inOrder.substr(rootIdx+1, preOrder.length()-rootIdx-1));
         return leftTree + rightTree + preOrder[0];
    }
