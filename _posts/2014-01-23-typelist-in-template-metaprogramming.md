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
    TypeList< FilterA,
    TypeList< FilterB,
    TypeList< FilterC,
    TypeList< FilterD,
    TypeList< FilterE,
        void > > > > > Filters;
{% endhighlight %}

感觉代码风格略微诡异了些，怎么像是LISP，呵呵。后来查了些资料，得知这就是我一直心仪已久的模板元编程中的一种技巧——** Typelist** 。

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
    struct TypeList
    {
        typedef H Head;
        typedef T Tail;
    };
{% endhighlight %}

首先，把这些类的集合看做是一个链表，这个链表的类型叫做`Type`，当然也可以是其它名称。假设有N个类型，链表的第0个类型是H，后面的`1 ~ N-1`组成一个联合类型T。接着，我们需要考虑一些特殊情况，这时候就需要用到**模板特化**了。首先，如果`1 ~ N-1`也是`Type`类型的时候，该如何定义呢？

{% highlight cpp linenos %}
    template<typename H, typename S, typename T>
    struct TypeList<H, TypeList<S, T> > : public TypeList<S, T>
    {
        typedef H Head;
        typedef TypeList<S, T> Tail;
    };
{% endhighlight %}

这样，就可以表示当Type的后一个模板参数也是一个Type类型时，Type类型应该是个什么样子，也就可以让Type类型无限延展下去了。最后，还需要考虑Type链表的最后一个元素应该如何定义。链表里面表示最后一个元素可以用`NULL`，那么我们在这边可以用一个没有任何意义的`NullType`类型、或者简单使用`void`表示Type链表的终结。

{% highlight cpp linenos %}
    strcut NullType {};

    template<typename H>
    struct TypeList<H, NullType>
    {
        typedef H Head;
        typedef NullType Tail;
    };
{% endhighlight %}

上面三种定义就覆盖了Typelist中所以可能的情况，当我们写出类似`TypeList< FilterA, TypeList< FilterB, TypeList< FilterC, NullType > > >`的时候，编译器就能自动帮我们进行匹配，并且自动生成一种复杂的类型！

# 遍历TypeList

既然定义好了，下面就可以对这个类型列表进行遍历了。以过滤商品这个场景为例，我们需要一个Filters，里面包含几种过滤方式，这几种过滤方式组成一个TypeList。

{% highlight cpp linenos %}
    struct Goods {};

    struct BlacklistFilter
    {
        bool static do_filter(Goods& goods) { std::cout << "BlacklistFilter" << std::endl; return true;}
    };  

    struct CateFilter
    {   
        bool static do_filter(Goods& goods) { std::cout << "CateFilter" << std::endl; return true;}
    };  

    struct StockFilter
    {   
        bool static do_filter(Goods& goods) { std::cout << "StockFilter" << std::endl; return true;}
    };  

    typedef TypeList<BlacklistFilter,
            TypeList<CateFilter,
            TypeList<StockFilter,
            NullType> > > 
            FilterList;
{% endhighlight %}

另外还需要定义一个模版类，用于遍历这些Filter并根据商品实例的属性进行判断。

{% highlight cpp linenos %}
    template<typename Tlist>
    struct GoodsFilters
    {   
        bool static filter(Goods& goods)
        {   
            return Tlist::Head::do_filter(goods)        //第一个Filter的过滤结果
                && GoodsFilters<typename Tlist::Tail>::filter(goods);   //剩下的Filters的过滤结果
        }   
    };  

    //终结情况
    template<>
    struct GoodsFilters<NullType>
    {
        bool static filter(Goods& goods) {return true;}
    };

    Goods example_goods;
    GoodsFilters<FilterList>::filter(example_goods);
{% endhighlight %}

最终输出结果：

    BlacklistFilter
    CateFilter
    StockFilter

这样，对于Typelist的遍历就完成了，使用方无需编写代码显式调用每一个类型的过滤函数，一切都由编译器帮你完成了！ : )

# 核心思想 —— 递归

使用了Typelist（或者说是MetaProgramming）技术之后，感觉其最核心的就是递归的思想（通过模板特化来体现）。我们平时写的递归算法，各种分支、结束条件都需要自己判断，而Typelist只需要把各种典型的条件一一罗列出来即可，编译器会很聪明的对你的代码进行**编译时if判断**。TypeList中，还有许多用到了递归的地方，比方说计算Typelist的长度、通过下标获取类型、根据类型或者实例对应的值、Append操作、Reverse操作等等。其中Append操作以及Reverse操作比较经典，样例代码如下，大家可以体会下递归的精妙：

{% highlight cpp linenos %}
    //Append操作，将一个类型添加至一个TypeList末尾
    template<typename Tlist, typename T>
    struct Append;

    template<>
    struct Append<NullType, NullType>
    {
        typedef NullType Result;
    };

    template<typename S>
    struct Append<NullType, S>
    {
        typedef TypeList<S, NullType> Result;
    };

    template<typename H, typename T, typename S>
    struct Append<TypeList<H, T>, S>
    {
        typedef TypeList<H, typename Append<T, S>::Result > Result;
    };

    template<typename H, typename T>
    struct Append<NullType, TypeList<H, T> >
    {
        typedef TypeList<H, T> Result;
    };


    //Reverse操作，将TypeList反转过来，例如：TypeList<A, TypeList<B, TypeList<C, NullType> > >    =>  TypeList<C, TypeList<B, TypeList<A, NullType> > >
    template<typename Tlist>
    struct Reverse;

    template<typename Tlist>
    struct Reverse
    {
        typedef typename Append<typename Reverse<typename Tlist::Tail>::Result, typename Tlist::Head>::Result
                Result;
    };

    template<>
    struct Reverse<NullType>
    {
        typedef NullType Result;
    };

{% endhighlight %}


# 参考资料

* [《C++设计新思维》](http://book.douban.com/subject/1119904/) （很可惜，这么好的书，居然市面上买不到了，看了同事的英文原版，然后在淘宝上买了D版，罪过罪过）
* [Loki-lib TypeList](http://loki-lib.sourceforge.net/html/a00681.html)  《C++设计新思维》这本书中讲解的Loki库中对应TypeList部分
* [http://aszt.inf.elte.hu/~gsd/halado_cpp/ch06.html](http://aszt.inf.elte.hu/~gsd/halado_cpp/ch06.html)  网上搜到的匈牙利罗兰大学的网上教程，也是基本上讲解Loki库的


-- EOF --
