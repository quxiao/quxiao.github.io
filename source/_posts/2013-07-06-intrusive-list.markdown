---
author: admin
comments: true
date: 2013-07-06 12:47:21+00:00
layout: post
slug: intrusive-list
title: Intrusive list
wordpress_id: 767
categories:
- Data Structure
- Essay
tags:
- intrusive list
---

链表，应该是每一个学编程的人都会接触到的经典数据结构，大部分计算机专业的同学至少在上学期间也实现过单向、双向链表。在我的印象里，链表一般都是这么表示的：
    
    struct LinkNode
    {
        LinkNode* prev;
        LinkNode* next;
        int my_data1;
        double my_data2;
        //...
    };
    
    class LinkList
    {
        LinkNode root;
        //...
    };


或者，我们就干脆直接是用`std::list<T>`。无论是哪一种，它都是将使用的数据“嵌入”到链表的节点中，或者说是节点包在数据之外。这种实现也是教科书里面讲到的那个实现，也是我之前知道的唯一的实现方式。

前段时间看网上的几篇博客（[Tips of Linux C programming](http://rdc.taobao.com/blog/cs/?p=1675)，[Avoiding game crashes related to linked lists](http://www.codeofhonor.com/blog/avoiding-game-crashes-related-to-linked-lists)），发现原来还有另外一种更优雅的链表实现——intrusive list。Linux kernel，以及许多底层软件（另外还有游戏星际争霸）的开发中，都是使用intrusive list进行链表操作的。

所谓intrusive list，就是指不像上面说的那样将链表节点包在数据外面，而是将链表节点包在数据里面。例如下面这样：

    
    struct list_node_t
    {
        list_node_t* prev;
        list_node_t* next;
    };
    
    struct MyClass
    {
        int data;
        list_node_t node;
    };


这样做，是为了让节点和数据在一个数据结构中。因为许多场景下，链表包含的数据是指针（考虑到数据拷贝构造的代价，以及一份数据被多个链表共享的情况），例如`std::list<MyClass*>`。使用intrusive list实现，就可以省去了"节点 -> 数据指针 -> 数据"的二次查找。找到intrusive list中的一个节点后，就可以立即找到这个节点对应的数据，即我知道一个`list_node_t`的地址，我改如何找到这个`list_node_t`对应的`MyClass`的`data`呢？这里用到了一段很优美的宏：

    
    #define GET_ENTRY(ptr, type, member)\
        ((type *)((char *)(ptr)-(unsigned long)(&((type *)0)->member)))


第一眼肯定觉得晕，不急，我们一步一步来。首先看参数：



	
  * ptr: `list_node_t`的地址

	
  * type: 包含这个`list_node_t`的类型，对应上面的例子，就是`Myclass`

	
  * member：这个`list_node_t`类型变量在MyClass中的变量名，需要这个得到改变量的offset


首先看

    
    (&((type *)0)->member)


这个的意思是：如果有个type类型的变量，它的地址是0，那type类型中的member变量的地址是多少，其实就是这个member变量距离所属的type类型变量的偏移量。

然后是

    
    ((char *)(ptr)-(unsigned long)(&((type *)0)->member))


这就很清楚了，知道了偏移量，我又知道了真正的member变量的地址（ptr），用ptr的地址减去ptr的偏移量，就得到了ptr所在的type变量的地址了，然后就可以获取type变量上的数据了。

自己简单实现了intrusive list，再和`std::list`做一个简单的性能测试，对一个int数据的链表插入，结果是：`std::list push_back`的操作操作时间基本上都在intrusive list的2倍以上！这么看来，intrusive list的优势还是挺明显的。


    QuXiaos-MacBook-Pro:intrusive_list quxiao$ ./a.out 1000
    std_list: 0.129 ms
    intrusive_list: 0.059 ms
    QuXiaos-MacBook-Pro:intrusive_list quxiao$ ./a.out 10000
    std_list: 1.297 ms
    intrusive_list: 0.550 ms
    QuXiaos-MacBook-Pro:intrusive_list quxiao$ ./a.out 100000
    std_list: 11.961 ms
    intrusive_list: 4.901 ms
    QuXiaos-MacBook-Pro:intrusive_list quxiao$ ./a.out 1000000
    std_list: 116.855 ms
    intrusive_list: 53.086 ms
    QuXiaos-MacBook-Pro:intrusive_list quxiao$ ./a.out 10000000
    std_list: 1061.955 ms
    intrusive_list: 544.419 ms


intrusive list的实现以及测试的代码如下：

    
{% highlight cpp linenos %}
    #include <stdlib.h>
    #include <stdio.h>
    #include <time.h>
    #include <list>
    
    struct list_node_t
    {
        list_node_t* prev;
        list_node_t* next;
    };
    
    struct MyClass
    {
        int data;
        list_node_t node;
    };
    
    void _list_add(list_node_t* cur, list_node_t* prev, list_node_t* next)
    {
        prev->next = cur;
        next->prev = cur;
        cur->prev = prev;
        cur->next = next;
    }
    
    void list_add_head(list_node_t* cur, list_node_t* head)
    {
        _list_add(cur, head, head->next);
    }
    
    void list_add_tail(list_node_t* cur, list_node_t* head)
    {
        _list_add(cur, head->prev, head);
    }
    
    void list_del(list_node_t* cur)
    {
        cur->prev->next = cur->next;
        cur->next->prev = cur->prev;
        cur->prev = cur->next = NULL;
    }
    
    #define GET_ENTRY(ptr, type, member)\
        ((type *)((char *)(ptr)-(unsigned long)(&((type *)0)->member)))
    
    #define INIT_NODE(ptr)\
        (ptr)->next = (ptr)->prev = ptr;
    
    #define list_for_each(pos, head) \
            for (pos = (head)->next; pos != (head); \
                            pos = pos->next)
    
    #define list_for_each_safe(pos, n, head) \
            for (pos = (head)->next, n = pos->next; pos != (head); \
                        pos = n, n = pos->next)
    
    void test_std_list(int run_num)
    {
        int t1 = clock();
        std::list<int> std_list;
        for (int i = 0; i < run_num; i ++)
        {
            std_list.push_back(i);
        }
    
        printf("std_list: %.3lf ms\n", double(clock() - t1) / CLOCKS_PER_SEC * 1000);
    }
    
    void test_intrusive_list(int run_num)
    {
        int t1 = clock();
        list_node_t list_head;
        INIT_NODE(&list_head);
    
        for (int i = 0; i < run_num; i ++)
        {
            MyClass* c = (MyClass*) malloc(sizeof(MyClass));
            c->data = i;
            list_add_tail(&c->node, &list_head);
        }
        printf("intrusive_list: %.3lf ms\n", double(clock() - t1) / CLOCKS_PER_SEC * 1000);
    }
    
    int main(int argc, char** argv)
    {
        if ( argc != 2 )
        {
            fprintf(stderr, "argc is not 2");
            return -1;
        }
        int run_num = atoi(argv[1]);
        if ( 0 == run_num )
        {
            fprintf(stderr, "argv[%s] is not num", argv[1]);
            return -1;
        }
    
        test_std_list(run_num);
        test_intrusive_list(run_num);
    
        return 0;
    }

{% endhighlight %}

另外，intrusive list还有几个特点，比如：

	
  * 移植性好，不像数据包在链表里面的实现，要么每种链表类型都写重复的代码，要么就使用template，但只能在C++中使用

	
  * 容易使用，你需要使用哪种类型的列表，就在这种类型中添加节点即可

	
  * 可以方便实现一份数据被多个链表共享的情况，有几个链表，就在类型下添加几个节点变量即可，多个链表直接不会互相干扰


要不是看了那几篇博客，我对链表的认识还停留在教科书上，好歹也看了几本数据结构的书，其中不乏经典书籍，但从没有听说过intrusive list这种实现，看来很多知识，并不是光靠看书就能获取的。

参考材料：

	
  * [Strategy: Stop Using Linked-Lists](http://highscalability.com/blog/2013/5/22/strategy-stop-using-linked-lists.html)

	
  * [Avoiding game crashes related to linked lists](http://www.codeofhonor.com/blog/avoiding-game-crashes-related-to-linked-lists)

	
  * [Tips of Linux C programming](http://rdc.taobao.com/blog/cs/?p=1675)

	
  * [Linux Kernel Linked List Explained](http://isis.poly.edu/kulesh/stuff/src/klist/)




--EOF--
