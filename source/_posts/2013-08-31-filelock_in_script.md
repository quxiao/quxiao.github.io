---
author: admin
comments: true
date: 2013-08-31 16:04:09+00:00
layout: post
slug: filelock_in_script
title: 在脚本中使用文件锁
wordpress_id: 782
categories:
- Note
tags:
- filelock
---

较早之前，一个模块的reload程序出现过这样一个问题：模块会定期检查一份文件是否被更新，如果被更新了，就reload；但是，如果文件正在被更新（没有写完）并且模块正好检测到更新，模块就会reload到一份不全的文件，导致数据异常。当时的解决方案是：使用一个done文件，推送文件的脚本在完成数据更新后，touch一下done，模块检测done文件的更新来判断是否reload。

最近这个事情又被同们事提起，不过稍微扩展了下，问题变成了：**有没有一种机制，能让许多并行的脚本能串行进行一系列操作**。比如多个脚本对文件进行依次读写，以免产生脏数据。无疑，如果脚本如果shell中有一种“锁”的机制，就可以解决这个问题。原来只使用过API级别的锁，脚本中的锁还真没用过。其实，如果要在shell脚本实现锁，需要满足两个条件：

  1. 一个全局可见的状态
  2. 一种“检测 + 加锁”的原子操作

大家可能会想到使用一个文件来当做锁，如果有这个文件，就表示某个脚本正在操作，其它脚本等待；如果没有这个文件，我就touch这个文件，然后开始我的操作。但是，“检测文件”和“新建文件”不是原子操作，所以是无法保证串行的。

google了一下，还是有几种满足条件的方案的。


# 文件夹锁


检测文件和新建文件无法做到原子性，但是mkdir操作，却能做到原子地检测文件夹和创建文件夹，有点儿意思！所以，当脚本想对于竞争数据进行操作，就`mkdir`某个文件夹，根据返回码得知申请所是否成功，申请成功、完成操作之后再`rm -rf`就可以实现了。例如这样：

    
``` bash
    #!/bin/sh
    
    if [ $# -ne 2 ]
    then
        exit 1
    else
        NUM="$1"
        SLEEP_TIME="$2"
    fi
    
    OUT_FILE="test.out"
    LOCK_FILE="lock.file"
    
    while :
    do
        mkdir ${LOCK_FILE}
        if [ $? -ne 0 ]
        then
            continue
        fi
        for ((i=0; i<10; i++))
        do
            echo "${NUM} is working in $i step!" >> ${OUT_FILE}
        done
        sleep ${SLEEP_TIME}
        rm -rf ${LOCK_FILE}
    done
```


# lockfile命令


原来直接就有个专门用于文件锁的命令，这个命令比mkdir更强大，可以设置申请文件锁的等待时长、重拾次数、锁的过期时间。但是，在写测试代码的时候，却发现只会有一个脚本一直获取到文件锁，其它脚本都处于申请锁等待并超时的状态，难道我参数用的不对？

    
``` bash
    #!/bin/sh
    
    if [ $# -ne 2 ]
    then
        exit 1
    else
        NUM="$1"
        SLEEP_TIME="$2"
    fi
    
    OUT_FILE="test.out"
    LOCK_FILE="lock.file"
    
    while :
    do
        echo "${NUM} try to get lockfile" >> ${OUT_FILE}
        lockfile  ${LOCK_FILE}
        if [ $? -ne 0 ]
        then
            echo "${NUM} wait lock failed" >> ${OUT_FILE}
            continue
        fi
        echo "${NUM} got lockfile" >> ${OUT_FILE}
        for ((i=0; i<10; i++))
        do
            echo "${NUM} is working in $i step!" >> ${OUT_FILE}
        done
        sleep ${SLEEP_TIME}
        echo "${NUM} delete lockfile" >> ${OUT_FILE}
        rm -rf ${LOCK_FILE}
    done
    
    exit 0
```




# 设置noclobber + 重定向文件


shell中有一个参数叫noclobber，设置了这个参数后，当脚本试图重定向文件时，如果发现改文件已经存在，重定向就会失败。这种方法自己没尝试过，下面是从网上抄来的code example：

    
``` bash
    if ( set -o noclobber; echo "locked" > "$lockfile") 2> /dev/null; then
      trap 'rm -f "$lockfile"; exit $?' INT TERM EXIT
      echo "Locking succeeded" >&2
      rm -f "$lockfile"
    else
      echo "Lock failed - exit" >&2
      exit 1
    fi
```


看来在脚本中使用文件锁，还是比较方便的。不过，在我看来在脚本中使用文件锁，有个致命弱点：操作系统不会禁止其它进程对作为锁的文件（或者文件夹）进行操作。相当于一个脚本已经申请到了文件锁并正在操作，但是其它进程完全可以不受限制的删除这个文件锁，这样就会使得期间其它脚本能够成功申请到文件锁。或者一些脚本使用文件锁对竞争资源进行操作，但其它脚本直接操作竞争资源，这种情况也是无法避免的。使用文件锁，完全靠自觉！

另外，文件锁也没有提供像共享锁、排它锁这样的高级功能，这也是文件锁的短板。

参考资料：

[http://en.wikipedia.org/wiki/File_locking](http://en.wikipedia.org/wiki/File_locking)

[Lock your script (against parallel run)](http://wiki.bash-hackers.org/howto/mutex)

--EOF--
