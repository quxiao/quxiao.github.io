---
author: admin
comments: true
date: 2011-08-16 06:18:33+00:00
layout: post
slug: hadoop_easy_test
title: Hadoop极精简运行实验
wordpress_id: 409
categories:
- Experiment
tags:
- hadoop
---

出于对海量数据处理的兴趣以及毕设实验的要求，最近空闲时间小搞了一下Hadoop。对Hadoop不了解的同学，可以看[这里](http://hadoop.apache.org/)。简单的说，它是一个利用MapReduce编程模型进行分布式大规模数据处理的框架。在Hadoop中，数据都表示为键/值对的形式，以便用MapReduce进行处理。MapReduce的思想十分简单，数据的处理过程分为两个阶段：map和reduce。map过程是将value（源数据）映射到不同的key中，这样所有数据就变成了key/value的集合，一个key会对应多个value；而在reduce阶段，则将每一个key的value集合进行处理，最后也同样得到key/value的集合，但这里一个key对应的一个value（计算结果）。

对于使用Hadoop的人来说，其实我们只需编写map和reduce函数，以及一个驱动程序即可，至于Hadoop是如何将多个value统一到一个key中，并且对这些value进行分组、排序，我们可以不考虑（只是暂时的）。由于手边没有“海量”数据，我只好自己造了。简单起见，我直接以键值对的形式生成了数据，而场景就是找到每个key的最大value。

我们首先来看map函数：

    
    package main;
    
    import java.io.IOException;
    
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Mapper.Context;
    
    /*
    * map function
    * the 4 class arguments of Mapper mean: input key, input value, output key, output value
    */
    public class FindMaxInKeyMapper extends Mapper<LongWritable, Text, IntWritable, IntWritable>
    {
    
    	public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    	{
    		String line = value.toString();
    		String[] results = line.split(" ");
    		int id, num;
    		try
    		{
    			id = Integer.parseInt(results[0]);
    			num = Integer.parseInt(results[1]);
    		}
    		catch (Exception ex)
    		{
    			return;
    		}
    		// add key/value
    		context.write(new IntWritable(id), new IntWritable(num));
    	}
    }


map阶段是数据的整理阶段，基类Mapper中的类型参数分别表示：输入的key、value类型，以及输出的key、value类型。注意，这里输出的key/value与MapReduce中的概念是一致的，而输入的key/value则不是，一般来说，输入的key表示输入数据的偏移量，而输入的value表示数据本身。所以，map阶段的主要任务就是把输入的value（源数据）表示成key/value的形式，供下一步的reduce使用。我们这个例子中，只需将表示一行的String分别分解成两个int就可以了。

好，再来看看reduce函数：

    
    package main;
    
    import java.io.IOException;
    
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.Reducer.Context;
    
    public class FindMaxInKeyReducer  extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable>
    {
    	/*
    	 * reduce function
    	 * find the max value in values (in the same key)
    	 */
    	public void reduce(IntWritable key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException
    	{
    		int maxValue = Integer.MIN_VALUE;
    		for (IntWritable value: values)
    		{
    			maxValue = Math.max(maxValue, value.get());
    		}
    		context.write(key, new IntWritable(maxValue));
    	}
    }


reduce阶段就是数据进行实质处理的阶段了，它的输入源于map阶段，是一个key和一个value的集合，它的输出就是一个key/value对。输入输出的key/value类型可以各不相同，只要map输出的key/value类型和reduce输入的一样就可以了。在我们的例子里面，只要在values里面找到一个最大的，然后以同样的key和那个最大的value最为输出、放入context即可。

最后，写一个驱动程序（不是设备的驱动程序，你懂的）：

    
    package main;
    
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    
    public class FindMaxInKey {	
    
    	public static void main(String[] args) throws Exception{
    		if ( args.length != 2 )
    		{
    			System.err.println("Usage: FindMaxInKey <input path> <output path>");
    			System.exit(-1);
    		}
    
    		Job job = new Job();
    		job.setJarByClass(FindMaxInKey.class);
    
    		FileInputFormat.addInputPath(job, new Path(args[0]));
    		FileOutputFormat.setOutputPath(job, new Path(args[1]));
    
    		job.setMapperClass(FindMaxInKeyMapper.class);
    		job.setReducerClass(FindMaxInKeyReducer.class);
    
    		job.setOutputKeyClass(IntWritable.class);
    		job.setOutputValueClass(IntWritable.class);
    
    		System.out.println("start working");
    		System.exit(job.waitForCompletion(true) ? 0: 1);
    	}
    
    }


驱动程序的工作也很简单，设置一下Map类、Reduce类、数据输入输出路径，基本上就可以了。好，下面，跑起来！

如果在配置中采用默认设置，Hadoop底层的文件系统就是本地操作系统的文件系统。不过，一般都要设置成HDFS分布式文件系统的。（肯定啊，否则Hadoop有个什么意义）将工程打成jar包，然后敲入命令：


> quxiao@ubuntu1:~/hadoop/bin$ ./hadoop jar /home/quxiao/workspace/HadoopTest/HadoopTest.jar main.FindMaxInKey /user/quxiao/data.in /user/quxiao/outputdata


就可以看到命令行中提示map和reduce的完成百分比，在Hadoop自带的监控系统可以更直观的看到运行的信息。

[![](http://www.qxavier.me/wp-content/uploads/2011/08/Screenshot-2-1024x573.png)](http://www.qxavier.me/wp-content/uploads/2011/08/Screenshot-2.png)

[![](http://www.qxavier.me/wp-content/uploads/2011/08/Screenshot-3-1024x573.png)](http://www.qxavier.me/wp-content/uploads/2011/08/Screenshot-3.png)

运行之后，程序的输出是这种形式：


> 

>     
>     0	2147435993
>     1	2147464484
>     2	2147479158
>     3	2147458512
>     4	2147389924
>     5	2147416503
>     6	2147378427
>     7	2147478518
>     8	2147433586
>     9	2147435229
>     10	2147381593
>     11	2147463851
>     12	2147454928
>     13	2147462197
>     14	2147480411
>     15	2147481870
>     16	2147469297
>     17	2147435232
>     18	2147439681
>     19	2147437661
> 
> 



好！这个极精简实验就算是运行成功了。初步来看，用Hadoop做数据分析还是挺简单的，你不用担心数据的分组、排序、存储，以及许多分布式相关的细节，只要写好map和reduce函数即可。
