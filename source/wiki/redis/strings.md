---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 字符串
order: 6
---

# Redis 字符串 ( Strings )

Redis 字符串命令用于管理 Redis 中的字符串值。

**语法：**

```
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

**例**

```
redis 127.0.0.1:6379> SET mykey redis.com.cn  
OK   
redis 127.0.0.1:6379> GET mykey   
"redis.com.cn"  
```

这里，SET 和 GET 是命令，`mykey` 是键。

------



## Redis 字符串命令

以下是一些用于在 Redis 中管理字符串的基本命令的列表：

| 命令                                                         | 描述                                                        |
| :----------------------------------------------------------- | :---------------------------------------------------------- |
| [SET](https://redis.com.cn/commands/set.html)                | 设置指定 key 的值                                           |
| [GET](https://redis.com.cn/commands/get.html)                | 获取指定 key 的值                                           |
| [GETRANGE](https://redis.com.cn/commands/getrange.html)      | 返回 key 中字符串值的子字符                                 |
| [GETSET](https://redis.com.cn/commands/getset.html)          | 将给定 key 的值设为 value ，并返回 key 的旧值 ( old value ) |
| [GETBIT](https://redis.com.cn/commands/getbit.html)          | 对 key 所储存的字符串值，获取指定偏移量上的位 ( bit )       |
| [MGET](https://redis.com.cn/commands/mget.html)              | 获取所有(一个或多个)给定 key 的值                           |
| [SETBIT](https://redis.com.cn/commands/setbit.html)          | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)    |
| [SETEX](https://redis.com.cn/commands/setex.html)            | 设置 key 的值为 value 同时将过期时间设为 seconds            |
| [SETNX](https://redis.com.cn/commands/setnx.html)            | 只有在 key 不存在时设置 key 的值                            |
| [SETRANGE](https://redis.com.cn/commands/setrange.html)      | 从偏移量 offset 开始用 value 覆写给定 key 所储存的字符串值  |
| [STRLEN](https://redis.com.cn/commands/strlen.html)          | 返回 key 所储存的字符串值的长度                             |
| [MSET](https://redis.com.cn/commands/mset.html)              | 同时设置一个或多个 key-value 对                             |
| [MSETNX](https://redis.com.cn/commands/msetnx.html)          | 同时设置一个或多个 key-value 对                             |
| [PSETEX](https://redis.com.cn/commands/psetex.html)          | 以毫秒为单位设置 key 的生存时间                             |
| [INCR](https://redis.com.cn/commands/incr.html)              | 将 key 中储存的数字值增一                                   |
| [INCRBY](https://redis.com.cn/commands/incrby.html)          | 将 key 所储存的值加上给定的增量值 ( increment )             |
| [INCRBYFLOAT](https://redis.com.cn/commands/incrbyfloat.html) | 将 key 所储存的值加上给定的浮点增量值 ( increment )         |
| [DECR](https://redis.com.cn/commands/decr.html)              | 将 key 中储存的数字值减一                                   |
| [DECRBY](https://redis.com.cn/commands/decrby.html)          | 将 key 所储存的值减去给定的减量值 ( decrement )             |
| [APPEND](https://redis.com.cn/commands/append.html)          | 将 value 追加到 key 原来的值的末尾                          |
