---
author: admin
comments: true
date: 2010-12-09 08:00:19+00:00
layout: post
slug: svn-documentation
title: SVN资料整理
wordpress_id: 117
categories:
- Essay
tags:
- svn
---

 

# SVN是什么？

 

 

SVN与CVS一样，是一个跨平台的软件，支持大多数常见的操作系统。作为一个开源的版本控制系统， Subversion 管理着随时间改变的数据。 这些数据放置在一个中央资料档案库 (repository) 中。 这个档案库很像一个普通的文件服务器, 不过它会记住每一次文件的变动。 这样你就可以把档案恢复到旧的版本, 或是浏览文件的变动历史。Subversion 是一个通用的系统, 可用来管理任何类型的文件, 其中包括了程序源码。

 

# 

 

# 用SVN需要安装哪些软件？

 

 

  
  * SubVersion：实现服务系统的软件。 
   
  * TortoiseSVN：是SVN客户端程序，为windows外壳程序集成到windows资源管理器和文件管理系统的Subversion客户端。 
   
  * Eclipse SVN Plugin：Eclipse IDE下的SVN插件。 
   
  * ………… 
 

# 

 

# SVN中哪些概念？

 

 

  
  * Repository 
 

版本库，用于储存所有的数据，版本库按照文件树形式储存数据－包括文件和目录，任意数量的客户端可以连接到版本库，读写这些文件。通过写数据，别人可以看到这些信息；通过读数据，可以看到别人的修改。

 

  
  * Checkout 
 

Checkout的意思签出，虽然和Export的效果一样是把代码从服务器下载到本地，但是Checkout有验证的功能，Checkout到某处的代码，将会被TortoiseSVN监视，里面的文件可以享受各种SVN的服务。

 

  
  * Commit 
 

假如你更新了目录中的文件，那么就可以用到commit功能。这个功能就是将你本地的文件修改记录上传到服务器上面，可以理解为上传。      
他会和服务器上面的文件进行对比，假如你更新了某个文件而服务器上面也有人更新了这个文件，并且是在你checkout之后做的更新，那么它会尝试将你的更新和他人的更新进行融合（merge），假如自动merge不成功，那么报告conflict，你必须自己来手动merge，也就是把你的更新和别人的更新无冲突的写在一起。

 

  
  * Update 
 

假如是多人合作的项目，自己不做修改的话别人也要修改，这时候就需要使用update来同步本地和服务器上的代码。      
注意，假如别人删除了某个文件，那么更新之后你在本地的也会被删除。如果本地的代码已经被修改，和commit一样会先进行merge，不成功的话就会报告conflict。

 

  
  * Branches / Tags 
 

版本控制系统的一个特性是能够把各种修改分离出来放在一个单独的开发线上。这条线被称为分支。分支经常被用来试验新的特性，而不会干扰正在修改编译器错误和 bug 的主开发线。当新的特性足够稳定之后，开发分支就可以合并回主分支里(主干)。      
版本控制系统的另一个特性是能够标记特殊的版本(例如一个发布版本)，所以你可以在任何时候重新建立一个特定的构建或环境。这个过程被称作标记。

 

  
  * Merge 
 

分支用来维护独立的开发支线，在一些阶段，你可能需要将分支上的修改合并到主干，或者相反。

 

# 

 

# 推荐的版本库结构

 

 

