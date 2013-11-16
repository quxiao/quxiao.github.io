---
author: admin
comments: true
date: 2011-12-04 09:10:39+00:00
layout: post
slug: some_python_plotting_tools
title: 一些Python图表工具
wordpress_id: 480
categories:
- Tool
tags:
- python
---

考虑到最近写毕业论文以及以后自己博客中的实验分析，绘制一些图表就变得在所难免，再加上最近对Python比较感兴趣，于是就找了写利用Python绘制图表的工具库。说实话，一找一大堆，挑了几个自己感觉不错的，作为候选。

 

# [matplotlib](http://matplotlib.sourceforge.net/)

 

matplotlib可谓是matlab的Python版本，可以绘制高质量的2D图表，例如柱状图，饼状图，散点图，以及很多我不知道科学图表。除了能以python script的形式编程，还能通过ipython shell来进行交互。另外，matplotlib不但能进行绘图，还能进行复杂的科学计算（通过 [numpy](http://scipy.org/Numpy_Example_List_With_Doc)和[matplotlib.mlab](http://matplotlib.sourceforge.net/api/mlab_api.html)）。如果之前熟悉matlab的话，应该能很快上手。      
matplotlib很好很强大，图表又好看，就算对于一个搞科研的人来说，也是足够的。不过相应的代价就是学习曲线较陡，尤其是对于之前没有接触过matlab和python的人来说。

 

# [Pycha](https://bitbucket.org/lgs/pycha/wiki/Home) (PYthon CHArts)

 

Pycha是一个基于[Cairo](http://www.cairographics.org/)图形库的、轻量级、易于使用的python包。Pycha在大多数默认情况下能有很漂亮的外观，当然你也可以定制各种细节。它并不像matplotlib那样可以画出几乎所有的图形，只是提供了一些常用的图表类型。

 

# [Pychart](http://home.gna.org/pychart/)

 

也是一个轻量级的python图形库，支持线状图、柱状图、饼状图、范围图，以及Encapsulated Postscript, PDF, PNG, SVG 等多种格式。图表外观朴素不花哨，已经够用了。     
有意思的是，官网上，作者还说自己写的这个图形库已经用在了自己发表的几篇论文里面了，还列举了一些与它类似的图形库并讲了他们的优缺点。再一查作者信息，果然是牛人，在HP实验室待过 ，05年去了google。

 

另外，还有像[CairoPlot](http://linil.wordpress.com/2008/09/16/cairoplot-11/)、[ChartDirector for Python](http://www.advsofteng.com/product.html)等也是不错的选择。下面是这些图形库的柱状图例子，可以有个直观的感觉。总的来说，如果时间充裕或者今后后大量用到科学图表的话，matplotlib就是最好的选择了。如果就偶尔画一些简单的图形的话，上面介绍的都能基本满足需求，实在不行干脆直接用VISIO！

 

**matplotlib**

 

[![](http://www.qxavier.me/wp-content/uploads/2011/12/matplotlib.png)](http://www.qxavier.me/wp-content/uploads/2011/12/matplotlib.png)

 

//////////////////////////////////////////////////////////////

 

**Pycha**

 

[![](http://www.qxavier.me/wp-content/uploads/2011/12/pycha.png)        
](http://www.qxavier.me/wp-content/uploads/2011/12/pycha.png)

 

//////////////////////////////////////////////////////////////

 

**Pychart**

 

[![](http://www.qxavier.me/wp-content/uploads/2011/12/pychart1-1024x350.png)](http://www.qxavier.me/wp-content/uploads/2011/12/pychart1.png)

 

//////////////////////////////////////////////////////////////

 

**CairoPlot**

 

[![](http://www.qxavier.me/wp-content/uploads/2011/12/cairoplot.png)](http://www.qxavier.me/wp-content/uploads/2011/12/cairoplot.png)

 

//////////////////////////////////////////////////////////////

 

**ChartDirector for Python**

 

[![](http://www.qxavier.me/wp-content/uploads/2011/12/ChartDirector.png)](http://www.qxavier.me/wp-content/uploads/2011/12/ChartDirector.png)

 

--EOF--
