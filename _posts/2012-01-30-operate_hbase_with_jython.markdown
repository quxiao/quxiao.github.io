---
author: admin
comments: true
date: 2012-01-30 07:59:06+00:00
layout: post
slug: operate_hbase_with_jython
title: 使用Jython操作HBase
wordpress_id: 518
categories:
- Note
tags:
- hbase
- jython
---

之前使用Python的[PLY](http://www.dabeaz.com/ply/)写了一个操作HBase的类SQL编译器的雏形，目前还只初步完成了词法语法分析器，等开始写后面的预处理器、逻辑计划产生器和物理计划产生器的时候，问题出现了：HBase以及整个Hadoop项目都是用Java写成的，当然是使用Java API最直接。如果想使用其他语言的API接口，可以有这么几种方式：


	
  * [Thrift API](http://wiki.apache.org/hadoop/Hbase/ThriftApi)

	
  * [RESTful API](http://wiki.apache.org/hadoop/Hbase/Stargate)

	
  * 其他基于JVM的语言（Jython、Groovy、Scala等）


现在，如果要完成操作HBase的类SQL编译器的话，有三种方案：

	
  1. 前端继续用Python写编译器，后端用Java API操作HBase，中间的结果用某种形式（比如文件）进行保存

	
  2. 重新用Java写词法语法编译器，然后直接使用HBase的Java API

	
  3. 继续用Python写编译器，然后使用Jython


对于方案一，中间的结果可能会是比较复杂的数据结构，不便于保存，就算可以保存，读写也会比较麻烦。

对于方案二，风险集中在需要重新寻找和学习Java的词法语法编译器。之前找到了[ANTLR](http://www.antlr.org/)、[JavaCC](http://javacc.java.net/)等等，但是大多都比较笨重，学习成本较高，况且我预期的类SQL编译器相对简单，不需要那么高级的工具库。

对于方案三，用PLY写的编译器可以说已经完成一半了，使用起来也很轻便，如果Jython能够比较好的操作HBase的话，进度还是可以保证的。试用了Jython一下，感觉还不错！

综合各方面因素，暂时决定采用方案三。

好了，有点跑题了，言归正传，来看看Jython如果操作HBase。（配置Jython和HBase就不在这里讲了）

首先，将HBase启动：


> bin/start-hbase.sh


或者，HBase用到的classpath（因为Jython需要使用在classpath下的Java类），


> 

>     
>     ps auwx | grep java | grep org.apache.hadoop.hbase.master.HMaster | perl -pi -e "s/.*classpath //"
> 
> 



ps是获取正在运行的进程，其中Java启动的进行的格式类似于


> /usr/lib/jvm/java-6-sun//bin/java –Xmx1000m … –classpath XXX


最后使用的perl就是用来获取`-classpath`后面的XXX的。-p是指对于输入一行一行的循环操作，i是指不需要对输入文件进行备份，-e是指执行命令`s/.*classpath //`。（这个命令是把classpath以及之前的字符都去掉，一定能保证-classpath是最后一个参数吗？后面还有与classpath无关的参数怎么办？实际的情况的确有无关的参数，幸好只有一点，这里算是个小bug了。）

将获取的classpath导入到环境变量中


> 

>     
>     export CLASSPATH=XXX
> 
> 



这样，你就可以用Jython运行python脚本操作HBase了。下面是一个简单的例子：

    
    import java.lang
    from org.apache.hadoop.hbase import HBaseConfiguration, HTableDescriptor, HColumnDescriptor, HConstants
    from org.apache.hadoop.hbase.client import HBaseAdmin, HTable, Put, Get
    
    conf = HBaseConfiguration()
    
    admin = HBaseAdmin(conf)
    
    tablename = "test_jython_hbase"
    
    desc = HTableDescriptor(tablename)
    desc.addFamily(HColumnDescriptor("content"))
    
    # Drop and recreate if it exists
    if admin.tableExists(tablename):
        admin.disableTable(tablename)
        admin.deleteTable(tablename)
    admin.createTable(desc)
    
    table = HTable(conf, tablename)
    
    # Add content
    row = 'row_x'
    put_row = Put(row)
    put_row.add('content', 'some_content', 'some_value')
    table.put(put_row)
    
    # Read content
    get = Get(row)
    data_row = table.get(get)
    data = java.lang.String(data_row.value(), "UTF8")
    print "The fetched row contains the value '%s'" % data
    
    # Delete the table.
    admin.disableTable(desc.getName())
    admin.deleteTable(desc.getName())


输出结果如下所示：


    …………
    12/01/29 23:55:51 DEBUG client.HConnectionManager$HConnectionImplementation: Cached location for .META.,,1.1028785192 is ubuntu2-vmware:60020
    12/01/29 23:55:52 DEBUG client.MetaScanner: Scanning .META. starting at row=test_jython_hbase,,00000000000000 for max=2147483647 rows
    12/01/29 23:55:52 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=ubuntu3-vmware:2181,ubuntu2-vmware:2181 sessionTimeout=180000 watcher=hconnection
    12/01/29 23:55:52 INFO zookeeper.ClientCnxn: Opening socket connection to server ubuntu3-vmware/192.168.1.202:2181
    12/01/29 23:55:52 INFO zookeeper.ClientCnxn: Socket connection established to ubuntu3-vmware/192.168.1.202:2181, initiating session
    12/01/29 23:55:52 INFO zookeeper.ClientCnxn: Session establishment complete on server ubuntu3-vmware/192.168.1.202:2181, sessionid = 0x1352c9556270012, negotiated timeout = 180000
    12/01/29 23:55:52 DEBUG client.HConnectionManager$HConnectionImplementation: Lookedup root region location, connection=org.apache.hadoop.hbase.client.HConnectionManager$HConnectionImplementation@8a2006; hsa=ubuntu3-vmware:60020
    12/01/29 23:55:52 DEBUG client.HConnectionManager$HConnectionImplementation: Cached location for .META.,,1.1028785192 is ubuntu2-vmware:60020
    12/01/29 23:55:52 DEBUG client.MetaScanner: Scanning .META. starting at row=test_jython_hbase,,00000000000000 for max=10 rows
    12/01/29 23:55:52 DEBUG client.HConnectionManager$HConnectionImplementation: Cached location for test_jython_hbase,,1327910151208.09451a5e064db613648741bd8c896eb7. is ubuntu2-vmware:60020
    The fetched row contains the value 'some_value'
    12/01/29 23:55:52 INFO client.HBaseAdmin: Started disable of test_jython_hbase
    12/01/29 23:55:52 DEBUG client.HBaseAdmin: Sleeping= 1000ms, waiting for all regions to be disabled in test_jython_hbase
    12/01/29 23:55:53 DEBUG client.HBaseAdmin: Sleeping= 1000ms, waiting for all regions to be disabled in test_jython_hbase
    12/01/29 23:55:54 INFO client.HBaseAdmin: Disabled test_jython_hbase
    12/01/29 23:55:55 INFO client.HBaseAdmin: Deleted test_jython_hbase

