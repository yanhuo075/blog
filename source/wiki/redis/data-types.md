---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 数据类型
order: 3
---

# redis 支持的数据类型

Redis 数据库支持多种数据类型。

- 字符串（string）
- 哈希（hash）
- 列表（list）
- 集合（set）
- 有序集合（sorted set）
- 位图 ( Bitmaps )
- 基数统计 ( HyperLogLogs )

------



## 字符串

String 是一组字节。在 Redis 数据库中，字符串是二进制安全的。这意味着它们具有已知长度，并且不受任何特殊终止字符的影响。可以在一个字符串中存储最多 512 兆字节的内容。

示例：

使用 SET 命令在 name 键中存储字符串 `redis.com.cn`，然后使用 GET 命令查询 name。

```
SET name "redis.com.cn"  
OK  
GET name   
"redis.com.cn" 
```

在上面的例子中，SET 和 GET 是 Redis 命令，name 是 Redis 中使用的 key，`redis.com.cn` 是存储在 Redis 中的字符串值。

------



## 哈希

哈希是键值对的集合。在 Redis 中，哈希是字符串字段和字符串值之间的映射。因此，它们适合表示对象。

示例：

让我们存储一个用户的对象，其中包含用户的基本信息。

```
HMSET user:1 username ajeet password redis alexa 2000  
OK  
HGETALL  user:1  
"username"  
"ajeet"  
"password"  
"redis"  
"alexa"  
"2000" 
```

这里，HMSET 和 HGETALL 是 Redis 的命令，而 user：1 是键。

每个哈希可以存储多达 232 - 1 个键-值对。

------



## 列表

Redis 列表定义为字符串列表，按插入顺序排序。可以将元素添加到 Redis 列表的头部或尾部。

示例：

```
lpush rediscomcn java  
(integer) 1  
lpush rediscomcn sql 
(integer) 1  
lpush rediscomcn mongodb 
(integer) 1  
lpush rediscomcn cassandra 
(integer) 1  
lrange rediscomcn 0 10  
"cassandra"  
"mongodb"  
"sql"  
"java"  
```

列表的最大长度为 232 – 1 个元素（超过 40 亿个元素）。

------



## 集合

集合（set）是 Redis 数据库中的无序字符串集合。在 Redis 中，添加，删除和查找的时间复杂度是 O(1)。

示例：

```
sadd tutoriallist redis  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist sql  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 0  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 0  
redis 127.0.0.1:6379> smembers tutoriallist  
1) "redis"  
2) "postgresql"  
3) "sql" 
```

在上面的示例中，您可以看到 postgresql 被添加了三次，但由于该集的唯一属性，它只添加一次。

集合中的最大成员数为 232 -1 个元素（超过 40 亿个元素）。

------



## 有序集合

Redis 有序集合类似于 Redis 集合，也是一组非重复的字符串集合。但是，排序集的每个成员都与一个分数相关联，该分数用于获取从最小到最高分数的有序排序集。虽然成员是独特的，但可以重复分数。

示例：

```
redis 127.0.0.1:6379> zadd tutoriallist 0 redis  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 sql  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 0  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 0  
redis 127.0.0.1:6379> ZRANGEBYSCORE tutoriallist 0 10  
1) "postgresql"  
2) "redis"  
3) "sql"   
```

------



## 位图

Redis Bitmap 通过类似 map 结构存放 0 或 1 ( bit 位 ) 作为值。

Redis Bitmap 可以用来统计状态，如`日活`是否浏览过某个东西。



### Redis setbit 命令

Redis setbit 命令用于设置或者清除一个 bit 位。



#### [*](https://redis.com.cn/redis-data-types.html#redis-setbit-命令语法格式)Redis setbit 命令语法格式

```
SETBIT key offset value
```



#### [*](https://redis.com.cn/redis-data-types.html#范例)范例

```
127.0.0.1:6379> setbit aaa:001 10001 1 # 返回操作之前的数值
(integer) 0
127.0.0.1:6379> setbit aaa:001 10002 2 # 如果值不是0或1就报错
(error) ERR bit is not an integer or out of range
127.0.0.1:6379> setbit aaa:001 10002 0
(integer) 0
127.0.0.1:6379> setbit aaa:001 10003 1
(integer) 0
```

------



## 基数统计

Redis HyperLogLog 可以接受多个元素作为输入，并给出输入元素的基数估算值

- 基数

集合中不同元素的数量，比如 {'apple', 'banana', 'cherry', 'banana', 'apple'} 的基数就是 3

- 估算值

算法给出的基数并不是精确的，可能会比实际稍微多一些或者稍微少一些，但会控制在合 理的范围之内

HyperLogLog 的优点是：**即使输入元素的数量或者体积非常非常大，计算基数所需的空间总是固定的、并且是很小的**。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 264 个不同元素的基数。

这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。



### Redis PFADD 命令

Redis PFADD 命令将元素添加至 HyperLogLog



#### [*](https://redis.com.cn/redis-data-types.html#redis-pfadd-命令语法格式)Redis PFADD 命令语法格式

```
PFADD key element [element ...]
```



#### [*](https://redis.com.cn/redis-data-types.html#范例)范例

```
127.0.0.1:6379> PFADD unique::ip::counter '192.168.0.1'
(integer) 1
127.0.0.1:6379> PFADD unique::ip::counter '127.0.0.1'
(integer) 1
127.0.0.1:6379> PFADD unique::ip::counter '255.255.255.255'
(integer) 1
127.0.0.1:6379> PFCOUNT unique::ip::counter
(integer) 3
```
