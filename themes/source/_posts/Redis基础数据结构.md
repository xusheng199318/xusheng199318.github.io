---
title: Redis基础数据结构
date: 2018-12-15 22:06:01
tags: Redis
---

#### String（字符串）

```c
127.0.0.1:6379> set name test1
OK
127.0.0.1:6379> get name
"test1"
127.0.0.1:6379> del name 
(integer) 1
127.0.0.1:6379> get name
(nil)
```

批量插入字符串：

```c
127.0.0.1:6379> mset name1 test1 name2 test2
OK
127.0.0.1:6379> mget name1 name2
1) "test1"
2) "test2"

```

设置过期时间：

```c
127.0.0.1:6379> set name test2
OK
127.0.0.1:6379> get name
"test2"
127.0.0.1:6379> expire name 5
(integer) 1
127.0.0.1:6379> ttl name
(integer) 3
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) 0
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
    
与上述代码等效 
127.0.0.1:6379> setex name 5 test1
OK


```

如果key不存在就创建

```c
127.0.0.1:6379> setnx name test1
(integer) 1
127.0.0.1:6379> get name
"test1"
127.0.0.1:6379> setnx name test2
(integer) 0
127.0.0.1:6379> get name
"test1"

```

计数：如果value是数字可以进行增加操作

```c
127.0.0.1:6379> set age 10
OK
127.0.0.1:6379> incr age
(integer) 11
127.0.0.1:6379> get age
"11"
127.0.0.1:6379> incr age 
(integer) 12
127.0.0.1:6379> get age
"12"
127.0.0.1:6379> incrby age -3
(integer) 9
127.0.0.1:6379> incrby age 3
(integer) 12
```

> Redis的字符串内部结构类似Java中的ArrayList，当字符串长度小于1M时，扩容是加倍现有空间，如果超过1M，扩容时只会增加1M空间

#### List（列表）

右边进左边出：

```c
127.0.0.1:6379> rpush name test1 test2 test3
(integer) 3
127.0.0.1:6379> llen name
(integer) 3
127.0.0.1:6379> lpop name
"test1"
127.0.0.1:6379> lpop name
"test2"
127.0.0.1:6379> lpop name
"test3"
127.0.0.1:6379> lpop name
(nil)

```

右边进右边出：

```c
127.0.0.1:6379> rpush name test1 test2 test3
(integer) 3
127.0.0.1:6379> rpop name
"test3"
127.0.0.1:6379> rpop name
"test2"
127.0.0.1:6379> rpop name
"test1"
127.0.0.1:6379> rpop name
(nil)

```

慢查询：

`lindex`相当于Java链表的get（int index）方法，需要对链表进行遍历，index可以为负数，index=-1表示倒数第一个元素

`ltrim`在区间内的保留，区间外的全部删除，可以通过`ltrim`实现定长链表

```c
127.0.0.1:6379> rpush name test1 test2 test3
(integer) 3
127.0.0.1:6379> lindex name
(error) ERR wrong number of arguments for 'lindex' command
127.0.0.1:6379> lindex name 1
"test2"
127.0.0.1:6379> lindex name 0
"test1"
 127.0.0.1:6379> lrange name 0 -1
1) "test1"
2) "test2"
3) "test3"
127.0.0.1:6379> ltrim name 1 -1
OK
127.0.0.1:6379> lrange name 0 -1
1) "test2"
2) "test3"   

```

#### Hash（字典）

```c
127.0.0.1:6379> hset name user1 test1
(integer) 1
127.0.0.1:6379> hget name user1
"test1"
127.0.0.1:6379> hset name user2 test2
(integer) 1
127.0.0.1:6379> hset name user3 test3
(integer) 1
127.0.0.1:6379> hgetall name
1) "user1"
2) "test1"
3) "user2"
4) "test2"
5) "user3"
6) "test3"
127.0.0.1:6379> hlen name
(integer) 3
127.0.0.1:6379> hset name user3 test33
(integer) 0
127.0.0.1:6379> hget name user3
"test33"
127.0.0.1:6379> hmset users name1 test1 name2 test2 name3 test3
OK
127.0.0.1:6379> hgetall users
1) "name1"
2) "test1"
3) "name2"
4) "test2"
5) "name3"
6) "test3"

```

增加操作：

```c
127.0.0.1:6379> hmset user age 1
OK
127.0.0.1:6379> hincrby user age 1
(integer) 2
```

> Redis的字典相当于Java中的HashMap，Redis采用了渐进式rehash策略

#### Set（集合）

```c
127.0.0.1:6379> sadd users test1
(integer) 1
127.0.0.1:6379> sadd users test2
(integer) 1
127.0.0.1:6379> sadd users test3
(integer) 1
127.0.0.1:6379> smembers users
1) "test3"
2) "test1"
3) "test2"
127.0.0.1:6379> sismember users test2
(integer) 1
127.0.0.1:6379> scard users
(integer) 3
127.0.0.1:6379> spop users
"test2"

```

> Set集合相当于Java语言里的HashSet，无序且唯一

#### Zset（有序集合）

```c
127.0.0.1:6379> zadd users 9 age
(integer) 1
127.0.0.1:6379> zadd users 10 age
(integer) 0
127.0.0.1:6379> zadd users 11 age
(integer) 0
127.0.0.1:6379> zrange user 0 -1
(empty list or set)
127.0.0.1:6379> zrange users 0 -1
1) "age"
127.0.0.1:6379> del users
(integer) 1
127.0.0.1:6379> clear
127.0.0.1:6379> zadd users 9 user1
(integer) 1
127.0.0.1:6379> zadd users 10 user2
(integer) 1
127.0.0.1:6379> zadd users 11 user3
(integer) 1
127.0.0.1:6379> zrange users 0 -1
1) "user1"
2) "user2"
3) "user3"
127.0.0.1:6379> zrevrange users 0 -1
1) "user3"
2) "user2"
3) "user1"
127.0.0.1:6379> zcard books
(integer) 0
127.0.0.1:6379> zcard users
(integer) 3
127.0.0.1:6379> zscore users user1
"9"
127.0.0.1:6379> zrangebyscore users 0 9
1) "user1"
127.0.0.1:6379> zrangebyscore users -inf 10 withscores
1) "user1"
2) "9"
3) "user2"
4) "10"
127.0.0.1:6379> zrem users user1
(integer) 1
127.0.0.1:6379> zrange users 0 -1
1) "user2"
2) "user3"
```

> Zset相当于Java的SortedSet和HashMap的结合，它有一个score代表这value的权重，内部使用的是一种叫`跳表`的数据结构。

