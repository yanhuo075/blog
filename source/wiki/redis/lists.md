---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 列表
order: 8
---

# Redis 列表 ( Lists )

Redis 列表是按插入顺序排序的字符串列表。可以在列表的头部（左边）或尾部（右边）添加元素。

列表可以包含超过 40 亿 个元素 ( 232 - 1 )。

**例**

用 LPUSH 命令将三个值插入了名为 language 的列表当中:

```
redis 127.0.0.1:6379> LPUSH rediscomcn sql  
(integer) 1  
redis 127.0.0.1:6379> LPUSH rediscomcn mysql  
(integer) 2  
redis 127.0.0.1:6379> LPUSH rediscomcn cassandra  
(integer) 3  
redis 127.0.0.1:6379> LRANGE rediscomcn 0 10  
1) "cassandra"  
2) "mysql"  
3) "sql"  
redis 127.0.0.1:6379>  
```

------



## Redis 列表命令

下表列出了列表相关命令：

| 命令                                                        | 描述                                                     |
| :---------------------------------------------------------- | :------------------------------------------------------- |
| [BLPOP](https://redis.com.cn/commands/blpop.html)           | 移出并获取列表的第一个元素                               |
| [BRPOP](https://redis.com.cn/commands/brpop.html)           | 移出并获取列表的最后一个元素                             |
| [BRPOPLPUSH](https://redis.com.cn/commands/brpoplpush.html) | 从列表中弹出一个值，并将该值插入到另外一个列表中并返回它 |
| [LINDEX](https://redis.com.cn/commands/lindex.html)         | 通过索引获取列表中的元素                                 |
| [LINSERT](https://redis.com.cn/commands/linsert.html)       | 在列表的元素前或者后插入元素                             |
| [LLEN](https://redis.com.cn/commands/llen.html)             | 获取列表长度                                             |
| [LPOP](https://redis.com.cn/commands/lpop.html)             | 移出并获取列表的第一个元素                               |
| [LPUSH](https://redis.com.cn/commands/lpush.html)           | 将一个或多个值插入到列表头部                             |
| [LPUSHX](https://redis.com.cn/commands/lpushx.html)         | 将一个值插入到已存在的列表头部                           |
| [LRANGE](https://redis.com.cn/commands/lrange.html)         | 获取列表指定范围内的元素                                 |
| [LREM](https://redis.com.cn/commands/lrem.html)             | 移除列表元素                                             |
| [LSET](https://redis.com.cn/commands/lset.html)             | 通过索引设置列表元素的值                                 |
| [LTRIM](https://redis.com.cn/commands/ltrim.html)           | 对一个列表进行修剪(trim)                                 |
| [RPOP](https://redis.com.cn/commands/rpop.html)             | 移除并获取列表最后一个元素                               |
| [RPOPLPUSH](https://redis.com.cn/commands/rpoplpush.html)   | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回 |
| [RPUSH](https://redis.com.cn/commands/rpush.html)           | 在列表中添加一个或多个值                                 |
| [RPUSHX](https://redis.com.cn/commands/rpushx.html)         | 为已存在的列表添加值                                     |
