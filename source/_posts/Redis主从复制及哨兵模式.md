---
title: Redis主从复制及哨兵模式
date: 2018-12-15 22:06:29
tags: Redis
---

#### Redis主从复制

> 在数据量非常庞大的应用场景下，我们可以选择用Redis缓存部分热点数据，以减少对数据库的访问。
>
> 应用逻辑：
>
> 1. 从Redis中读取是否有想要的数据
> 2. 如果有，从Redis中将数据取出并返回
> 3. 没有，从数据库中读取相关数据，存入Redis并将数据返回给页面
>
> 如果在用户访问量非常大的时候Redis服务down了，并且没有配置持久化策略，那么当我们重新启动Redis服务的时候缓存中的数据也是空，即每一次请求都会去访问数据库，造成数据库的压力。

对于上述问题：可以选择Redis主从赋值

> - 当一个 master 实例和一个 slave 实例连接正常时， master 会发送一连串的命令流来保持对 slave 的更新，以便于将自身数据集的改变复制给 slave ， ：包括客户端的写入、key 的过期或被逐出等等。
> - 当 master 和 slave 之间的连接断开之后，因为网络问题、或者是主从意识到连接超时， slave 重新连接上 master 并会尝试进行部分重同步：这意味着它会尝试只获取在断开连接期间内丢失的命令流。
> - 当无法进行部分重同步时， slave 会请求进行全量重同步。这会涉及到一个更复杂的过程，例如 master 需要创建所有数据的快照，将之发送给 slave ，之后在数据集更改时持续发送命令流到 slave 。

配置：只需要在从机的`redis.conf`配置文件中进行配置

```c
# slaveof <masterip> <masterport>
slaveof 127.0.0.1 7001
```

#### 哨兵模式

Redis主从复制可以将master的数据复制到slave中，但是slave默认是只读的，无法在slave中进行set操作，即无法实现**自动故障转移**。通过哨兵模式可以实现：当master服务down了之后，slave替代master成为主机，当原master恢复后自动成为slave。

修改sentinel.conf文件：

```c
# sentinel monitor [master-group-name] [ip] [port] [quorum]
sentinel monitor mymaster 127.0.0.1 7001 1
```

通过以下命令运行哨兵：

```c
./redis-sentinel ../sentinel.conf
```

#### 哨兵工作模式

> - 每个 Sentinel 以**每秒钟一次**的频率向它所知的主服务器、从服务器以及其他 Sentinel 实例发送一个 PING 命令。
> - 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 `down-after-milliseconds` 选项所指定的值， 那么这个实例会被 Sentinel 标记为主观下线。 一个有效回复可以是： +PONG 、 -LOADING 或者 -MASTERDOWN 。
> - 如果一个主服务器被标记为主观下线， 那么正在监视这个主服务器的所有 Sentinel 要以**每秒一次**的频率确认主服务器的确进入了主观下线状态。
> - 如果一个主服务器被标记为主观下线， 并且有足够数量的 Sentinel （至少要达到配置文件指定的数量,`quorum`）在指定的时间范围内同意这一判断， 那么这个主服务器被标记为客观下线。
> - 在一般情况下， 每个 Sentinel 会以**每 10 秒一次**的频率向它已知的所有主服务器和从服务器发送 INFO 命令。 当一个主服务器被 Sentinel 标记为客观下线时， Sentinel 向下线主服务器的所有从服务器发送 INFO 命令的频率会从 **10 秒一次**改为**每秒一次**。
> - 当没有足够数量的 Sentinel 同意主服务器已经下线， 主服务器的客观下线状态就会被移除。 当主服务器重新向 Sentinel 的 PING 命令返回有效回复时， 主服务器的主管下线状态就会被移除。





