---
author: admin
comments: true
date: 2012-09-19 13:44:25+00:00
layout: post
slug: blog-has-moved
title: 博客搬家了
wordpress_id: 587
categories:
- Essay
---

博客搬家了，从原来[胡戈戈](http://hugege.com/)的虚拟主机搬到了[Hello,Host!](http://hellohostnet.com)的VPS上面，域名也从qxavier.info换成了[qxavier.me](http://qxavier.me)了。给博客搬家不是因为主机的质量或者服务问题，对于胡戈戈的服务态度以及虚拟主机的质量，我感到很满意。每次我有问题，他都能及时的响应，给我满意的答复。博客搬家的主要原因是自己想“折腾”、或者说是虚拟主机的限制有些多，除了写博客，其他事情基本做不了，就算在上面搭一个很简单的PHP程序，也很费劲。于是自己就将眼光投向了自由度更大、费用也还算合理的VPS。

自由度大了，但你也要因此而付出代价，那就是在博客搬家途中会踩到各种各样的“坑”。在这也整理一下，给大家提供些参考。


# 域名解析


.me等“非主流”域名，需要第三方解析服务，例如dnspod.cn，可以参考[这里](http://hellohostnet.com/wiki/doku.php?id=dnspod_%E8%AE%BE%E7%BD%AE%E6%95%99%E7%A8%8B)。这也是防止域名或者DNS服务器被墙的一个措施。


# FTP服务安装


wordpress在安装插件或者上传图片时，需要FTP服务。因为操作系统用的是ubuntu，所以安装软件apt-get install一下就可以了，安装的是vsftpd。安装过程很简单，但关键是修改配置。除了以下配置：

    
    # Uncomment this to allow local users to log in.
    local_enable=YES
    #
    # Uncomment this to enable any form of FTP write command.
    write_enable=YES


还需要特别注意掩码的设置，之前就是因为忘了设置，结果在上面耗了几乎两天的时间！

    
    # Default umask for local users is 077. You may wish to change this to 022,
    # if your users expect that (022 is used by most other ftpd's)
    local_umask=022




# 用户权限设置




一般VPS直接就给我们root帐号用了，但是如果没有其他帐号，vsftpd是无法使用的，因为它不支持root登录，所以之前还得创建一个用户。另外，如果是将wordpress文件夹放在该用户下，还需要将里面所有文件的所有者和所有组从root改为这个用户。





# Apache开启rewrite_mod模块


wordpress的文章url是虚拟的，依赖于apache的rewrite_mod模块，所以必须开启该模块。


# 数据库导入


之前以为很困难，其实比较简单。因为换了域名，所以数据库中的出现域名的地方需要替换成新的域名，只要dump处理，替换一下，然后导入到新的数据库就行了。

--EOF--
