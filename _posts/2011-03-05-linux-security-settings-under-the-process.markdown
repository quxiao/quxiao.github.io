---
author: admin
comments: true
date: 2011-03-05 02:00:37+00:00
layout: post
slug: linux-security-settings-under-the-process
title: Linux下进程的安全性设置
wordpress_id: 225
categories:
- Development
tags:
- alarm
- linux
- setrlimit
---

上一次我们讲过，如何用setrlimit限制进程的资源，并以时间和内存为例。这次我们来试试看更多的资源限制，以防止恶意代码的攻击，增加系统的安全性。


## **超时**


超时上次不是讲过了吗？嗯，上次讲的是“忙”的超时，而这次是“sleep”的超时。通过实验发现，如果程序一直咋sleep，父进程是无法检测到子进程超时的，我想这应该是因为sleep的时间本身不算在进程的CPU时间内。那啥办呢？好办，加定时器（alarm）。例如：

    
    signal(SIGALRM, my_alarm_handler);
    alarm(timeLimit * 2);


这样，当进程流逝的时间超过alarm的限制时，父进程也会受到相应的信号。

例如对于这样的代码：

    
    #include <unistd.h>
    
    int main ()
    {
    	sleep(15);
    
    	return 0;
    }


运行结果如下：


> quxiao@quxiao-laptop:~/judge$ ./linux_judge_svn/linux/judge.o /home/quxiao/judge/a_and_b.in /home/quxiao/judge/out.txt 1 64 /home/quxiao/judge/just_sleep.o
input file: /home/quxiao/judge/a_and_b.in
output file: /home/quxiao/judge/out.txt
time limit: 1
memory limit: 67108864
cmd: /home/quxiao/judge/just_sleep.o
================parent process=====================
child exit status: 14
exit abnormally.
exit by signal: 14 | 14
parent process is finished.
child process time: 0 ms
maximum resident set size:    292




## 生成/打开文件


有时我们并不希望子进程有文件操作（当然，除了输入输出设备这些特殊文件），可以通过修改RLIMIT_NOFILE的值来达到此效果。

如果程序会生成一些文件，代码如下：

    
    for (i = 0; i < 10; i ++)
    {
    	char template[] = "template-XXXXXX";
    	fd = mkstemp(template);
    	if ( fd == -1 )
    	{
    		printf("can't create file!\n");
    	}
    }


那么，修改过限制之后的代码运行结果就是：


> quxiao@quxiao-laptop:~/judge$ ./linux_judge_svn/linux/judge.o /home/quxiao/judge/a_and_b.in /home/quxiao/judge/out.txt 1 64 /home/quxiao/judge/too_many_file.o
input file: /home/quxiao/judge/a_and_b.in
output file: /home/quxiao/judge/out.txt
time limit: 1
memory limit: 67108864
cmd: /home/quxiao/judge/too_many_file.o
================parent process=====================
child exit status: 0
exit normally.
parent process is finished.
child process time: 0 ms
maximum resident set size:    364
===================================================

quxiao@quxiao-laptop:~/judge$ cat out.txt
can't create file!
can't create file!
can't create file!
can't create file!
can't create file!
can't create file!
can't create file!
can't create file!
can't create file!


不过，从例子中可以看出，表面上父进程并没有检测到子进程的任何异常，但我们可以从输出文件中看到子进程的生成文件是受到限制的。另外，还有一点值得考虑，那就是子进程还是生成了一个文件的，RLIMIT_NOFILE到底应该设置成多少，还要再研究研究。


## 生成子进程


同样，让进程可以任意地生成子进程也是很危险的。RLIMIT_NPROC可以限制生成子进程的数目。对于如下代码：

    
    	for (i = 0; i < 3; i ++)
    	{
    		pid_t pid;
    		if ( (pid=fork()) == -1 )
    		{
    			printf("error!\n");
    		}
    		else if ( pid == 0 )
    		{
    			printf("this is child process\n");
    		}
    		else
    		{
    			printf("this is parent process\n");
    		}
    	}


运行结果如下：


> quxiao@quxiao-laptop:~/judge$ ./linux_judge_svn/linux/judge.o /home/quxiao/judge/a_and_b.in /home/quxiao/judge/out.txt 1 64 /home/quxiao/judge/too_many_child.o
input file: /home/quxiao/judge/a_and_b.in
output file: /home/quxiao/judge/out.txt
time limit: 1
memory limit: 67108864
cmd: /home/quxiao/judge/too_many_child.o
================parent process=====================
child exit status: 0
exit normally.
parent process is finished.
child process time: 0 ms
maximum resident set size:    340
===================================================

quxiao@quxiao-laptop:~/judge$ cat out.txt
error!
error!
error!




## runtime error


当程序出现一些运行时错误的时候，父进程也可以通过信号检测到。这里其实跟资源限制没太大关系，但检测这方面的信息也是很重要的。举2个例子，1个是除0，1个是数组越界。

除0程序：

    
    int main ()
    {
    	int a, b;
    	a = 1;
    	b = 0;
    	a = a / b;
    
    	return 0;
    }




> quxiao@quxiao-laptop:~/judge$ ./linux_judge_svn/linux/judge.o /home/quxiao/judge/a_and_b.in /home/quxiao/judge/out.txt 1 64 /home/quxiao/judge/divide0.o
input file: /home/quxiao/judge/a_and_b.in
output file: /home/quxiao/judge/out.txt
time limit: 1
memory limit: 67108864
cmd: /home/quxiao/judge/divide0.o
================parent process=====================
child exit status: 8
exit abnormally.
exit by signal: 8 | 8
parent process is finished.
child process time: 0 ms
maximum resident set size:    292
===================================================




数组越界程序：

    
    const int MAX = 100;
    
    int main ()
    {
    	int array[MAX];
    	int i;
    	i = 0;
    	while ( ++i )	/* so dangerous! */
    	{
    		array[i] = 0;
    	}
    
    	return 0;
    }




> quxiao@quxiao-laptop:~/judge$ ./linux_judge_svn/linux/judge.o /home/quxiao/judge/a_and_b.in /home/quxiao/judge/out.txt 1 64 /home/quxiao/judge/re.o
input file: /home/quxiao/judge/a_and_b.in
output file: /home/quxiao/judge/out.txt
time limit: 1
memory limit: 67108864
cmd: /home/quxiao/judge/re.o
================parent process=====================
child exit status: 11
exit abnormally.
exit by signal: 11 | 11
parent process is finished.
child process time: 0 ms
maximum resident set size:    288
===================================================