[![clip_image002](http://www.qxavier.me/wp-content/uploads/2010/12/clip_image002_thumb.jpg)](http://www.qxavier.me/wp-content/uploads/2010/12/clip_image002.jpg)

 

一个好的做法是将一个公司或者部门的所有项目的代码都放在一个版本库中，因为这样可以更好的在项目之间使用SVN的copy,diff已经merge等功能。      
建议项目由3个子文件夹组成：trunk, brached, tags。trunk存放主线代码；branches存放基于主线的某一分支；而tags则存放官方的发行版本。

 

# 

 

# 附：CVS，GIT，Mercurial和SVN比较

 

 

    

      
        

特征

      
       
        

CVS

      
       
        

Git

      
       
        

Mercurial

      
       
        

Subversion

      
          

      
        

是否原子提交

      
       
        

CVS: 没有. CVS提交不是原子的

      
       
        

Git: 是的. 提交都是原子的

      
       
        

Mercurial: 是的

      
       
        

Subversion: 提交都是原子的

      
          

      
        

文件和目录是否可以移动或重命名

      
       
        

CVS: 不是. 重命名不支持. 如果手动进行, 可能会损坏历史记录

      
       
        

Git: 支持重命名, 这是很实用的目的. git甚至能检测到重命名之后文件的改变. 尽管如此, 基于特殊的存储结构, 重命名不会被显示的记录, git能够推导出来(在实际使用中很容易做到)

      
       
        

Mercurial: 是的, 重命名是支持的

      
       
        

Subversion: 是的. 支持重命名

      
          

      
        

在移动或重命名之后智能合并

      
       
        

CVS: 不能. 重命名都不支持, 就不必说智能了

      
       
        

Git: 不支持. 细节在Git FAQ里: “Git有一个重命名的命令git mv, 但是这仅仅是为了便利. 效果和移掉某个文件, 增加另外一个文件没有任何区别”

      
       
        

Mercurial: 是的. 重命名之后智能合并是支持的. Mercurtial文档说:“如果我修改一个文件,而你重新命名了这个文件, 然后我们合并我们的变更, 那么我所做的修改就会被更新到根据旧文件名字而产生的新文件里(这可能就是你所期望的‘最简单的动作’, 但是不是所有版本控制系统都支持)

      
       
        

Subversion: 不支持. “svn help me“中提到“注意: 这个子命令相当于拷贝和删除.“并且可能有个bug

      
          

      
        

文件和目录拷贝

      
       
        

CVS: 不能. 拷贝不支持

      
       
        

Git: 不能. 拷贝不支持

      
       
        

Mercurtial: 是的. 支持拷贝

      
       
        

Subversion: 是的. 并且拷贝非常容易(O(1)). 包括产生分支

      
          

      
        

远程存储仓库的备份

      
       
        

CVS: 间接的. 可以使用John Polstra写的CVSup

      
       
        

Git: 是的. 是git的内部特征

      
       
        

Mercurial: 是的

      
       
        

Subversion: 间接的. 可以使用Chia-liang Kao的SVN::Mirror插件(好像是台湾人)或Shlomi Fish的SVN-Pusher工具

      
          

      
        

是否传递变更到父仓库

      
       
        

CVS: 不会

      
       
        

Git: 是的(Linux内核开发过程经常使用这个特征)

      
       
        

Mercurtial: 是的

      
       
        

Subversion: 是的, 使用要么是Chia-Ling Kao的SVN::Mirror脚本或者Shlomi Fish的svn-push工具

      
          

      
        

仓库权限

      
       
        

CVS: 很有限. “pre-commit hook scripts“能够被用来实现各种权限控制系统

      
       
        

Git: 请看和Git一起附带的contrib/hooks/update-paranoid. 看和svnperms类似的path_rules的代码

      
       
        

Mercutial: 是的. 它能够锁住仓库, 子目录或者使用hooks后的文件

      
       
        

Subversion: 是的. 基于HTTP权限的WebDAV-based模块能够支持基于目录级的仓库

      
          

      
        

变更集

      
       
        

CVS: 不是. 变更是基于文件的

      
       
        

Git: 是的. 是支持的, 创建他们很容易

      
       
        

Mercurial: 是的. 变更集是支持的

      
       
        

Subversion: 部分支持. 对于一次提交会隐式创建一个变更集

      
          

      
        

跟踪线性的文件历史

      
       
        

CVS: 是的. cvs annotate

      
       
        

Git: 是的.(git blame)

      
       
        

Mercurial: 是的(hg annotate)

      
       
        

Subversion: 是的(svn blame)

      
          

      
        

能够只在仓库的单目录下作用

      
       
        

CVS: 是的

      
       
        

Git: 不是. 尽管如此, 提交多少能被限制, 请看“Repository Permissions”

      
       
        

Mercurial: 能够基于某树的某个子集进行提交. 也有局部检出的能力

      
       
        

Subversion: 是的

      
          

      
        

跟踪未提交的变化

      
       
        

CVS: 是的. 通过cvs diff

      
       
        

Git: 是的. 另外, 分支在git里非常智能, 在某些工作流里能够被当成是另外一个未提交代码的存储库. 请看“git stash“命令

      
       
        

Mercurial: 是的. 使用hg diff

      
       
        

Subversion: 是的. 使用svn diff

      
          

      
        

基于单个文件的提交信息

      
       
        

CVS: 不是. 提交信息是基于单次变化的

      
       
        

Git: 是的. 提交信息基于变更集

      
       
        

Mercurial: 不是

      
       
        

Subversion: 不是. 没有这个特征

      
          

      
        

文档

      
       
        

CVS: 非常棒. 有很多在线的tutorials和资源, 在线的书籍. 命令行客户端也支持一个在线的帮助系统

      
       
        

Git: 良好. 短的帮助比较简洁难懂. man页很有分量, 但容易误解. 有很多tutorial

      
       
        

Mercurial: 很好. 有基于公司的书籍和wiki. 每个命令都集成了帮助

      
       
        

Subversion: 很好. 有一些在线的书籍和一些在线的tutorials和资源. 并且书籍是以docbook/xml写的所以很容易变换成其他格式. 命令行同样提供了在线的帮助系统

      
          

      
        

配置是否轻松

      
       
        

CVS: 好. 是个事实上的标准. 基于每个系统都有并且很容易配置

      
       
        

Git: 好. 在现有平台上二进制可用. 需要C编译器和Perl. 在windows上需要cygwin. 并有一些Unix特征

      
       
        

Mercurial: 非常好. 几乎所有平台都有二进制包. 从源码编译需要python2.3以上, 并且需要C编译器

      
       
        

Subversion: Subversion服务器需要安装在apache2模块里(如果有人希望HTTP作为底层协议的话)或使用它自身的服务器. 客户端需要Subversion特征的逻辑还有WebDAV库(针对HTTP). 安装组件很直接, 但是需要一些额外的工作(假定subversion在某些平台没有二进制包可用)

      
          

      
        

命令集

      
       
        

CVS: 包含了3个经常用到的命令的简单的命令集(cvs commit, cvs update和cvs checkout)和其它一些

      
       
        

Git: 命令集很丰富, 并且和CVS不兼容

      
       
        

Mercurial: 尝试模仿CVS交互方式, 但是偏离了基于不同的设计的意图

      
       
        

Subversion: 类CVS的命令集, 能够很容易被CVS用户使用

      
          

      
        

网络支持

      
       
        

CVS: 好. cvs在不同的场合使用不同的协议. 协议能够通过ssh链接的加密隧道进行

      
       
        

Git: 非常棒. 能够使用本地的git协议, 但也能在rsync, ssh, HTTP和HTTPS上使用

      
       
        

Mercurial: 非常棒. 使用HTTP或ssh. 远程访问会非常安全, 在只读网络里不需要上锁

      
       
        

Subversion: 非常好. Subversion服务器支持WebDAV+DeltaV(基于HTTP或HTTPS)作为底层协议, 或者它自身的协议同样能在ssh链接通道里使用.

      
          

      
        

可移植性

      
       
        

CVS: 好. 客户端能在UNIX, Windows和Mac OS上使用. 服务器端能在UNIX, 附有UNIX模拟层的Windows上使用

      
       
        

Git: 客户端运行在大多数的UNIX系统上, 但没有MS-Windows本地程序. 基于cygwin的系统看起来也能使用

      
       
        

Mercurial: 非常棒. 运行在基于所有能运行python的平台.仓库是兼容性的基于CPU结构和字节序的

      
       
        

Subversion: 非常好. 客户端和服务器端都能在UNIX, Windows和Mac OS X上运行

      
          

      
        

web接口

      
       
        

CVS: 是的. CVSweb, ViewVC, Chora和wwCVS

      
       
        

Git: 是的. Gitweb包含在发布包中

      
       
        

Mercurial: 是的. Web接口是内置组件

      
       
        

Subversion: 是的. ViewVC, SVN::Web, WebSVN, ViewSVN, mod_svn_view, Chora, Trac, SVN::RaWeb::Light, SVN Browser, Insurrection和perl_svn.另外, Subversion的apache服务也提供了一个基础的web接口

      
          

      
        

图形用户界面

      
       
        

CVS: 非常好. 有很多图形界面可以用: WinCVS, Cervisia(对于KDE), TortoiseCVS(Windows浏览器插件)

      
       
        

Git: Gitk包含在发行版中. Qqit和Git-gui工具也可使用

      
       
        

Mercurial: 通过hgit扩展查看历史; 检入扩展(hgct)使得提交很容易. 一些第三方的IDEs和GUI工具(如eric3, meld)有一些集成的Mercurial支持

      
       
        

Subversion: 非常好. 有很多GUIs可用: RapidSVN(跨平台), TortoiseSVN(Windows浏览器插件), Jsvn(java), 等. 大多数都还在开发中

      
       
