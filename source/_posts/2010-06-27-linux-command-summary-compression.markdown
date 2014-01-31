---
author: admin
comments: true
date: 2010-06-27 03:04:10+00:00
layout: post
slug: linux-command-summary-compression
title: Linux压缩命令小结
wordpress_id: 99
categories:
- Digest
tags:
- Linux embedded programming
- 压缩
---

**后缀 .tar**

解压命令: tar -xvf InFile.tar

压缩命令: tar -cvf OutFile.tar InFile

(x——extract	v——verbose	c——create	f——file)

**后缀 .tar.gz**

解压命令: tar -zxvf InFile.tar.gz

压缩命令: tar -zcvf OutFile.tar.gz InFile

(z——gzip)

**后缀 .tar.bz2**

解压命令: tar -jxvf InFile.tar.bz2

压缩命令: tar -jcvf OutFile.tar.bz2 InFile

(j——bzip2)

**后缀 .tar.Z**

解压命令: tar Zxvf InFile.tar.Z

压缩命令: tar Zcvf OutFile.tar.Z InFile

**后缀 .gz**

解压命令: gzip -d InFile.gz 或 gunzip InFile.gz

压缩命令: gzip InFile

(d——decompress)

**后缀 .zip**

解压命令: unzip InFile.zip

压缩命令: zip OutFile.zip InFile

**后缀 .bz2**

解压命令: bzip2 -d InFile.bz2 或 bunzip2 InFile.bz2

压缩命令: bzip2 -z InFile

**后缀 .Z**

解压命令: uncompress InFile.Z

压缩命令: compress InFile

**后缀 .rar**

解压命令: rar x InFile.rar

压缩命令: rar a InFile
