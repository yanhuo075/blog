---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 键
order: 5
---

# Redis 键(key)

Redis key 作为参数与 Redis 命令配合使用。

Redis 键的类型, redis是key-value的数据，所以每个数据都是一个键值对，键的类型是字符串。

**语法：**

```
redis 127.0.0.1:6379> COMMAND KEY_NAME 
```

**例**

让我们以 Redis DEL 命令为例。如果键被删除，它将给出输出 1，否则它将为 0。

```
redis 127.0.0.1:6379> SET mykey redis.com.cn   
OK   
redis 127.0.0.1:6379> DEL mykey  
(integer) 1 
```

这里，[DEL](https://redis.com.cn/commands/del.html) 是 Redis 命令，而 `mykey` 是键。

------



## Redis keys 命令

下表列出了 Redis 键相关的命令:

| 命令                                                      | 描述                                                  |
| :-------------------------------------------------------- | :---------------------------------------------------- |
| [DEL](https://redis.com.cn/commands/del.html)             | 用于删除 key                                          |
| [DUMP](https://redis.com.cn/commands/dump.html)           | 序列化给定 key ，并返回被序列化的值                   |
| [EXISTS](https://redis.com.cn/commands/exists.html)       | 检查给定 key 是否存在                                 |
| [EXPIRE](https://redis.com.cn/commands/expire.html)       | 为给定 key 设置过期时间                               |
| [EXPIREAT](https://redis.com.cn/commands/expireat.html)   | 用于为 key 设置过期时间，接受的时间参数是 UNIX 时间戳 |
| [PEXPIRE](https://redis.com.cn/commands/pexpire.html)     | 设置 key 的过期时间，以毫秒计                         |
| [PEXPIREAT](https://redis.com.cn/commands/pexpireat.html) | 设置 key 过期时间的时间戳(unix timestamp)，以毫秒计   |
| [KEYS](https://redis.com.cn/commands/keys.html)           | 查找所有符合给定模式的 key                            |
| [MOVE](https://redis.com.cn/commands/move.html)           | 将当前数据库的 key 移动到给定的数据库中               |
| [PERSIST](https://redis.com.cn/commands/persist.html)     | 移除 key 的过期时间，key 将持久保持                   |
| [PTTL](https://redis.com.cn/commands/pttl.html)           | 以毫秒为单位返回 key 的剩余的过期时间                 |
| [TTL](https://redis.com.cn/commands/ttl.html)             | 以秒为单位，返回给定 key 的剩余生存时间(              |
| [RANDOMKEY](https://redis.com.cn/commands/randomkey.html) | 从当前数据库中随机返回一个 key                        |
| [RENAME](https://redis.com.cn/commands/rename.html)       | 修改 key 的名称                                       |
| [RENAMENX](https://redis.com.cn/commands/renamenx.html)   | 仅当 newkey 不存在时，将 key 改名为 newkey            |
| [TYPE](https://redis.com.cn/commands/type.html)           | 返回 key 所储存的值的类型                             |
