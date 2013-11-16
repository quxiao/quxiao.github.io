---
author: admin
comments: true
date: 2011-02-24 14:38:51+00:00
layout: post
slug: linux-api-practice-the-process-of-resource-constraints
title: Linux API 实践：限制进程资源
wordpress_id: 206
categories:
- Development
tags:
- linux
- setrlimit
---

[上一篇](http://www.qxavier.me/2011/02/18/linux-api-practice-access-to-the-process-of-resource-constraints/)文章，讲到了如何用getrlimit获取进程资源上限。既然能getrlimit，那当然我们也可以setrlimit，对进程的资源进行限制。我们就以进程的CPU时间和占用的内存为例，讲述如何限制进程相应的资源以及当资源上限超过时，是如何捕捉的。

一个对子进程的资源（时间和内存）进行限制的程序：

    
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
    
    #define SYSTEM_ERROR		10
    
    #define MIN_TIME_LIMIT 1
    #define MAX_TIME_LIMIT 100
    #define MIN_MEM_LIMIT 4
    #define MAX_MEM_LIMIT 1024
    
    struct rlimit tLimit, mLimit;
    
    int main (int argc, char** argv)
    {
    	int timeLimit, memLimit, ret;
    	pid_t pid;
    	int s;
    	struct rusage rUsage;
    	double passTime;
    	char** childCmd;
    
    	if ( argc < 4 )
    	{
    		printf("argument number error!\n");
    		printf("usage: timeLimt(%d~%ds) memoryLimit(%d~%dMb) commandLine\n",
    				MIN_TIME_LIMIT, MAX_TIME_LIMIT, MIN_MEM_LIMIT, MAX_MEM_LIMIT);
    		return SYSTEM_ERROR;
    	}
    	//获取时间限制和内存限制
    	if ( isdigit(argv[1][0]) )
    		timeLimit = atoi(argv[1]);
    	if ( isdigit(argv[2][0]) )
    		memLimit = atoi(argv[2]);
    	if ( timeLimit < MIN_TIME_LIMIT || timeLimit > MAX_TIME_LIMIT )
    	{
    		printf("time limit argument error!(%d~%ds)\n", MIN_TIME_LIMIT, MAX_TIME_LIMIT);
    		return SYSTEM_ERROR;
    	}
    	if ( memLimit < MIN_MEM_LIMIT || memLimit > MAX_MEM_LIMIT )
    	{
    		printf("memory limit argument error!(%d~%dMb)\n", MIN_MEM_LIMIT, MAX_MEM_LIMIT);
    		return SYSTEM_ERROR;
    	}
    	memLimit = memLimit * 1024 * 1024;
    
    	if ( (pid=fork()) < 0 )
    	{
    		printf("fork error!\n");
    		return SYSTEM_ERROR;
    	}
    	else if ( pid == 0 )		/*child process*/
    	{
    		getrlimit(RLIMIT_CPU, &tLimit);
    		getrlimit(RLIMIT_AS, &mLimit);
    
    		tLimit.rlim_cur = timeLimit;
    		mLimit.rlim_cur = memLimit;
    
    		if ( setrlimit(RLIMIT_CPU, &tLimit) == -1 )		/*设置时间限制*/
    			printf("setrlimit time limit error!\n");
    		if ( setrlimit(RLIMIT_AS, &mLimit) == -1 )		/*设置内存限制*/
    			printf("setrlimit memory limit error!\n");
    
    		childCmd = argv + 3;
    		execvp(childCmd[0], childCmd);
    	}
    	else
    	{
    		waitpid(pid, &s, 0);		//等待子进程结束，并获取信号
    		printf("================parent process=====================\n");
    		printf("child exit status: %d\n", s);
    		if ( WIFEXITED(s) )
    			printf("exit normally.\n");
    		else
    			printf("exit abnormally.\n");
    		if ( WIFSIGNALED(s) )
    			printf("exit by signal: %d | %d\n", s, WTERMSIG(s));
    		printf("parent process is finished.\n");
    		getrusage(RUSAGE_CHILDREN, &rUsage);
    		passTime = (rUsage.ru_utime.tv_sec + rUsage.ru_stime.tv_sec) * 1000 + (float)(rUsage.ru_stime.tv_usec + rUsage.ru_utime.tv_usec) / 1000;
    		printf("child process time: %d ms\n", (int)passTime);
    		printf("===================================================\n\n");
    		return 0;
    	}
    }


好了，下面我们写几个测试程序。

第一个，写个占用CPU时间很长的程序：

    
    #include <stdio.h>
    #include <unistd.h>
    
    const int MAX = 50000;
    
    int main ()
    {
    	int i, j, k;
    	for (i = 0; i < MAX; i ++)
    		for (j = 0; j < MAX; j ++)
    			k = i * j;
    	return 0;
    }


测试运行结果：


    > quxiao@quxiao-laptop:~/learnLinux/setrlimit$ ./setrlimit_ex.o 1 64 ./run_so_long.o

    ================parent process=====================

    child exit status: 24

    exit abnormally.

    exit by signal: 24 | 24

    parent process is finished.

    child process time: 1000 ms

    ===================================================


信号24是**SIGXCPU**信号，表示超出CPU时间上限（我们的程序中为1s）。

好，再写个申请过多内存空间的程序：

    
    #define MAX 10000
    
    int arr[MAX][MAX];
    
    int main ()
    {
    	return 0;
    }


测试运行结果：


    > quxiao@quxiao-laptop:~/learnLinux/setrlimit$ ./setrlimit_ex.o 1 64 ./so_long_array.o

    ================parent process=====================

    child exit status: 9

    exit abnormally.

    exit by signal: 9 | 9

    parent process is finished.

    child process time: 0 ms

    ===================================================


信号9表示**SIGKILL**，申请过多内存空间时会产生该信号。（还有哪些情况会产生这个信号？）

这样，我们就可以对进程的行为进行限制，并测量进程的各项资源了。（这也是我们的OJ所需要的 :) ）

具体所有信号所表示的含义，可以查看[这里](http://linux.die.net/man/7/signal)。
