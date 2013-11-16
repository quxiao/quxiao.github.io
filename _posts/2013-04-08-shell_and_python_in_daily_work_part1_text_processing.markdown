---
author: admin
comments: true
date: 2013-04-08 14:44:54+00:00
layout: post
slug: shell_and_python_in_daily_work_part1_text_processing
title: 日常工作中用到的Shell/Python命令——文本篇
wordpress_id: 715
categories:
- Development
tags:
- python
- shell
---

作为一名RD，作为一名后端RD，每天大部分的工作都是在终端上面完成的，除了在vim上面编写代码，也需要分析日志、数据，定时跑一些任务，因此各种shell、各种脚本语言也是不少用。罗列些自己平时工作中用到查看、处理文本的命令：


## 查看日志


平常查看模块打印的日志，肯定是必需的，比如需要知道一个模块重启后运行是否正常，需要查看请求返回的一些字段。日志比较小的话（100Mb以内），可以直接用`vim`打开，否则会把服务器内存全部吃掉，甚至把服务器搞挂。。。

文件太大的话，可以用`less`或者`more`命令打开，这样不会把整个文件都load到内存，然后再使用“/”或者"?"向后或者向前查找需要的字符串

有时只需要看看文件的头尾几行，看看日志的格式如何，用`head`以及`tail`就可以，特别，如果需要实时查看日志，则可以使用`tail -f`。


## 过滤日志


有时只需要获取日志的某些行，比如查找哪些行包含"FATAL"，就可以用

`grep "FATAL" xxx.log`

如果先是想看看有没有这类日志存在，就用管道结合`wc -l`

`grep "FATAL" xxx.log | wc -l`

也可以结合之前的`less`、`head`、`tail`查看部分日志

`grep "FATAL" xxx.log | less`

如果是想获取日志中的某些字段，可以使用`cut`命令进行分割

假设有如下日志：


> 1 2 3 4 5

11 12 13 14 15


获取第三列，用`cat xxx.log | cut -d'\t' -f3`，就可以得到


> 3

13


有时候，日志比较复杂，比如：


> a=1,b=2,c=3


比如还是获取第三列，则可以：

`cat xxx.log | cut -d',' -f3 | cut -d'=' -f2`

过滤日志，也可以使用`awk`，但感觉如何不是对于日志的数据进行计算，只用`awk`进行过滤，未免有些大材小用了，呵呵。


## 计算日志字段


好，这个时候`awk`就可以排上用场了，它不但能对字段进行过滤、累加、对日志按类似C语言`printf()`的格式进行输出，还可以进行dict运算。比如有一份日志保存着每次请求的query字段：


> htc

iphone4s

htc

sumsung

xiaomi


我需要知道每个query的频次，用几行python应该也能很快搞定，不过用`awk`，仅需用一行：

    
    cat xxx.log | awk '{dict[$1]++;} END{for(i in dict){printf "query:[%s] pv:[%d]\n", i, dict[i];}}'




> query:[xiaomi] pv:[1]

query:[sumsung] pv:[1]

query:[htc] pv:[2]

query:[iphone4s] pv:[1]




## 遍历文件


有时候需要依次对一些文件进行相同的处理，如果只是想遍历文件一个文件夹下面的某些文件，可以直接

    
    for file in ./*.log; do echo "$file"; done




> ./zhixin_stat.log
./zhixin_stat.py.log


有时候需要构造一些序列，一般情况，用类似C语言的for循环，比如：

    
    for ((i=0; i<=10; i++)); do echo 201304$i; done




> 2013040
2013041
2013042
2013043
2013044
2013045
2013046
2013047
2013048
2013049
20130410


但是可以发现，生成的序列宽度是不一样的，就需要用到seq命令了，它可能生成固定宽度的序列：

    
    for i in `seq -w 1 10`; do echo 201304$i; done




> 20130401
20130402
20130403
20130404
20130405
20130406
20130407
20130408
20130409
20130410




## 用python进行排序


统计数据，难免需要对某几个维度进行分析，在python中进行排序，无非就是对于一个`list`进行排序，`list`中的元素一般为某个`class`或者一个`dict`。

对于排序class，需要定义'<', '>', '<=', '>=', '==', '!='六种比较行为，就好像C++中的operator重载一样。具体Python doc中这篇[HowTo](http://docs.python.org/2/howto/sorting.html)讲解的很清楚，这里不再赘述了。

对于dict进行排序，如果需要按照dict的某个key进行排序，则可以这么写：

    
    from operator import itemgetter
    #按照name字段进行排序  
    newlist = sorted(list_to_be_sorted, key=itemgetter('name'))




## 用python发送邮件


好了，文本统计完了，生成了一份统计报表，这时候可能就需要我们通过邮件的形式发送给同事看。用`crontab`每天凌晨开始抓数据、跑统计、然后把统计数据邮件给大家，显得专业！

发送邮件的功能，可以用python + sendmail实现：

    
    from os import path
    import datetime
    import time
    import sys
    import subprocess
    import smtplib
    from email.mime.text import MIMEText
    from email.mime.multipart import MIMEMultipart
    from email.mime.application import MIMEApplication
    
    sender = 'quxiao@somewhere.com'
    receiver = 'someone@somewhere.com'
    
    def main(date):
        EmailBody = 'blablabla'
        msg = MIMEMultipart()
        msg['Subject'] = 'some subject'
        msg['From'] = sender
        msg['To'] = receiver
        msg.attach(MIMEText(EmailBody,'html','GB18030'))
        #send
        proc = subprocess.Popen(['/usr/lib/sendmail','-t'],stdin = subprocess.PIPE)
        proc.stdin.write(str(msg))
        proc.stdin.close()


--EOF--
