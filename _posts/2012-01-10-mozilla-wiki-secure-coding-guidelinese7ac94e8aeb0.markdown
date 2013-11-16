---
author: admin
comments: true
date: 2012-01-10 04:04:52+00:00
layout: post
slug: mozilla-wiki-secure-coding-guidelines%e7%ac%94%e8%ae%b0
title: Mozilla Wiki Secure Coding Guidelines笔记
wordpress_id: 515
categories:
- Note
tags:
- security
- web
---

最近网站安全问题频发啊，我在某技术论坛上的帐号也沦陷了（虽然N年不用了）。自己赶紧补补这方面的知识，重点看了下Mozilla Wiki中的Secure Coding Guidelines，记下一些重点和之前忽略的地方。

 

# 密码怎么设计？

 

密码设计原则

 

  
  * 最好是8位或8位以上 
   
  * 要有数字、字母的组合，最好是包含字母大小写和特殊字符 
   
  * 设置密码黑名单，禁止用户注册过于简单的密码，以防止黑客通过社会工程学猜到密码 
 

之前做项目只想到第一条，而且还是只要求6位。要是用户只注册含有数字的6为密码，那就可以被轻易破解了。人人网都没有做到以上全部，我的人人密码就是6位数字。。。虽说密码设计应该是用户自己的事情，但是简单的密码实在是太好破解了，还是最好引导用户设计出相对复杂的密码比较好。

 

# 密码怎么保存？

 

密码应该用[hmac](http://en.wikipedia.org/wiki/HMAC)+[bcrypt](http://en.wikipedia.org/wiki/Bcrypt)进行加密保存。

 

简单的说，hmac是通过含有密钥的hash算法，生成消息认证码（Message Authentication Code），可以进行数据一致性检测和消息认证；bcrypt是一种较“慢”的hash算法，这样可以使得暴力枚举破解的时间成本极大提升。

 

bcrypt的Python实现[py-bcrypt](http://www.mindrot.org/projects/py-bcrypt/)和Java实现[jBCrypt](http://www.mindrot.org/projects/jBCrypt/)。

 

我感觉，即使不用Mozilla推荐的hmac+bcrypt的策略，至少也应该是用md5和sha来保存的。明文是绝对不允许的，否则一抓包，用户名和密码就搞到手了。让黑客暴力破解出密码可能不是你的错，但是直接把密码告诉黑客，就是你的不对了。

 

# 输入是在客户端验证，还是在服务端验证？

 

都要！因为即使有客户端的js验证，用户也可以禁用浏览器的js功能，将“脏”数据传到服务端。

 

在客户端验证，是为了有很好的用户体验，避免什么数据都要先提交到服务端，然后再告诉你提交的数据有误；在服务端验证，则是加上了第二把锁，并且，有的数据是一定要到服务端才能检测的，比如帐号密码。

 

另外，在验证输入时，应该检测符合要求的格式，即“**accept known good**”，而不是检测不符合要求的格式，即“reject known bad”。符合要求的格式是可以设计全面的，它的补集则不可以。

 

# 输出编码

 

输出编码是防止XSS和各种注入的首要手段。（输入验证是次要手段）

 

    输出编码就是将一些HTML、JS、XML等格式中的特殊字符进行转义，比如将<script>转化为&lt;script&gt;。这样，即使用户在之前的输入中填写了恶意代码，等到系统显示的时候，它也不会被执行了。

 

在进行SQL查询时，应该使用参数化的SQL语句（parameterized queries），千万不要使用字符串拼接来进行SQL查询（貌似之前的项目绝大部分都是这么做的 :( ）。一些ORM（比如Hibernate）可以阻止一部分的SQL注入，但是在一些情况下，ORM仍然会使用native SQL，所以ORM不是万能的。

 

# 访问控制

 

应该防止一些URL暴露保密的内容，即使你没有提供给用户，一些用户也会有意无意的访问到那些URL（比如通过枚举）。设计某个Web工程的时候，应该先要按照某种原则设计好URL目录，然后根据不同的路径设计不同的访问权限，比如可以通过url filter实现。

 

安全可是永恒的话题，先写这么多，以后还要在实践中慢慢领悟。

 

--EOF--
