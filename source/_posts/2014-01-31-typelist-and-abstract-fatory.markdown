---
layout: post
title: "typelist && abstract fatory"
date: 2014-01-31 14:35:26 +0800
comments: true
categories: 
---

上一篇博客讲到了模板元编程中的Typelist，这种技术能够让编译器帮你生成许多结构类似的代码，省去了程序员自己编写代码的时间以及一些运行时的效率损失。设计模式中的Abstract Factory，其Template MetaProgramming版本也依赖于TypeList技术。

# 普通实现

Abstract Factory模式，即规定了一组抽象产品（Abstract Product）的接口，再由各个具体的Factory生成不同组的具体产品（Concrete Product）。我一下子想到了公司的RD、QA以及PM，就以这三种角色为例吧。（虽然不是很恰当，因为不同级别的不同职位是可以共存的）

普通的代码一般会写成这样：

``` cpp
class AbstractFactory
{
public:
    virtual RD* create_RD() = 0;
    virtual QA* create_QA() = 0;
    virtual PM* create_PM() = 0;
};

class JuniorFactory: public AbstractFactory
{
public:
    RD* create_RD() {return new JuniorRD();}
    QA* create_QA() {return new JuniorQA();}
    PM* create_PM() {return new JuniorPM();}
};

class SeniorFactory: public AbstractFactory
{
public:
    RD* create_RD() {return new SeniorRD();}
    QA* create_QA() {return new SeniorQA();}
    PM* create_PM() {return new SeniorPM();}
};

//使用方代码
AbstractFactory* fact = NULL;
if (/**/) {
    fact = new JuniorFactory();
} else (/**/) {
    fact = new SeniorFactory();
}

RD* rd = fact->create_RD();
//...
```

可以看出来，每一个具体的Factory类只是重复的实现AbstractFactory提供的接口，唯一的不同就是填写具体的类名，其它的语句都是重复的。（我不得不在编写以上代码的时候使用了Vim的替换功能）

# 定义接口

有了Typelist，我们可以联想到，如果有了一个Typelist例如：

``` cpp
TypeList<RD, TypeList<QA, TypeList<PM, NullType> > >
```

再加上一个模板函数：

``` cpp
template <typename T>
T* create()
{
    return new T();
}
```

应该就可以生成一组接口了吧！但是有个问题，如何根据Typelist中的每一个类，生成一个对应的接口呢？这里用到了一种技巧，能够根据Typelist，生成一个**松散结构**的复杂类，这个复杂类中就声明了一组接口。其代码如下：

``` cpp
/*
 * Scatter Hierarchy
*/

template<class TypeList, template<class> class Unit>
class ScatterHierarchy;

template<class Head, class Tail, template<class> class Unit>
class ScatterHierarchy<TypeList<Head, Tail>, Unit>
    : public ScatterHierarchy<Head, Unit>
    , public ScatterHierarchy<Tail, Unit>
{};

template<class SingleType, template<class> class Unit>
class ScatterHierarchy: public Unit<SingleType>
{};

template<template<class> class Unit>
class ScatterHierarchy<NullType, Unit>
{};
```

ScatterHierarchy类型，其模板参数有两个，一个是Typelist，这个很明显，第二个的语法有点怪，是一个`template template parameter`，他是一种类型，并且这个类型也是一个模板类，就是他规定了接口。假设实际应用中，Unit的定义是这样的：

``` cpp
template<class T>
struct Type2Type
{
    typedef T OriginType;
};

template<class T>
class AbstractFactoryUnit
{
public:
    virtual T* doCreate(Type2Type<T>) = 0;
    virtual ~AbstractFactoryUnit() {};
};
```

这样如果代入Typelist的话，我们就会得到三个AbstractFactoryUnit基类，分别是：`AbstractFactoryUnit<RD>`、`AbstractFactoryUnit<QA>`以及`AbstractFactoryUnit<PM>`，他们都有一个`doCreate`接口。Wait a minute，那个`Type2Type`是做什么的？里面不就是`typedef`了一下嘛！这个其实是一个tricky的技巧：**让不同接口的doCreate函数有不同的函数签名**。这样当我调用`doCreate`接口的时候，就能让编译器确定接口是哪个Type（RD、QA or PM）的了。

好了，有了这些基础，就可以开始定义AbstractFactory了，其实我们仅需要继承上面的ScatterFactory即可，然后将具体的TypeList和AbstractFaUnit作为模板参数即可：

``` cpp
template
<
typename Tlist,
template<typename> class Unit = AbstractFactoryUnit     /*默认模板参数*/
>
class AbstractFactory: ScatterHierarchy<Tlist, Unit>
{
public:
    typedef Tlist ProductList;
    template<typename T>
        T* create()
        {
            Unit<T>* unit = this;   //AbstractFactory是任意一个Unit<T>的子类, for T in Tlist
            return unit->doCreate(Type2Type<T>());
        }
};

//使用方代码，定义了三个接口的AbstractFactory
typedef AbstractFactory
<
TypeList<PM, TypeList<RD, TypeList<QA, NullType> > >
>
AbstractSoftwareEngineerFactory;
```

# 实例化

现在来想想看怎么实例化一个抽象工厂。需要这样三部分元素：

