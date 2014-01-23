---
layout: post
title: "模板元编程中的Typelist"
description: ""
category: ""
tags: ["typelist", "template metaprogramming"]
---
{% include JB/setup %}

前段时间在看其他同事的代码，无意间看了类似下面的代码：

{% highlight cpp linenos %}
    Type< FilterA,
    Type< FilterB,
    Type< FilterC,
    Type< FilterD,
    Type< FilterE,
        void > > > > > Filters;
{% endhighlight %}

感觉代码风格略微诡异了些，后来查了些资料，得知这就是我一直心仪已久的模板元编程中的一种技巧——** Typelist** 。

当你需要对于大量的类型进行相似的操作时，比如对于返回的商品进行五种类型的过滤，许多（手工写的）代码都是重复、冗余的，代码可能是这样的：

{% highlight cpp linenos %}
    Goods* goods = get_goods(goods_id);
    if (goods_blacklist.filter(goods)) {
        //...
    } else if (cate_blacklist.filter(goods)) {
        //...
    } else if (stock_blacklist.filter(goods)) {
        //...
    }
    //...
{% endhighlight %}

显然代码是很冗余的，再抽象一下，也许可以写成这样：

{% highlight cpp linenos %}
    vector<FilterBase*> filters;
    filters.push_back(new GoodsBlacklist());
    filters.push_back(new CateBlacklist());
    filters.push_back(new StockBlacklist());
    //...

    Goods* goods = get_goods(goods_id);
    for (size_t i = 0; i < filters.size(); i ++) {
        if (filters[i]->do_filter(goods)) {
            break;
        }
    }

{% endhighlight %}

代码整洁了一些，不过还是需要程序在运行时`new`出来各种类、加入到集合中、然后还要考虑调用虚函数的成本，仍然不是高效的。这个时候，Typelist就派上用场了，他可以让编译器帮你自动生成许多类（而不是类的实例）的集合，并且可以像链表一样对这些类进行遍历，从而达到自动进行相似、重复工作的效果。

# 定义Typelist

首先来看看如何定义一个Typelist吧

{% highlight cpp linenos %}
    template<typename H, typename T>
    struct Type
    {
        typedef H Head;
        typedef T Tail;

        H* value_;      //current type
    };
{% endhighlight %}

首先，把这些类的集合看做是一个链表，这个链表的类型叫做`Type`，当然也可以是其它名称。假设有N个类型，链表的第0个类型是H，后面的`1 ~ N-1`组成一个联合类型T。接着，我们需要考虑一些特殊情况，这时候就需要用到**模板特化**了。首先，如果`1 ~ N-1`也是`Type`类型的时候，该如何定义呢？

{% highlight cpp linenos %}
    template<typename H, typename S, typename T>
    struct Type<H, Type<S, T> > : public Type<S, T>
    {
        typedef H Head;
        typedef Type<S, T> Tail;

        H* value_;
    };
{% endhighlight %}

这样，就可以表示当Type的后一个模板参数也是一个Type类型时，Type类型应该是个什么样子，也就可以让Type类型无限延展下去了。最后，还需要考虑Type链表的最后一个元素应该如何定义。链表里面表示最后一个元素可以用`NULL`，那么我们在这边可以用一个没有任何意义的`NullType`类型、或者简单使用`void`表示Type链表的终结。

{% highlight cpp linenos %}
    template<typename H>
    struct Type<H, void>
    {
        typedef H Head;
        typedef void Tail;

        H* value_;
    };
{% endhighlight %}

上面三种定义就覆盖了Typelist中所以可能的情况，当我们写出类似`Type< FilterA, Type< FilterB, Type< FilterC, void > > >`的时候，编译器就能自动帮我们进行匹配，并且自动生成一种类型！
