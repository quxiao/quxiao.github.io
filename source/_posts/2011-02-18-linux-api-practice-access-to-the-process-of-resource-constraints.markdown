---
author: admin
comments: true
date: 2011-02-18 13:55:40+00:00
layout: post
slug: linux-api-practice-access-to-the-process-of-resource-constraints
title: Linux API 实践：获取进程资源限制
wordpress_id: 169
categories:
- Development
tags:
- getrlimit
---

进程在运行时，会占用计算机的各种资源，比如CPU时间、内存、文件等等。但是，进程是不可以占用无限多的资源的，操作系统会给进程设定所使用资源的上限。想获取这些资源的上限值，是需要调用getrlimit()即可。

    
    int getrlimit(int resource, struct rlimit *rlptr);


第一个参数是资源，有哪些资源呢？








资源


粗略含义






RLIMIT_AS


进程可使用的内存的最大值






RLIMIT_CORE


核心文件(core file)的最大值






RLIMIT_CPU


CPU时间最大值






RLIMIT_DATA


数据段（已初始化数据+未初始化数据+堆）的最大值






RLIMIT_FSIZE


新建文件的最大字节数






RLIMIT_LOCKS


持有的锁的最大数






RLIMIT_MEMLOCK


锁定内存的最大字节数






RLIMIT_NOFILE


打开文件的最大数目






RLIMIT_NPROC


每个实际用户(real user)的最大子进程数目






RLIMIT_RSS


RSS(Resident Set Size)的最大字节数






RLIMIT_SBSIZE


socket buffer的最大字节数






RLIMIT_STACK


进程栈的最大字节数






RLIMIT_VMEM


与RLIMIT_AS含义一致




第二个参数是rlimit，rlimit结构是这样的：

    struct rlimit

    {

    rlim_t rlim_cur; /* soft limit: current limit */

    rlim_t rlim_max; /* hard limit: maximum value for rlim_cur */

    };

其中含有软限制和硬限制。超级用户可以增加硬限制；一般用户可以降低硬限制，但不能增加硬限制，一般用户还可修改软限制，但修改的软限制不能超过硬限制。

实际运行的效果如何呢？实践一下吧！

    
    #include <stdio.h>
    #include <sys/resource.h>
    
    #define doit(name) pr_limit(#name, name)
    
    void pr_limit(char* name, int resource);
    
    int main ()
    {
    	printf("resource name  soft\thard \n");
    #ifdef  RLIMIT_AS
    	doit(RLIMIT_AS);
    #endif
    	doit(RLIMIT_CORE);
    	doit(RLIMIT_CPU);
    	doit(RLIMIT_DATA);
    	doit(RLIMIT_FSIZE);
    #ifdef  RLIMIT_LOCKS
    	doit(RLIMIT_LOCKS);
    #endif
    #ifdef  RLIMIT_MEMLOCK
    	doit(RLIMIT_MEMLOCK);
    #endif
    	doit(RLIMIT_NOFILE);
    #ifdef  RLIMIT_NPROC
    	doit(RLIMIT_NPROC);
    #endif
    #ifdef  RLIMIT_RSS
    	doit(RLIMIT_RSS);
    #endif
    #ifdef  RLIMIT_SBSIZE
    	doit(RLIMIT_SBSIZE);
    #endif
    	doit(RLIMIT_STACK);
    #ifdef  RLIMIT_VMEM
    	doit(RLIMIT_VMEM);
    #endif
    
    	return 0;
    }
    
    void pr_limit(char* name, int resource)
    {
    	struct rlimit limit;
    	if ( getrlimit(resource, &limit) < 0 )
    	{
    		printf("getrlimit error!\n");
    		return;
    	}
    	printf("%-14s ", name);
    	if ( limit.rlim_cur == RLIM_INFINITY )
    		printf("infinite ");
    	else
    		printf("%8ld ", limit.rlim_cur);
    
    	if ( limit.rlim_max == RLIM_INFINITY )
    		printf("infinite ");
    	else
    		printf("%8ld ", limit.rlim_max);
    	putchar('\n');
    }


运行的结果：

    
    resource name  soft	hard
    RLIMIT_AS      infinite infinite
    RLIMIT_CORE           0 infinite
    RLIMIT_CPU     infinite infinite
    RLIMIT_DATA    infinite infinite
    RLIMIT_FSIZE   infinite infinite
    RLIMIT_LOCKS   infinite infinite
    RLIMIT_MEMLOCK    65536    65536
    RLIMIT_NOFILE      1024     1024
    RLIMIT_NPROC   infinite infinite
    RLIMIT_RSS     infinite infinite
    RLIMIT_STACK    8388608 infinite
