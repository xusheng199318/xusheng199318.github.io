---
title: Docker基础命令
date: 2019-06-12 14:25:30
tags: docker
---

### 查看docker信息

> docker info

### 查看docker帮助文档

> docker --help

### 获取镜像

> docker pull NAME[:TAG]：TAG如果没有默认为lastest

### 查看镜像信息

> docker images：查看本机上已有的镜像
>
> docker inspect REPOSITORY | IMAGE ID：查看镜像详细信息

### 搜索镜像

> docker search TERM：搜索镜像
>
> --filter=is-automated=true：仅显示自动创建的镜像
>
> --no-trunc=true：输出信息不截断显示
>
> --filter=stars=800：显示评价为星级以上的镜像

### 删除镜像

> docker rmi IMAGE [IMAGE...]：删除指定镜像

### 新建容器

> docker create -it IMAGE：创建一个容器

### 查看容器

> docker ps：查看运行中的容器
>
> -a：查看所有容器

### 新建并启动容器

> docker run IMAGE /bin/bash：等价于先docker create 后在docker start
>
> 例如：docker run -it ubuntu:14.04 /bin/bash 
>
> -t：让Docker分配一个伪终端并绑定到容器的标准输入上，容器启动后进入其命令行
>
> -i：让容器的标准输入保持打开，运行容器
>
> --name：执行容器名称
>
> -v：目录映射关系，前一个是宿主机目录
>
> -d：以守护进程方式运行
>
> -p：端口映射
>
> docker run -d IMAGE：让容器在后台以守护态运行
>
> 例如：docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
>
> -P：随机映射端口，可通过docker port CONTAINER PORT 查看映射信息
>
> -p：指定端口映射
>
> 例如：docker run -p port:port  IMAGE NAME
>
> docker ps ： 查看容器
>
> -a：查看所有容器
>
> -q：查看处于终止状态的容器
>
> docker logs IMAGE：查看容器的输出日志

### 终止容器

> docker stop IMAGE：终止一个运行中的容器
>
> docker start IMAGE：启动容器
>
> docker restart IMAGE：重启容器

### 进入容器

> docker exec -it CONTAINERID /bin/bash：进入容器

### 删除容器

> docker rm [OPTIONS] CONTAINER [CONTAINER...]：删除容器
>
> -f：强行终止并删除运行中的容器
>
> -l：删除容器的连接，但保留容器
>
> -v：删除容器挂在的数据卷

### 文件拷贝

> docker cp 需要拷贝的文件或目录	容器名称:容器目录：将文件拷贝到容器中
>
> docker cp 容器名称:容器目录	需要拷贝的文件或目录：将容器中的文件拷贝到宿主机

### 查看容器IP地址

> docker	inspect	容器名称（ID）：查看容器运行的各种数据
>
> docker 	inspect	--format='{{.NetworkSettings.IPAddress}}'	容器名称（ID）：查看容器IP地址