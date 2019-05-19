---
title: 初识Redis
date: 2019-05-19 21:38:41
tags: Redis
---

### Redis基础数据结构

> string：字符串
>
> hash：哈希
>
> list：列表
>
> set：集合
>
> zset：有序集合
>
> bitmap：位图
>
> HyperLogLog：
>
> GEO：地理信息定位

### Redis使用场景

> 1. 缓存
> 2. 排行榜系统
> 3. 计数器应用
> 4. 社交网络
> 5. 消息队列系统
> 6. 分布式锁

### Redis不可以做什么

> 将冷数据当做缓放在`Redis`中
>
> 将经常需要修改的数据当做缓存放在`Redis`中

### Redis持久化策略

> AOF：基于日志
>
> RDB：基于快照

### Redis配置、启动、操作、关闭

> `redis-server [redis.conf path]`：redis启动命令,如果没有设置redis.conf的路径则默认加载默认路径下的redis.conf文件
>
> `redis-cli -h ${host} -p ${port}`：交互式链接redis
>
> `redis-clid shutdown`：停止redis服务

















