* 抽象接口（AbstractSoftwareEngineerFactory提供）
* 实例Typelist（`TypeList<JuniorRD, TypeList<JuniorQA, Typelist<JuniorPM, NullType> > >`）
* 实例函数（当用户代码为`create<RD>()`时，能够`new JuniorRD();`）

前两部分都比较容易提供，第三部分的实例函数怎么提供呢？怎么能够一一对应上呢？初步的想法是：AbstractSoftwareEngineerFactory中有Abstract Product（RD / QA / PM），实例Typelist中有Concrete Product（Junior RD / Junior QA / Junior PM），能够把这两种Typelist中的第一个类型（`TypeList::Head`）对应起来，并且实现了`doCreate`接口，接着再用递归的思想将剩下的Type依次实现即可！

模板元编程中的递归思想，其表现形式我想到了两种：1）类继承；2）模板特化。另外还需考虑到的是，这种递归应该是一种**线性式**的，因为需要按顺序遍历Typelist。与之前提到过的ScatterHierarchy实现类似，还有一种技术能够生成一种线性式的类继承关系——**LinearHierarchy**

``` cpp
/*
 * Linear Hierarchy
 */

template
<
    class Tlist,
    template<class SingleType, class Base> class Unit,
    class Root = NullType
>
class LinearHierarchy {};

template
<
    class Head,
    class Tail,
    template<class, class> class Unit,
    class Root
>
class LinearHierarchy<TypeList<Head, Tail>, Unit, Root>
    : public Unit< Head, LinearHierarchy<Tail, Unit, Root> > {};    //Head被“提取”出来了

template
<
    class T,
    template<class, class> class Unit,
    class Root
>
class LinearHierarchy<TypeList<T, NullType>, Unit, Root>
    : public Unit<T, Root> {};

```

如果，我们将Tlist设置为Concrete Product的Typelist，Root中含有Abstract Product的Typelist，这样就不难找到对应关系了。别忘了，AbstractSoftwareEngineerFactory中就含有Abstract Product，Root用它就好了。

那么对于`RD / QA / PM`的例子，生成的类之间的继承关系就会如下所示（最上面是基类）：

       ASF
        |
    Unit<PM, ASF>
        |
    LinearHierarchy<TypeList(PM), ASF>
        |
    Unit<QA, LinearHierarchy<TypeList(PM), ASF>
        |
    LinearHierarchy<TypeList(QA, PM), ASF>
        |
    Unit<RD, LinearHierarchy<TypeList(QA, PM), ASF>
        |
    LinearHierarchy<TypeList(RD, QA, PM), ASF>
        |
    ConcreteFactory

对于Unit，我们可以这样实现：

``` cpp
template<class ConcreteProduct, class Base>
class OpNewUnit: public Base
{
private:
    //AbstractSoftwareEngineerFactory中的TypeList
    typedef typename Base::ProductList BaseProductList;

protected:
    //将Abstract Product中的Head“吃掉”，将Tail作为整个Typelist传至子类，精妙！
    typedef typename BaseProductList::Tail ProductList;

public:
    typedef typename BaseProductList::Head AbstractProduct;

    ConcreteProduct* doCreate(Type2Type<AbstractProduct>)
    {
        return new ConcreteProduct();
    }
};
```

最后，终于可以实例化一个工厂模板类了，只需继承一个含有以上三部分元素（抽象接口、具体TypeList、实例函数）的LinearHierarchy模板类就可以了：

``` cpp
template
<
    class AF,
    class ConcreteProductTypeList,
    template<class, class> class Creator = OpNewUnit
>
class ConcreteFactory: public LinearHierarchy<typename Reverse<ConcreteProductTypeList>::Result, Creator, AF>
{
public:
    typedef typename AF::ProductList ProductList;
    typedef ConcreteProductTypeList ConcreteProductList;
};

//使用方代码
typedef ConcreteFactory
<
    AbstractSoftwareEngineerFactory,
    TypeList<JuniorRD, TypeList<JuniorQA, TypeList<JuniorPM, NullType> > >
>
    JuniorFactory;

typedef ConcreteFactory
<
    AbstractSoftwareEngineerFactory,
    TypeList<SeniorRD, TypeList<SeniorQA, TypeList<SeniorPM, NullType> > >
>
    SeniorFactory;

AbstractSoftwareEngineerFactory* junior_fact = new JuniorFactory;
PM* pm = junior_fact->create<PM>(); //"junior PM"
cout << pm->get_title() << endl;

AbstractSoftwareEngineerFactory* senior_fact = new SeniorFactory;
RD* rd = senior_fact->create<RD>(); //"seinor RD"
cout << rd->get_title() << endl;
```

可能你会看到，上面的代码用到了`Reverse`，把使用方的ConcreteProductList给反转过来，这是为什么呢？可以再看下`OpNewUnit`的实现，它是将AbstractProductList中的Head去掉之后，将剩下的部分传递至**子类**，而根据`LinearHierarchy`的实现，在ConcreteProductList中越是头部的类，越是在子类中，所以需要将其中一个ProductList给反转过来，以便让`OpNewUnit`中的`BaseProductList::Head`和模板参数`ConcreteProduct`能够对应起来。关于`Reverse`的实现，可以查看上一篇文章的最后部分。

# 参考资料

* [《C++设计新思维》](http://book.douban.com/subject/1119904/) TypeList && AbstractFactory章节

-- EOF --
