---
layout: post
title: "2021年1月23日java-打开文件数（Too many open files)"
date: 2015-08-25 
description: "linux 打开文件数（Too many open files)"
tag: java,linux
---   

转:https://blog.csdn.net/c814276009/article/details/84891109

	Linux系统下Java程序抛Too many open files异常，常见于高并发访问文件系统、多线程网络连接等场景。

程序打开的文件数过多，这里的file包括经常访问的文件、网络通信连接（socket）等等，有时也叫句柄。这个错误也可以叫做句柄数超过系统限制数。
### 详解：
- file entry	
	linux系统需要记录当前访问file的name、location、access、authority等信息。
- open files table：
以数组的形式存储file entry
- file descriptor table（文件描述符）：
作为进程到open files table的指针，也就是open files table的下标索引。将每个进程与它访问的文件关联起来。
每个进程中都有一个file descriptor table管理当前进程所访问（open或create）的所有文件。
> file descriptor table（文件描述符）关联着open files table中文件的file entry。

Linux系统配置open files table的文件限制，如果超过配置值，就会拒绝其他文件操作请求。并抛出Too many open files异常。限制分为系统和用户之分。
### 系统级：
系统级设置对所有用户有效。查看分为两种方式：
- 1:cat /proc/sys/fs/file-max  
- 2:sysctl -a 查看结果中fs.file-max这项的配置数量
- 3:增加配置数量:vim /etc/sysctl.conf文件，配置fs.file-max属性，如果属性不存在就添加。 
- 立即生效： sysctl -p

### 用户级：
Linux限制每个登录用户的可连接文件数。
- 查询限制数
> ulimit -n
- 设置限制数：
> ulimit -n <setting number>

对于文件描述符增加的比例，资料推荐是以2的幂次为参考。如当前文件描述符数量是1024，可增加到2048，如果不够，可设置到4096，依此类推。 

### 原因分析以及排查：
在出现Too many open files问题后，最大的可能是打开的文件或是socket没有正常关闭。为了定位问题是否由Java进程引起，通过Java进程号查看当前进程占用文件描述符情况： 
```
lsof -p $java_pid 每个文件描述符的具体属性  (获取PID:ps -ef|grep xxxx)
lsof -p $java_pid | wc -l 当前Java进程file descriptor table中FD的总量
```

### 检查程序问题：
如果你对你的程序有一定的解的话，应该对程序打开文件数(链接数)上限有一定的估算，如果感觉数字异常，请使用第一步的lsof -p 进程id > openfiles.log命令，获得当前占用句柄的全部详情进行分析，
1. 打开的这些文件是不是都是必要的？
2. 定位到打开这些文件的代码
3. 是否程序操作了文件写入，但是没有进行正常关闭
4. 是否程序进行了通讯，但是没有正常关闭(也就是没有超时结束的机制) 






