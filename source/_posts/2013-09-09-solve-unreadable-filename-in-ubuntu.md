---
title: ubuntu中文文件名乱码处理
date: '2013/09/09 20:00:00'
tags:
  - ubuntu
  - 乱码
---

中文文件名乱码产生的原因有二：<br>
一是挂载NTFS或FAT文件系统时，编码指定不正确导致乱码（或问号）； 二是在文件系统中文件名存储的编码不正确，导致乱码。

详细介绍可以看 [中文文件名乱码问题](http://linux-wiki.cn/wiki/zh-hans/%E4%B8%AD%E6%96%87%E6%96%87%E4%BB%B6%E5%90%8D%E4%B9%B1%E7%A0%81%E9%97%AE%E9%A2%98)。下面说说第二种情况。

文件是在WIndows 下创建的,Windows 的文件名中文编码默认为GBK,而Linux中默认文件名编码为UTF8,由于编码不一致所以导致了文件名乱码的问题，解决这个问题需要对文件名进行转码。

# 1.安装 convmv

```
sudo apt-get install convmv  
```

# 2.convmv简介

convmv存在于常见操作系统的软件仓库中。如果当前没有工具，可以直接安装。

```
convmv -f 源编码 -t 新编码 [选项] 文件名
```

常见有用的选项有：

```
-r 递归处理子文件夹
--notest 真正进行操作，请注意在默认情况下是不对文件进行真实操作的，而只是试验。
--unescape 可以做一下转义，比如把%20变成空格，如果你不知道这是在做什么，就不要尝试了。
```

# 3.实例

先试着将gbk文件名改为UTF-8：

```
convmv -f gbk -t utf8 -r MY_DIR  
```

如果确认输出无误：

```
convmv --notest -f gbk -t utf8 -r MY_DIR[MY_DIR是目录]
```

其他

```
convmv --notest -f gbk -t utf8 -r *[把当前文件夹下所有乱码文件名改过来]
```
