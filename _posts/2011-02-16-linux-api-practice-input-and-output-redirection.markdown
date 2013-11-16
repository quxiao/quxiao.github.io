---
author: admin
comments: true
date: 2011-02-16 11:51:15+00:00
layout: post
slug: linux-api-practice-input-and-output-redirection
title: Linux API 实践：输入输出重定向
wordpress_id: 161
categories:
- Development
tags:
- dup
---

在shell中对程序进行重定向很简单，用<和>符号就可以了，但在自己的程序中怎么实现输入输出重定向呢？

先来看看Linux内核中，文件（还是设备）是通过哪些数据结构保存的：

[![image](http://www.qxavier.me/wp-content/uploads/2011/02/image_thumb1.png)](http://www.qxavier.me/wp-content/uploads/2011/02/image1.png)

每个进程都保存一份文件描述符的表格，每一行又指向file table，然后file table再指向v-node table。v-node table我们可以暂不考虑，暂且把它当作文件的内容。而从process table entry中的file pointer可以指向不同或者相同的file table。原本标准输入输出是指向“键盘”和“屏幕”这两个设备的，如果可以将它们指向我们指定的文件，就可以实现重定向了。

先用open()打开需要重定向到的文件，获取去文件描述符fd，在用dup2()把进程中原先的输入输出文件描述符STDIN_FILENO和STDOUT_FILENO重定向至fd，这样就可以实现输入输出重定向了。我想，shell实现重定向也应该是类似的思想，关键代码如下：

    
    #include <stdio.h>
    #include <stdlib.h>
    #include <ctype.h>
    #include <unistd.h>
    #include <sys/wait.h>
    #include <sys/resource.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <errno.h>
    #include <fcntl.h>
    #include <signal.h>
    
    int main (int argc, char** argv)
    {
    	if ( argc != 3 )
    	{
    		printf("usage: inputFile outputFilen");
    		return 1;
    	}
    	int inFd, outFd;
    
    	inFd = open(argv[1], O_RDONLY);
    	if ( inFd < 0 )
    	{
    		printf("inFd open error!n%sn",strerror(errno));
    		return 1;
    	}
    	outFd = open(argv[2], O_CREAT | O_TRUNC | O_RDWR, S_IRWXU | S_IRGRP | S_IROTH);
    	if ( outFd < 0 )
    	{
    		printf("outFd open error!n%sn", strerror(errno));
    		return 1;
    	}
    
    	if ( dup2(inFd, STDIN_FILENO) < 0 )
    	{
    		printf("inFd dup2 error!n");
    		return 1;
    	}
    	if ( dup2(outFd, STDOUT_FILENO) < 0 )
    	{
    		printf("outFd dup2 error!n");
    		return 1;
    	}		
    
    	char line[128];
    	while ( scanf("%s", &line) != EOF )
    	{
    		printf("%sn", line);
    	}
    
    	return 0;
    }
