---
author: admin
comments: true
date: 2011-11-14 07:09:09+00:00
layout: post
slug: some_concepts_in_hadoop
title: Hadoop中的一些概念
wordpress_id: 456
categories:
- Development
tags:
- hadoop
---

Hadoop，在不同的上下文的环境下，会表示不同范围的概念。有人可能把Hadoop单单理解为一个分布式文件系统，我感觉欠妥，因为也有很多其他的分布式文件系统。Hadoop之所以称为Hadoop，其核心在于它的MapReduce框架。所以，Hadoop的最精简的理解应该是HDFS （Hadoop Distributed File System）+ MapReduce。除了核心，也可以算上它的许多子项目，例如Hbase、Pig、Hive等等，可以通过下图来了解下Hadoop的Big Picture:

[![hadoop_ecosys](http://www.qxavier.me/wp-content/uploads/2011/11/hadoop_ecosys_thumb.jpg)](http://www.qxavier.me/wp-content/uploads/2011/11/hadoop_ecosys.jpg)

既然HDFS以及MapReduce是Hadoop的核心，对于它们的一些概念就要有所掌握。好，下面来讲解一下。


### HDFS中的重要概念：


**NameNode
**它是整个HDFS的核心，里面包含了整个文件系统中的所有文件的路径，并且监控存有文件的服务器，NameNode本身并不存储文件。NameNode就相当于一个接口，当用户想要对HDFS中的文件进行读/写/添加/删除时，首先要询问NameNode，然后NameNode再给你一个存储着文件的服务器（DataNode）的列表。
注意：HDFS中只有一个NameNode （除非使用ZooKeeper），也就是意味着如果NameNode失效了，整个HDFS也就失效了。

**DataNode
**存放HDFS数据的节点，一般来说，HDFS都有多个DataNode，一个数据会在不同的DataNode上存储多份，以防止部分节点失效后数据丢失。DataNode启动后，会一直试图通知NameNode，以便让NameNode知道它的存在。

**SecondaryNameNode
**为了弥补NameNode单点失效的缺点，可以在一台单独机器上配置SecondaryNameNode。不过，SecondaryNameNode只是保存一些记录点（check point），并不会对数据进行冗余保存。


### MapReduce计算模型中的重要概念：


**TaskTracker
**集群中真正执行计算的节点，它的操作包括：Map、Reduce以及Shuffle。每个TaskTracker都设置了一系列的slot，表示这个TaskTracker可以同时接受任务的数量。

**JobTracker
**分发MapReduce任务的节点。这个（活动的）JobTracker在一个MapReduce服务中只有一个，因此JobTracker也是单点失效的，如果JobTracker失效了，所有TaskTracker中的任务都会停止。

JobTracker和TaskTracker的交互过程：



	
  * 用户提交一个MapReduce任务（Job）给JobTracker

	
  * JobTracker与NameNode通信，以确定数据的位置

	
  * 根据数据的位置，定位那些1）在数据节点或者在数据节点附近 2）有可用slot的TaskTracker

	
  * JobTracker向TaskTracker提交对应的任务

	
  * JobTracker开始监控TaskTracker，如果监控不到TaskTracker的心跳，或者TaskTracker上面执行的任务失败，就将任务提交到其他TaskTracker

	
  * 当TaskTracker上的任务完成后，JobTracker会更新它的状态


嗯，差不多就这么多吧，有时间再深入研究研究！

--EOF--
