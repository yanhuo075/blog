---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 哈希
order: 7
---

# Redis 哈希 ( Hashes )

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

每个哈希键中可以存储多达 40 亿个字段值对。



### 例

```
127.0.0.1:6379> HMSET mysite name "redis" url "https://redis.com.cn" rank 1 visitors 240000000
OK
127.0.0.1:6379> HGETALL mysite 
1) "name"
2) "mysite"
3) "url"
4) "https://redis.com.cn"
5) "rank"
6) "1"
7) "visitors"
8) "230000000"
```

在上面的例子中，`mysite` 是 Redis 哈希列表，它包含详细信息（name，url，rank，visitors）属性。

------



## Redis哈希命令

| 命令                                                         | 描述                                                      |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| [HDEL](https://redis.com.cn/commands/hdel.html)              | 删除一个或多个哈希表字段                                  |
| [HEXISTS](https://redis.com.cn/commands/hexists.html)        | 查看哈希表 key 中，指定的字段是否存在                     |
| [HGET](https://redis.com.cn/commands/hget.html)              | 获取存储在哈希表中指定字段的值                            |
| [HGETALL](https://redis.com.cn/commands/hgetall.html)        | 获取在哈希表中指定 key 的所有字段和值                     |
| [HINCRBY](https://redis.com.cn/commands/hincrby.html)        | 为哈希表 key 中的指定字段的整数值加上增量 increment       |
| [HINCRBYFLOAT](https://redis.com.cn/commands/hincrbyfloat.html) | 为哈希表 key 中的指定字段的浮点数值加上增量 increment     |
| [HKEYS](https://redis.com.cn/commands/hkeys.html)            | 获取所有哈希表中的字段                                    |
| [HLEN](https://redis.com.cn/commands/hlen.html)              | 获取哈希表中字段的数量                                    |
| [HMGET](https://redis.com.cn/commands/hmget.html)            | 获取所有给定字段的值                                      |
| [HMSET](https://redis.com.cn/commands/hmset.html)            | 同时将多个 field-value (域-值)对设置到哈希表 key 中       |
| [HSET](https://redis.com.cn/commands/hset.html)              | 将哈希表 key 中的字段 field 的值设为 value                |
| [HSETNX](https://redis.com.cn/commands/hsetnx.html)          | 只有在字段 field 不存在时，设置哈希表字段的值             |
| [HVALS](https://redis.com.cn/commands/hvals.html)            | 获取哈希表中所有值                                        |
| [HSCAN](https://redis.com.cn/commands/hscan.html)            | 迭代哈希表中的键值对                                      |
| [HSTRLEN](https://redis.com.cn/commands/hstrlen.html)        | 返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度 |
