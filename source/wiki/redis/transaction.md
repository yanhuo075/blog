---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 事务
order: 12
---

# Redis 事务

事务是指一个完整的动作，要么全部执行，要么什么也没有做。

Redis 事务不是严格意义上的事务，只是用于帮助用户在一个步骤中执行多个命令。单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

Redis 事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

MULTI、EXEC、DISCARD、WATCH 这四个指令构成了 redis 事务处理的基础。

1. MULTI 用来组装一个事务；
2. EXEC 用来执行一个事务；
3. DISCARD 用来取消一个事务；
4. WATCH 用来监视一些 key，一旦这些 key 在事务执行之前被改变，则取消事务的执行。

在 Redis 中，通过使用MULTI命令启动事务，然后需要传递应在事务中执行的命令列表，之后整个事务由EXEC命令执行。

![Redis交易1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-transactions1-1.png)

![Redis交易2](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-transactions2-1.png)



## 例子

如何启动和执行 Redis 事务。

```
redis 127.0.0.1:6379> MULTI  
OK  
redis 127.0.0.1:6379> EXEC  
(empty list or set)  
redis 127.0.0.1:6379> MULTI  
OK  
redis 127.0.0.1:6379> SET rediscomcn redis  
QUEUED  
redis 127.0.0.1:6379> GET rediscomcn  
QUEUED  
redis 127.0.0.1:6379> INCR visitors  
QUEUED  
redis 127.0.0.1:6379> EXEC  
1) OK  
2) "redis"  
3) (integer) 1  
```

在上面的例子中，我们看到了 QUEUED 的字样，这表示我们在用 MULTI 组装事务时，每一个命令都会进入到内存队列中缓存起来，如果出现 QUEUED 则表示我们这个命令成功插入了缓存队列，在将来执行 EXEC 时，这些被 QUEUED 的命令都会被组装成一个事务来执行。

对于事务的执行来说，如果 redis 开启了 AOF 持久化的话，那么一旦事务被成功执行，事务中的命令就会通过 write 命令一次性写到磁盘中去，如果在向磁盘中写的过程中恰好出现断电、硬件故障等问题，那么就可能出现只有部分命令进行了 AOF 持久化，这时 AOF 文件就会出现不完整的情况，这时，我们可以使用 redis-check-aof 工具来修复这一问题，这个工具会将 AOF 文件中不完整的信息移除，确保 AOF 文件完整可用。

------



## Redis 事务错误

有关事务，大家经常会遇到的是两类错误：

1. 调用 EXEC 之前的错误
2. 调用 EXEC 之后的错误

调用 EXEC 之前的错误，有可能是由于语法有误导致的，也可能时由于内存不足导致的。只要出现某个命令无法成功写入缓冲队列的情况，redis 都会进行记录，在客户端调用 EXEC 时，redis 会拒绝执行这一事务。（这是 2.6.5 版本之后的策略。在 2.6.5 之前的版本中，redis 会忽略那些入队失败的命令，只执行那些入队成功的命令）。我们来看一个这样的例子：

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> haha //一个明显错误的指令
(error) ERR unknown command 'haha'
127.0.0.1:6379> ping
QUEUED
127.0.0.1:6379> exec
//redis无情的拒绝了事务的执行，原因是“之前出现了错误”
(error) EXECABORT Transaction discarded because of previous errors.
```

而对于调用 EXEC 之后的错误，redis 则采取了完全不同的策略，即 redis 不会理睬这些错误，而是继续向下执行事务中的其他命令。这是因为，对于应用层面的错误，并不是 redis 自身需要考虑和处理的问题，所以一个事务中如果某一条命令执行失败，并不会影响接下来的其他命令的执行。我们也来看一个例子：

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 23
QUEUED
//age不是集合，所以如下是一条明显错误的指令
127.0.0.1:6379> sadd age 15 
QUEUED
127.0.0.1:6379> set age 29
QUEUED
127.0.0.1:6379> exec //执行事务时，redis不会理睬第2条指令执行错误
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get age
"29" //可以看出第3条指令被成功执行了
```

最后，我们来说说最后一个指令WATCH，这是一个很好用的指令，它可以帮我们实现类似于“乐观锁”的效果，即CAS（check and set）。

WATCH 本身的作用是监视 key 是否被改动过，而且支持同时监视多个 key，只要还没真正触发事务，WATCH 都会尽职尽责的监视，一旦发现某个 key 被修改了，在执行 EXEC 时就会返回 nil，表示事务无法触发。

```
127.0.0.1:6379> set age 23
OK
127.0.0.1:6379> watch age //开始监视age
OK
127.0.0.1:6379> set age 24 //在EXEC之前，age的值被修改了
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 25
QUEUED
127.0.0.1:6379> get age
QUEUED
127.0.0.1:6379> exec //触发EXEC
(nil) //事务无法被执行
```



## Redis 事务命令

下表列出了 Redis 事务的相关命令

| 命令    | 描述                                 |
| :------ | :----------------------------------- |
| DISCARD | 取消事务，放弃执行事务块内的所有命令 |
| EXEC    | 执行所有事务块内的命令               |
| MULTI   | 标记一个事务块的开始                 |
| UNWATCH | 取消 WATCH 命令对所有 key 的监视     |
| WATCH   | 监视一个(或多个) key                 |
