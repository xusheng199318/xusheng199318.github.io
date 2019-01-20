---
title: 系统级IO
date: 2019-01-15 12:46:13
tags: 操作系统
---

###   系统级I/O

>  输入操作从IO设备把数据复制到主存，输出操作是从主存复制数据到IO设备

### 文件

* 普通文件：文本文件和二进制文件
* 目录：包含一组`链接`的文件，其中每个链接都将一个文件名映射到一个文件，这个文件可能是另一个目录
* 套接字：用来与另一个进程进行跨网络通信的文件

### 打开和关闭文件

~~~C
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open (char *filename, int flags, mode_t mode);
~~~

> 进程通过open函数来打开一个已存在的文件或者创建一个新文件

### 读和写文件

~~~c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t n);

ssize_t write(int fd, const void *buf, size_t n);
~~~

### 读取目录内容

~~~c
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name);
~~~

~~~c
#include <dirent.h>
struct dirent *readdir(DIR * dirp);
~~~

~~~c
#include <dirent.h>
int closedir(DIR *dirp);
~~~

