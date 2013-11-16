---
author: admin
comments: true
date: 2011-03-31 12:15:33+00:00
layout: post
slug: java-memory-mapped-file-stream-and-the-i-o-performance-comparison
title: Java中Stream和Memory-mapped File的I/O性能对比
wordpress_id: 244
categories:
- Development
---

建立和使用I/O系统，不管对于哪一种语言都是困难的（在Java中使用I/O库尤其困难）。在Java中，会有多种方式对于文件或者设备进行I/O操作。方法不同，性能很有差异。今天我们来对比一下通过Stream以及Memory-mapped File这两种方式的I/O性能对比。

 

**Stream**

 

“流”是I/O系统中通常使用的抽象概念，表示可以发出或者接受数据的点（source / sink）。“流”的概念也屏蔽掉了不同设备间的细节差异。     
Input stream通常可以为：

 

  
  * 字节数组 
   
  * String对象 
   
  * 文件 
   
  * 管道（pipe） 
   
  * 其他流（将多个流组成一个流） 
   
  * 网络设备等 
 

Output stream通常可以为：

 

  
  * 字节数组 
   
  * 文件 
   
  * 管道 
 

**Memory-mapped File**

 

对于那些很大的文件，将整个文件全部读进内存，将会极大地影响性能。通过Memory-mapped File可以只将文件的一部分读入内存，其他部分被换出（swap out），这样就能效率很高的读写大文件。

 

在Java的nio包中，用到了两种更接近于操作系统具体实现的数据结构：channel和buffer，memory-mapped file中就用到了channel。这种I/O方式，用户不需要跟数据源打交道，而只需把数据放到channel和buffer中，然后再调相应函数就可以了，通过这种“底层”的操作，已得到最大的I/O性能。（这种方式其实就跟C中的read和write一样了）

 

通过测试程序，我们可以看到两种方式性能的对比：

 

(注：表格中的文件大小N表示文件中含有N个char，而读写时间为ms)

 

    

      
读写方式  文件大小
       
10
       
100
       
2^10
       
2^20
       
2^21
       
2^22
          

      
Stream Write
       
1.874
       
1.974
       
1.828
       
161.258
       
368.252
       
656.428
          

      
Memory-mapped File Write
       
2.5
       
6.648
       
1.265
       
11.774
       
20.23
       
46.262
          

      
Stream Read
       
0.122
       
0.174
       
0.58
       
158.352
       
313.37
       
627.531
          

      
Memory-mapped File Read
       
0.428
       
0.463
       
0.382
       
10.797
       
19.381
       
39.749
         

[![image](http://www.qxavier.me/wp-content/uploads/2011/03/image_thumb.png)](http://www.qxavier.me/wp-content/uploads/2011/03/image.png)

 

可以看到，文件很小时，Memory-mapped方式是没有优势的，而当读写大文件时，优势就十分明显了。

 

关键代码如下：

 
    
    public class IOPerComp {
    
    	/**
    	 * @param args
    	 */
    	public static void main(String[] args) {
    		// TODO Auto-generated method stub
    		int[] intSizes = {10, 100, 1<<10, 1<<20, 1<<21, 1<<22};
    		for (int i = 0; i < intSizes.length; i ++)
    		{
    			INT_SIZE = intSizes[i];
    			System.out.println("file size : " + INT_SIZE);
    			for (Tester tester : tests)
    			{
    				tester.runTest();
    			}
    			System.gc();
    			System.out.println("**********************");
    		}
    	}
    
    	private static final String FILE_LOC = "D:/tmpFile.txt";
    	private static int INT_SIZE = 1024 * 1024;
    	private abstract static class Tester {
    		private String name;
    		public Tester () {}
    		public Tester (String name)
    		{
    			this.name = name;
    		}
    		public void runTest ()
    		{
    			System.out.println("name : " + name);
    			long start = System.nanoTime();
    			try
    			{
    				test();
    			}
    			catch (IOException ioe)
    			{
    				ioe.printStackTrace();
    			}
    			start = System.nanoTime() - start;
    			System.out.format("time : %.3fn", start/1.0e6);
    		}
    		public abstract void test() throws IOException;
    	}
    
    	private static Tester[] tests= {
    		new Tester("Stream Write")
    		{
    			public void test() throws IOException
    			{
    				File f = new File (FILE_LOC);
    				if ( f.exists() )
    					f.delete();
    				DataOutputStream dos = new DataOutputStream(
    						new BufferedOutputStream(
    								new FileOutputStream(FILE_LOC)));
    				for (int i = 0; i < INT_SIZE; i ++)
    					dos.writeInt(i);
    				dos.close();
    			}
    		},
    		new Tester("Mapped Write")
    		{
    			public void test() throws IOException
    			{
    				FileChannel fc = new RandomAccessFile(FILE_LOC, "rw").getChannel();
    				IntBuffer ib = fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size()).asIntBuffer();
    				for (int i = 0; i < INT_SIZE; i ++)
    					ib.put(i);
    				fc.close();
    			}
    		},
    		new Tester("Stream Read")
    		{
    			public void test() throws IOException
    			{
    				DataInputStream dis = new DataInputStream(
    						new BufferedInputStream(
    								new FileInputStream(FILE_LOC)));
    				for (int i = 0; i < INT_SIZE; i ++)
    				{
    					dis.readInt();
    				}
    				dis.close();
    			}
    		},
    		new Tester("Mapped Read")
    		{
    			public void test() throws IOException
    			{
    				FileChannel fc = new FileInputStream(FILE_LOC).getChannel();
    				IntBuffer ib = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size()).asIntBuffer();
    				while ( ib.hasRemaining() )
    				{
    					ib.get();
    				}
    				fc.close();
    			}
    		}
    	};
    }
