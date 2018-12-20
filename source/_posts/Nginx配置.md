---
title: Nginx配置
date: 2018-12-20 10:55:25
tags: Nginx
---

### Nginx模块结构

![img](nginx文件结构.png)

#### 全局模块

~~~nginx
worker_processes  2;	#指定工作进程为2也可以设置为auto

worker_cpu_affinity 01 10;	#指定CPU核占用情况。例如：CPU有2核，则第一个进程占用第二个核，第二个进程占用第一个核，此处0和1仅代表占位符，与二进制无关

worker_rlimit_nofile 65535;	#同时打开文件数，需要操作系统支持
~~~

#### events模块

~~~nginx
worker_connections  1024;	#一个进程同时可开启的最大连接数

accept-mutex on;	#on表示工作进程排队处理请求，防止多个进程同时抢占资源引起“惊群”，

accept-mutex_delay 500ms;	#每隔500ms工作进程查看上一个工作进程是否释放了锁，

multi_accept on;	#允许同时接收多个连接
~~~

#### http模块

~~~nginx
sendfile on;	#开启通过sendfile()零拷贝传输文件，sendfile()可以在磁盘和TCP socket之间互相拷贝数据

sendfile_max_chunk 128k;	#每个进程调用sendfile()传输的数据量最大不能超过该值，默认为0表示无限制

tcp_nopush on;	#在一个数据包里发送头文件，而不是一个接一个发送

tcp_nodelay on;	#不缓存数据，直接发送

keepalive_timeout 60 10;	#60代表对服务器端保持连接时间，默认为75s,10代表发给用户端的相应报文中Keep-Alive时间

keepalive_requests 10000;	#单连接请求上线，限制用户通过某一连接发送请求次数

client_body_timeout 10;	#读取客户端请求体超时时间，超时针对2个连续读操作之间的时间段，并不是整个请求体传输的时间
~~~

