---
author: admin
comments: true
date: 2011-07-21 13:12:20+00:00
layout: post
slug: avl_tree_implimentation
title: AVL树初步实现
wordpress_id: 350
categories:
- Data Structure
tags:
- AVL Tree
---

这两天复习数据结构，看到AVL树这一部分，之前这部分一直处于“纸上谈兵”的阶段，表示有些遗憾。趁着最近比较闲，把它实现一下！

 

还是先讲一下AVL树的基本概念吧。其实，AVL树就是二叉树，与普通二叉树不同的是，它能通过某种操作（“旋转”）使得树保持几乎平衡，这里就是指左子树和右子树的树高差的绝对值<=1。当二叉树是平衡的情况下，在树上的搜索效率就会提高。极端情况下，能从O(N)降到O(logN)。

 

OK，那么又有哪几种不平衡的情况呢？我们来看一下下面几种情况：

 

右子树的右子树太“深”

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image.png)

 

树往一边倾斜，这是一种常见的不平衡状态。比如把一个已排序的数据插入二叉树就是这种情况。遇到这种情况，如果可以把k2“提”起来就好了。很简单，我们可以把k2作为树根，k1做k2的左孩子，再把B当作k1的右子树。旋转之后可以得到：

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb1.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image1.png)

 

这样，树又基本平衡了。左子树的左子树太深的情况可依此类推。这两种情况成为单旋转。

 

还有一种情况，右子树的左子树太“深”

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb2.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image2.png)

 

这样就k2不好“提”了，因为旋转之后，B子树的树高没有改变。把树描述的更加详细一些：

 

[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb3.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image3.png) => [![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb4.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image4.png)

 

我们的目标是把k3以及BC放的高一些，以降低整体树高。因为k1<=k3<=k2，那可否让k3来做树根呢？k3做了树根，k1和k2是其左右孩子，AD子树可以不变，B可以作为k1右子树，C可以作为k2左子树，这样就搞定了！

 

同样，左子树的右子树太深的情况也是类似。这两种情况成为双旋转。

 

好，用代码实现这些操作！

 

根据单旋转的图例，两步操作就可以完成：

 

  
  1. k2的左指针指向k1 
   
  2. k1的右指针指向B 
 

代码实现就是这样：

 

 
    
    AVLTree* singleRotateWithRight(AVLTree* pNode)
    {
    	if ( pNode == NULL || pNode->right == NULL )
    		return NULL;
    	AVLTree* rightNode = pNode->right;
    	pNode->right = rightNode->left;
    	rightNode->left = pNode;
    
    	pNode->height = max(getHeight(pNode->left), getHeight(pNode->right)) + 1;
    	rightNode->height = max(getHeight(rightNode->left), getHeight(rightNode->right)) + 1;
    
    	return rightNode;
    }





再看双旋转的实现，很奇妙的是，一个双旋转可以用两个但旋转来代替！





[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb5.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image5.png)=>[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb6.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image6.png) =>[![image](http://www.qxavier.me/wp-content/uploads/2011/07/image_thumb7.png)](http://www.qxavier.me/wp-content/uploads/2011/07/image7.png)





实现起来相当简单：








    
    AVLTree* doubleRotateWithRight (AVLTree* pNode)
    {
    	pNode->right = singleRotateWithLeft(pNode->right);
    	return singleRotateWithRight(pNode);
    }





插入操作怎么实现呢？其实就是在普通二叉树的插入操作之后，加入判断是否不平衡，若不平衡，判断是哪种情况，进行相应旋转操作。








    
    AVLTree* insert(int data, AVLTree* pNode)
    {
    	if ( pNode == NULL )
    	{
    		pNode = new AVLTree();
    		pNode->data = data;
    		pNode->height = 0;
    		pNode->left = pNode->right = NULL;
    	}
    	else if ( data < pNode->data )
    	{
    		pNode->left = insert(data, pNode->left);
    		if ( getHeight(pNode->left) - getHeight(pNode->right) == 2 )
    		{
    			if ( data < pNode->left->data )
    			{
    				pNode = singleRotateWithLeft(pNode);
    			}
    			else
    			{
    				pNode = doubleRotateWithLeft(pNode);
    			}
    		}
    	}
    	else if ( data > pNode->data )
    	{
    		pNode->right = insert(data, pNode->right);
    		if ( getHeight(pNode->right) - getHeight(pNode->left) == 2 )
    		{
    			if ( data > pNode->right->data )
    			{
    				pNode = singleRotateWithRight(pNode);
    			}
    			else
    			{
    				pNode = doubleRotateWithRight(pNode);
    			}
    		}
    	}
    
    	return pNode;
    }






    
好吧，先写这么多，性能对比以后再写。全部实现代码可以在[这里](http://code.google.com/p/quxiao-source-code/source/browse/#svn%2Ftrunk%2Fdata_structure%2FAVLTreeCPP%2FAVLTreeCPP)看到。
