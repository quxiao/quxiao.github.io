---
author: admin
comments: true
date: 2013-05-31 01:30:39+00:00
layout: post
slug: memcached_source_code_study_thread_model
title: Memcached源码学习——线程模型
wordpress_id: 750
categories:
- Source Reading
tags:
- memcached
---

Memcached中有以下几类线程：

  * 主线程
  * 工作线程
  * 维护线程


主线程，又可以叫分发线程，除了完成程序的各种参数、以及其他线程的初始化以外，还会listen端口，新建连接，并且将该连接分发到其他的工作线程。

工作线程，大部分实际的工作都是他们干的，包括读取请求的协议内容、解析、进行具体存、取、更新、删除kv的操作，最后返回结果。

另外，还有一个维护线程，它的工作就是在需要的时候（存放的item大于总量的2/3）对hash表进行扩展。

Memcached处理请求时，采用的是单进程多线程的Master-Worker模型，通过libevent这个事件响应库来实现的。

首先来看一下主线程和工作线程之间是怎么交互的吧：

工作线程在初始化的时候，会建立一个pipe（管道），两端分别为：`notify_receive_fd`，以及`notify_send_fd`：

``` cpp
    for (i = 0; i < nthreads; i++) {
        int fds[2];
        if (pipe(fds)) {
            perror("Can't create notify pipe");
            exit(1);
        }

        threads[i].notify_receive_fd = fds[0];
        threads[i].notify_send_fd = fds[1];

        setup_thread(&threads[i]);
        /* Reserve three fds for the libevent base, and two for the pipe */
        stats.reserved_fds += 5;
    }
```

也就是说当其他线程向`notify_send_fd`文件描述符写内容的时候，`notify_receive_fd`就可以接受到。

接着，就用到了libevent的API：

``` cpp
    me->base = event_init();

    /* Listen for notifications from other threads */
    event_set(&me->notify_event, me->notify_receive_fd,
              EV_READ | EV_PERSIST, thread_libevent_process, me);
    event_base_set(me->base, &me->notify_event);

    if (event_add(&me->notify_event, 0) == -1) {
        fprintf(stderr, "Can't monitor libevent notify pipe\n");
        exit(1);
    }
```


每个工作线程都新建一个libevent实例(`me->base`)，并且将`notify_event`绑定在这个实例上。

`notify_event`什么时候触发？
当`notify_receive_fd`有内容的时候被触发。

触发了执行什么函数？
执行`thread_libevent_process (me) `函数。

那在哪个地方会写`notify_send_fd`呢？
在主线程将新建的连接分发给工作时，就会向某个线程的`notify_send_fd`写一个空的字符串用来唤醒这个线程。下面的代码一目了然：

``` cpp
    void dispatch_conn_new(int sfd, enum conn_states init_state, int event_flags,
                           int read_buffer_size, enum network_transport transport) {
        CQ_ITEM *item = cqi_new();
        /*这就是所谓的round robin*/
        int tid = (last_thread + 1) % settings.num_threads;
    
        LIBEVENT_THREAD *thread = threads + tid;
    
        last_thread = tid;
    
        item->sfd = sfd;
        /* ... */
    
        cq_push(thread->new_conn_queue, item);
    
        if (write(thread->notify_send_fd, "", 1) != 1) {
            perror("Writing to thread notify pipe");
        }
    }
```


首先来看看主线程，当他把其他工作线程、维护线程启起来之后，就开始侦听socket端口了（可以在memcached的源码中看出tcp和udp在处理逻辑上有很多不同的地方，但我不知道为什么不一样，就只看了处理tcp部分的代码，看来改补一补网络通信的知识了……），主要逻辑在`server_sockets`函数中：

``` cpp
    if (IS_UDP(transport)) {
        int c;

        for (c = 0; c < settings.num_threads_per_udp; c++) {
            /* this is guaranteed to hit all threads because we round-robin */
            dispatch_conn_new(sfd, conn_read, EV_READ | EV_PERSIST,
                              UDP_READ_BUFFER_SIZE, transport);
        }
    } else {
        if (!(listen_conn_add = conn_new(sfd, conn_listening,
                                         EV_READ | EV_PERSIST, 1,
                                         transport, main_base))) {
            fprintf(stderr, "failed to create listening connection\n");
            exit(EXIT_FAILURE);
        }
        listen_conn_add->next = listen_conn;
        listen_conn = listen_conn_add;
    }
```


`conn_new`函数建立了连接之后，将socket文件描述符于`event_handler`函数绑定，当有socket请求过来的时候，就执行`event_handler`。在`event_handler`中，就是直接调用`drive_machine`这个大大的状态转移函数，一次连接的所有状态就都在这个函数里面处理了。

主线程首先达到`drive_machine`中的`conn_listenning`状态，然后通过`dispatch_conn_new`将这次的连接分配给某个工作线程，工作线程再经历其他的状态，完成一次请求。

`drive_machine`的具体逻辑比较复杂，这里就不讲了。

好，最后简单回顾下这个工作流程：

  1. 主线程接受到memcached客户端的请求（是在哪儿一直接收请求的？）
  2. 主线程通过round robin找到一个工作线程
  3. 主线程将创建的连接push到工作线程的连接队列中，然后唤醒这个工作线程
  4. 工作线程被唤醒后，从自己的线程队列中取出一个连接
  5. 解析请求、对hash表进行相应的操作，写入返回



整体看下来，感觉memcached的源码并没有什么高深的算法，用的都是很朴素的链表、底层的网络通信、线程间互斥、字符串解析等等，感觉除了网络通信比较繁琐以外，其他的地方都是一个刚毕业的计算机专业的学生可以也应该掌握的。通过对一些简单、实用的东西进行有效的组合，就可以获取一个功能更强大的合成体。

--EOF--
