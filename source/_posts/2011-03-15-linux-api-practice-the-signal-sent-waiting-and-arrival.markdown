---
author: admin
comments: true
date: 2011-03-15 13:07:15+00:00
layout: post
slug: linux-api-practice-the-signal-sent-waiting-and-arrival
title: Linux API实践：信号的发出、等待以及到达
#wordpress_id: 235
categories:
- Development
tags:
- linux
- signal
- sigpending
---

信号是Linux中的重要概念，其表示着某种事件的发生（或者结束）。

信号可以用三种状态：产生（generated）、等待（pending）以及到达（delivered）。这三种状态是按时间顺序出现的，我们尤其要注意pending状态，因为并不是信号一旦产生，就会立即到达目标进程，信号在产生和到达中间的状态就为等待。

进程可以对到达的信号进行阻止（block），如果被阻止的信号到达进程，该信号的状态就会一直保持等待，直到

  * 进程解除对该信号的阻止，或者
  * 进程忽略（ignore）到该信号


如果被阻止的信号在解除阻止之前产生了多次，多个相同种信号会进行队列处理（queued）吗？答案是大部分的UNIX系统都不会将多个同种信号进行队列处理，而是只算一次。（貌似在POSIX.1上增加实时扩展功能才能支持多个同种pending信号的队列化处理。）话不多说，实践一下吧！

程序1简单的向自身发出了多次SIGUSR1信号，并且在信号处理函数中记录了发生的次数：

```cpp
    #include <stdio.h>
    #include <signal.h>
    #include <stdlib.h>
    
    static void sigusr1_handler(int signo);
    
    int main ()
    {
    	sigset_t oldmask, newmask, pendmask;
    
    	if ( signal(SIGUSR1, sigusr1_handler) == SIG_ERR )
    	{
    		printf("signal error\n");
    		exit(1);
    	}
    
    	int i;
    	for (i = 0; i < 5; i ++)
    	{
    		raise(SIGUSR1);
    		printf("send SIGUSR1\n");
    	}
    
    	printf("sleep started\n");
    	sleep(5);
    	printf("sleep finished\n");
    
    	exit(0);
    }
    
    static void sigusr1_handler(int signo)
    {
    	static int n = 1;
    	printf("receive user1 signal %d times\n", n ++);
    	if ( signal(SIGUSR1, sigusr1_handler) == SIG_ERR )
    	{
    		printf("can't reset SIGUSR1");
    		exit(1);
    	}
    }
```


程序的结果应该很明显：

    receive user1 signal 1 times
    send SIGUSR1
    receive user1 signal 2 times
    send SIGUSR1
    receive user1 signal 3 times
    send SIGUSR1
    receive user1 signal 4 times
    send SIGUSR1
    receive user1 signal 5 times
    send SIGUSR1
    sleep started
    sleep finished


程序2在程序1的基础上，增加了发出信号之前阻止了该信号，以及之后再解除阻止的操作：

``` cpp
    #include <stdio.h>
    #include <signal.h>
    #include <stdlib.h>
    
    static void sigusr1_handler(int signo);
    
    int main ()
    {
    	sigset_t oldmask, newmask, pendmask;
    
    	//加入SIGUSR1信号处理函数
    	if ( signal(SIGUSR1, sigusr1_handler) == SIG_ERR )
    	{
    		printf("signal error\n");
    		exit(1);
    	}
    
    	//阻止SIGUSR1信号
    	sigemptyset(&newmask);
    	sigaddset(&newmask, SIGUSR1);
    	if ( sigprocmask(SIG_BLOCK, &newmask, &oldmask) < 0 )
    	{
    		printf("sigprocmask block error\n");
    		exit(1);
    	}
    	printf("SIGUSR1 is blocked\n");
    
    	//发出信号
    	int i;
    	for (i = 0; i < 5; i ++)
    	{
    		raise(SIGUSR1);
    		printf("send SIGUSR1\n");
    	}
    
    	printf("sleep started\n");
    	sleep(5);
    	printf("sleep finished\n");
    
    	if ( sigpending(&pendmask) < 0 )
    	{
    		printf("sigpending error\n");
    		exit(1);
    	}
    	if ( sigismember(&pendmask, SIGUSR1) )
    	{
    		printf("SIGUSR1 was pending\n");
    	}
    
    	//恢复之前的信号掩码
    	if ( sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0 )
    	{
    		printf("sigprocmask setmask error\n");
    		exit(1);
    	}
    	printf("restore old signal mask\n");
    
    	exit(0);
    }
    
    static void sigusr1_handler(int signo)
    {
    	static int n = 1;
    	printf("receive user1 signal %d times\n", n ++);
    	if ( signal(SIGUSR1, sigusr1_handler) == SIG_ERR )
    	{
    		printf("can't reset SIGUSR1");
    		exit(1);
    	}
    }
```


再来看运行的结果：

    SIGUSR1 is blocked
    send SIGUSR1
    send SIGUSR1
    send SIGUSR1
    send SIGUSR1
    send SIGUSR1
    sleep started
    sleep finished
    SIGUSR1 was pending
    receive user1 signal 1 times
    restore old signal mask


虽然发出了多次信号，但由于之前的阻止设置使其发出后处于等待状态，多个同种信号pending，信号处理函数果然只接受到了1次，跟书中讲的是一致的。
