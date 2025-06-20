---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 配置文件详解
order: 3
---

# Redis 配置文件详解

在 Redis 中，Redis 的根目录中有一个配置文件（redis.conf）。您可以通过 Redis CONFIG 命令获取和设置所有 Redis 配置。



## 查看Redis配置

**句法**

CONFIG GET 命令的基本语法:

```
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
```

查看日志等级的配置

```
redis 127.0.0.1:6379> CONFIG GET loglevel
```

**输出**

```
1)  "loglevel"
2)  "verbose"
```

![Redis配置1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-configuration1-1.png)

要获取所有配置设置，请使用 `*` 代替 `CONFIG_SETTING_NAME`

```
redis 127.0.0.1:6379> CONFIG GET *
```

**输出**

```
1.  "dir"
2.  "C:\\\\Program Files\\\\Redis"
3.  "dbfilename"
4.  "dump.rdb"
5.  "requirepass"
6.  (nil)
7.  "masterauth"
8.  (nil)
9.  "maxmemory"
10. "0"
11. "maxmemory-policy"
12. "volatile-lru"
13. "maxmemory-samples"
14. "3"
15. "timeout"
16. "0"
17. "appendonly"
18. "no"
19. "no-appendfsync-on-rewrite"
20. "no"
21. "appendfsync"
22. "everysec"
23. "save"
24. "3600 1 300 100 60 10000"
25. "auto-aof-rewrite-percentage"
26. "100"
27. "auto-aof-rewrite-min-size"
28. "1048576"
29. "slave-serve-stale-data"
30. "yes"
31. "hash-max-zipmap-entries"
32. "512"
33. "hash-max-zipmap-value"
34. "64"
35. "list-max-ziplist-entries"
36. "512"
37. "list-max-ziplist-value"
38. "64"
39. "set-max-intset-entries"
40. "512"
41. "zset-max-ziplist-entries"
42. "128"
43. "zset-max-ziplist-value"
44. "64"
45. "slowlog-log-slower-than"
46. "10000"
47. "slowlog-max-len"
48. "64"
49. "loglevel"
50. "verbose"
```

![Redis配置2](https://www.redis.com.cn/wp-content/uploads/2020/03/redis-configuration2-1.png)\

![Redis配置3](https://www.redis.com.cn/wp-content/uploads/2020/03/redis-configuration3-1.png)

------



## 编辑修改Redis配置

要更新配置，可以直接编辑 redis.conf 文件，也可以通过 CONFIG set 命令更新配置。

**语法**

以下是 CONFIG SET 命令的基本语法。

```
redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
```

示例如下：

```
127.0.0.1:6379> CONFIG SET loglevel "verbose"
OK
127.0.0.1:6379> CONFIG GET loglevel
1) "loglevel"
2) "verbose"
```

Redis 的日志级别有以下四种：

- debug：会打印出很多信息，适用于开发和测试阶段。
- verbose（冗长的）：包含很多不太有用的信息，但比debug简化一些。
- notice：适用于生产模式。
- warning : 警告信息。

Redis 默认设置为 verbose，开发测试阶段可以用 debug，生产模式一般选用 notice。



## Redis 常用配置参数说明

redis.conf 配置项说明如下：

| 参数                        | 描述                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| **daemonize**               | Redis 默认不是以守护进程的方式运行 **daemonize no**，可以通过该配置项修改，使用 yes 启用守护进程 |
| **pidfile**                 | 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定 **pidfile /var/run/redis.pid** |
| **port**                    | **port 6379** 指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字 |
| **bind**                    | 监听的主机地址 **bind 127.0.0.1**                            |
| **timeout**                 | **timeout 300** 当客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能 |
| **loglevel**                | **loglevel verbose** 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 verbose |
| **logfile**                 | **logfile stdout** 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null |
| **databases**               | **databases 16** 设置数据库的数量，默认数据库为 0，可以使用 SELECT `<dbid>` 命令在连接上指定数据库 id |
| **save**                    | **save `<seconds>` `<changes>`** 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合, Redis 默认配置文件中提供了三个条件：**save 900 1**、**save 300 10**、**save 60 10000** ,分别表示 90 0 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。 |
| **rdbcompression**          | **rdbcompression yes** 10.指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| **dbfilename**              | **dbfilename dump.rdb** 指定本地数据库文件名，默认值为 dump.rdb |
| **dir**                     | **dir ./** 指定本地数据库存放目录                            |
| **slaveof**                 | **slaveof `<masterip>` `<masterport>`** 设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步 |
| **masterauth**              | **masterauth `<master-password>`** master 服务设置了密码保护时，slave 服务连接 master 的密码 |
| **requirepass**             | **requirepass foobared** 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH `<password>`命令提供密码，默认关闭 |
| **maxclients**              | **maxclients 128** 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
| **maxmemory**               | **maxmemory `<bytes>`** 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
| **appendonly**              | **appendonly no** 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no |
| **appendfilename**          | **appendfilename appendonly.aof** 指定更新日志文件名，默认为 appendonly.aof |
| **appendfsync**             | **appendfsync everysec** 指定更新日志条件，共有 3 个可选值：**no**：表示等操作系统进行数据缓存同步到磁盘（快）; **always**：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全）; **everysec**：表示每秒同步一次（折衷，默认值） |
| **vm-enabled**              | **vm-enabled no** 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制） |
| **vm-swap-file**            | **vm-swap-file /tmp/redis.swap** 虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享 |
| **vm-max-memory**           | **vm-max-memory 0** 将所有大于 vm-max-memory 的数据存入虚拟内存,无论 vm-max-memory 设置多小,所有索引数据都是内存存储的( Redis 的索引数据就是 keys ),也就是说,当 vm-max-memory 设置为 0 的时候,其实是所有 value 都存在于磁盘。默认值为 0 |
| **vm-page-size**            | **vm-page-size 32** Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64 bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
| **vm-pages**                | **vm-pages 134217728** 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
| **vm-max-threads**          | **vm-max-threads 4** 设置访问 swap 文件的线程数,最好不要超过机器的核数,如果设置为 0,那么所有对 swap 文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为 4 |
| **glueoutputbuf**           | **glueoutputbuf yes** 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
| **hash-max-zipmap-entries** | **hash-max-zipmap-entries 64** 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| **hash-max-zipmap-value**   | **sh-max-zipmap-value 512** 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| **activerehashing**         | **activerehashing yes** 指定是否激活重置哈希，默认为开启（后面在介绍 Redis 的哈希算法时具体介绍） |
| **include**                 | **include /path/to/local.conf** 指定包含其它的配置文件，可以在同一主机上多个 Redis 实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |
