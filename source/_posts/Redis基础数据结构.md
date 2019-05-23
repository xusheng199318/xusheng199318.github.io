---
title: Redis基础数据结构
date: 2018-12-15 22:06:01
tags: Redis
---

#### String（字符串）

命令基本格式：

~~~c
set key value [ex seconds] [px milliseconds] [nx|xx]
ex seconds：为键设置秒级过期时间
px millisecondds：为键设置毫秒级过期时间
nx：键必须不能存在，才可以设置成功，用于添加
xx：键必须存在，才可以设置成功，用于更新
~~~



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

不常用命令：

~~~c
append key value ：向字符串尾部追加值
strlen key ：字符串长度
getset key value ：设置并返回原值
setrange key offset value ：设置指定位置的字符
getrange key start end ：获取部分字符串
~~~

> Redis的字符串内部结构类似Java中的ArrayList，当字符串长度小于1M时，扩容是加倍现有空间，如果超过1M，扩容时只会增加1M空间
>
> 值最大不能超过512M
>
> **内部编码：**
>
> int ：8个字节的长整形
>
> embstr ：小于等于39个字节的字符串
>
> raw ：大于39个字节的字符串

#### List（列表）

基本命令格式：

> rpush key value [value ...]
>
> lpush key value [value ...]
>
> linsert key before|after pivot value
>
> lrange key start end：包含start和end
>
> lindex key index
>
> llen key
>
> lpop key
>
> rpop key
>
> lrem key count value：lrem命令会从列表中找到等于value的元素进行删除。count > 0 从做到右，删除最多count个元素。count < 0 从右到左，删除最多count绝对值个元素，count = 0删除所有
>
> ltrim key start end：按照索引范围修剪列表
>
> lset key index newValue
>
> blpop key [key ...] timeout
>
> brpop key [key ...] timeout

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

基本命令格式

>hset key field value
>
>hget key field
>
>hdel key field [field...]
>
>hlen key
>
>hgetall key
>
>hmset key field value [field value]
>
>hmget key field [field...]
>
>hexists key
>
>hkeys key：获取所有的field
>
>hvals key：获取所有的value
>
>hgetall key
>
>hsetnx key field value 
>
>hincrby key field increment
>
>hstrlen key field

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
>
> ziplist：当哈希类型的元素个数小于hash-max-ziplist-entries（默认512）时，ziplist更加紧凑，多个元素连续存储
>
> hashtable：当哈希类型无法满足ziplist的条件时，使用hashtable。例如：当value大于64字节时，或者元素个数大于512时

#### Set（集合）

基本命令格式

> sadd key element [element]
>
> srem key element [element]
>
> scard key：计算元素个数
>
> sismember key element：判断元素是否在集合中
>
> srandmember key [count]：随机从集合中返回count（默认1）个元素
>
> spop key：从集合中随机弹出一个元素
>
> smembers key：获取所有元素，属于较重命令，可以使用sscan代替
>
> spop key：从集合随机弹出元素
>
> smembers key：获取所有元素，返回结果是无序的
>
> sinter key [key...]：求集合的交集
>
> sunion key [key...]：求集合的并集
>
> sdiff key [key...]：差集
>
> sinterstore destination key [key...]
>
> suniontstore destination key [key...]
>
> sdiffstore destination key [key...]

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
>
> intset：当集合中的元素都是整数切元素个数小于set-max-intset-entried时，内部编码为intset
>
> hashtable：当元素个数超过512个时或当某个元素不为整数时，内部编码为hashtable

#### Zset（有序集合）

基本命令格式

> zadd key score member [score memeber...]
>
> zcard key
>
> zscore key member
>
> zrank key member：计算成员的排名，正序
>
> zrevrank key member
>
> zrem key member [member...]
>
> zincrby key increment member
>
> zrange key start end [withscores]：返回指定排名范围的成员
>
> zrebrange key start end [withscores]
>
> zrangescore key min max [withscores] \[limit offset count]：返回指定分数范围的成员，min和max支持开区间（小括号），闭区间（中括号），-inf（无穷小）,+inf（无限大）
>
> zrevrangebyscore key max min [withscores] \[limit offset count]
>
> zcount key min max：返回指定分数范围成员个数
>
> zremrangebyrank key start end：删除制定排名内的升序元素
>
> zremrangebyscore key min max：删除制定分数范围的成员
>
> zinterstore destination numkeys key [key...] \[weights weight [weight]] \[aggregate sum|min|max]

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

> ziplist：当有序集合的元素个数小于`zset-max-ziplist-entries`默认128，同时每个元素的值都小于`zset-max-ziplist-value`默认64字节时，使用ziplist
>
> skiplist：当ziplist条件不满足时，会转为skiplist

#### 键管理

> rename key newkey：键重命名，如果rename之前，键已经存在，那么它的值也将被覆盖，推荐使用`renamenx`命令
>
> randomkey：随机返回一个键
>
> expire key seconds：键在多少秒后过期

#### 迁移键

> dump + restore：在源redis上，dump命令会将键值序列化，格式采用的是RDB格式，然后在目标redis上通过restore命令将上面序列化的值进行复原。dump key，restore key ttl value
>
> migrate：migrate host port key|"" destination-db timeout [copy] \[replace] \[keys key [key...]]
>
> migrate 127.0.0.1 6380 "" 0 5000 keys key1 key2 key3

#### 遍历键

> keys pattern：*代表匹配任意字符，?代表匹配一个字符，[]代表匹配部分字符，[1-10]代表匹配1到10，[1,3]代表匹配1或3，\x代表转义，例如要匹配星号、问号需要转义。但是由于redis是单线程架构，所以如果包含了大量的键，很可能会造成redis阻塞，所以在生产环境尽量避免使用`keys`
>
> scan cursor [match pattern] \[count number]：count代表每次遍历键的个数，scan方法是渐进式遍历，cursor从0开始遍历，每次返回当前游标的值，直到游标值为0的时候则遍历结束。在遍历期间如果有键值变化则无法精确统计

