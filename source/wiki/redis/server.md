---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 服务器
order: 15
---

# Redis 服务器

Redis Server 命令用于管理 Redis 服务器。

有不同的服务器命令可用于获取服务器信息，统计信息和其他特征。



## 例

我们举一个例子来看看如何获取有关服务器的所有统计信息和信息。

```
redis 127.0.0.1:6379> ping  
PONG  
redis 127.0.0.1:6379> AUTH "password"  
(error) ERR Client sent AUTH, but no password is set  
redis 127.0.0.1:6379> PING  
PONG  
redis 127.0.0.1:6379> ECHO "Welcome to rediscomcn"  
"Welcome to rediscomcn"  
redis 127.0.0.1:6379> INFO  
redis_version:2.4.6  
redis_git_sha1:26cdd13a  
redis_git_dirty:0  
arch_bits:64  
multiplexing_api:winsock2  
gcc_version:4.6.1  
process_id:6360  
uptime_in_seconds:4442  
uptime_in_days:0  
lru_clock:1716856  
used_cpu_sys:1.80  
used_cpu_user:0.42  
used_cpu_sys_children:0.00  
used_cpu_user_children:0.00  
connected_clients:1  
connected_slaves:0  
client_longest_output_list:0  
client_biggest_input_buf:0  
blocked_clients:0  
used_memory:1188152  
used_memory_human:1.13M  
used_memory_rss:1188152  
used_memory_peak:1188112  
used_memory_peak_human:1.13M  
mem_fragmentation_ratio:1.00  
mem_allocator:libc  
loading:0  
aof_enabled:0  
changes_since_last_save:0  
bgsave_in_progress:0  
last_save_time:1506142039  
bgrewriteaof_in_progress:0  
total_connections_received:1  
total_commands_processed:4  
expired_keys:0  
evicted_keys:0  
keyspace_hits:0  
keyspace_misses:0  
pubsub_channels:0  
pubsub_patterns:0  
latest_fork_usec:0  
vm_enabled:0  
role:master  
```

![Redis发布服务器1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-server1-1.png)

------



## Redis 管理 redis 服务相关命令

下表列出了管理 redis 服务相关的命令

| 命令             | 描述                                             |
| :--------------- | :----------------------------------------------- |
| BGREWRITEAOF     | 异步执行一个 AOF（AppendOnly File） 文件重写操作 |
| BGSAVE           | 在后台异步保存当前数据库的数据到磁盘             |
| CLIENT           | 关闭客户端连接                                   |
| CLIENT LIST      | 获取连接到服务器的客户端连接列表                 |
| CLIENT GETNAME   | 获取连接的名称                                   |
| CLIENT PAUSE     | 在指定时间内终止运行来自客户端的命令             |
| CLIENT SETNAME   | 设置当前连接的名称                               |
| CLUSTER SLOTS    | 获取集群节点的映射数组                           |
| COMMAND          | 获取 Redis 命令详情数组                          |
| COMMAND COUNT    | 获取 Redis 命令总数                              |
| COMMAND GETKEYS  | 获取给定命令的所有键                             |
| TIME             | 返回当前服务器时间                               |
| COMMAND INFO     | 获取指定 Redis 命令描述的数组                    |
| CONFIG GET       | 获取指定配置参数的值                             |
| CONFIG REWRITE   | 修改 redis.conf 配置文件                         |
| CONFIG SET       | 修改 redis 配置参数，无需重启                    |
| CONFIG RESETSTAT | 重置 INFO 命令中的某些统计数据                   |
| DBSIZE           | 返回当前数据库的 key 的数量                      |
| DEBUG OBJECT     | 获取 key 的调试信息                              |
| DEBUG SEGFAULT   | 让 Redis 服务崩溃                                |
| FLUSHALL         | 删除所有数据库的所有 key                         |
| FLUSHDB          | 删除当前数据库的所有 key                         |
| INFO             | 获取 Redis 服务器的各种信息和统计数值            |
| LASTSAVE         | 返回最近一次 Redis 成功将数据保存到磁盘上的时间  |
| MONITOR          | 实时打印出 Redis 服务器接收到的命令，调试用      |
| ROLE             | 返回主从实例所属的角色                           |
| SAVE             | 异步保存数据到硬盘                               |
| SHUTDOWN         | 异步保存数据到硬盘，并关闭服务器                 |
| SLAVEOF          | 将当前服务器转变从属服务器(slave server)         |
| SLOWLOG          | 管理 redis 的慢日志                              |
| SYNC             | 用于复制功能 ( replication ) 的内部命令          |
