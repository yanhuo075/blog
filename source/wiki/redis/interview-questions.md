---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 面试题
order: 1
---

# redis面试题

**Redis 常见面试问题**总结和答案。



## 什么是Redis？

Redis(Remote Dictionary Server) Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 [API](https://www.redis.com.cn/clients.html) 的非关系型数据库。

传统数据库遵循 ACID 规则。而 Nosql（Not Only SQL 的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称） 一般为分布式而分布式一般遵循 CAP 定理。

Github 源码：https://github.com/antirez/redis

Redis 官网：https://redis.io/

与传统数据库不同的是 Redis 的数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存，每秒可以处理超过 10 万次读写操作，是已知性能最快的 Key-Value 数据库。另外，Redis 也经常用来做分布式锁。除此之外，Redis 支持事务、持久化、LUA 脚本、LRU 驱动事件、多种集群方案。



## Redis支持的数据类型？

Redis 可以存储键和不同类型的值之间的映射。键的类型只能为字符串，值常见有五种数据类型：字符串、列表、集合、散列表、有序集合。

**String字符串**：

格式: set key value

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

**Hash（哈希）**

格式: hmset name key1 value1 key2 value2

Redis hash 是一个键值(key=>value)对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

**List（列表）**

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

格式: lpush name value

在 key 对应 list 的头部添加字符串元素

格式: rpush name value

在 key 对应 list 的尾部添加字符串元素

格式: lrem name index

key 对应 list 中删除 count 个和 value 相同的元素

格式: llen name

返回 key 对应 list 的长度

**Set（集合）**

格式: sadd name value

Redis的Set是string类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

**zset(sorted set：有序集合)**

格式: zadd name score value

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。



## 什么是Redis持久化？Redis有哪几种持久化方式？优缺点是什么？

持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。

Redis 提供了两种持久化方式:RDB（默认） 和AOF

**RDB：**

rdb是Redis DataBase缩写

功能核心函数rdbSave(生成RDB文件)和rdbLoad（从文件加载内存）两个函数

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq1.png)

**AOF:**

Aof是Append-only file缩写

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq2.png)

每当执行服务器(定时)任务或者函数时flushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作

aof写入保存：

WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件

SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。

**存储结构:**

内容是redis通讯协议(RESP )格式的命令文本存储。

**比较**：

1、aof文件比rdb更新频率高，优先使用aof还原数据。

2、aof比rdb更安全也更大

3、rdb性能比aof好

4、如果两个都配了优先加载AOF



## 刚刚上面你有提到redis通讯协议(RESP )，能解释下什么是RESP？有什么特点？（可以看到很多面试其实都是连环炮，面试官其实在等着你回答到这个点，如果你答上了对你的评价就又加了一分）

RESP 是redis客户端和服务端之前使用的一种通讯协议；

RESP 的特点：实现简单、快速解析、可读性好

For Simple Strings the first byte of the reply is "+" 回复

For Errors the first byte of the reply is "-" 错误

For Integers the first byte of the reply is ":" 整数

For Bulk Strings the first byte of the reply is "$" 字符串

For Arrays the first byte of the reply is "*" 数组



## Redis 有哪些架构模式？讲讲各自的特点

**单机版**

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq3.png)

特点：简单

问题：

1、内存容量有限 2、处理能力有限 3、无法高可用。

**主从复制**

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq4.png)

Redis 的复制（replication）功能允许用户根据一个 Redis 服务器来创建任意多个该服务器的复制品，其中被复制的服务器为主服务器（master），而通过复制创建出来的服务器复制品则为从服务器（slave）。 只要主从服务器之间的网络连接正常，主从服务器两者会具有相同的数据，主服务器就会一直将发生在自己身上的数据更新同步 给从服务器，从而一直保证主从服务器的数据相同。

特点：

1、master/slave 角色

2、master/slave 数据相同

3、降低 master 读压力在转交从库

问题：

无法保证高可用

没有解决 master 写的压力

**哨兵**

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq5.png)

Redis sentinel 是一个分布式系统中监控 redis 主从服务器，并在主服务器下线时自动进行故障转移。其中三个特性：

监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。

提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作。

特点：

1、保证高可用

2、监控各个节点

3、自动故障迁移

缺点：主从模式，切换需要时间丢数据

没有解决 master 写的压力

**集群（proxy 型）：**

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq6.png)

Twemproxy 是一个 Twitter 开源的一个 redis 和 memcache 快速/轻量级代理服务器； Twemproxy 是一个快速的单线程代理程序，支持 Memcached ASCII 协议和 redis 协议。

特点：1、多种 hash 算法：MD5、CRC16、CRC32、CRC32a、hsieh、murmur、Jenkins

2、支持失败节点自动删除

3、后端 Sharding 分片逻辑对业务透明，业务方的读写方式和操作单个 Redis 一致

缺点：增加了新的 proxy，需要维护其高可用。

failover 逻辑需要自己实现，其本身不能支持故障的自动转移可扩展性差，进行扩缩容都需要手动干预

**集群（直连型）：**

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/riq7.png)

从redis 3.0之后版本支持redis-cluster集群，Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

特点：

1、无中心架构（不存在哪个节点影响性能瓶颈），少了 proxy 层。

2、数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布。

3、可扩展性，可线性扩展到 1000 个节点，节点可动态添加或删除。

4、高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做备份数据副本

5、实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave到 Master 的角色提升。

缺点：

1、资源隔离性较差，容易出现相互影响的情况。

2、数据通过异步复制,不保证数据的强一致性



## 什么是一致性哈希算法？什么是哈希槽？

这两个问题篇幅过长 网上找了两个解锁的不错的文章

https://www.cnblogs.com/lpfuture/p/5796398.html

http://www.jasontec.cn/articles/2020/04/11/1586586130767.html



## Redis常用命令？

Keys pattern

列出所有key，*表示区配所有。

Set

设置 key 对应的值为 string 类型的 value。

setnx

设置 key 对应的值为 string 类型的 value。如果 key 已经存在，返回 0，nx 是 not exist 的意思。

Del

删除某个key，第一次返回1 删除了 第二次返回0

Expire

设置过期时间（单位秒）

TTL

查看剩下多少时间，返回负数则key失效，key不存在了

Setex

设置 key 对应的值为 string 类型的 value，并指定此键值对应的有效期。

Mset

一次设置多个 key 的值，成功返回 ok 表示所有的值都设置了，失败返回 0 表示没有任何值被设置。

Getset

设置 key 的值，并返回 key 的旧值。

Mget

一次获取多个 key 的值，如果对应 key 不存在，则对应返回 nil。

Incr

对 key 的值做加加操作,并返回新的值。注意 incr 一个不是 int 的 value 会返回错误，incr 一个不存在的 key，则设置 key 为 1

incrby

同 incr 类似，加指定值 ，key 不存在时候会设置 key，并认为原来的 value 是 0

Decr

对 key 的值做的是减减操作，decr 一个不存在 key，则设置 key 为-1

Decrby

同 decr，减指定值。

Append

给指定 key 的字符串值追加 value,返回新字符串值的长度。

Strlen

取指定 key 的 value 值的长度。

persist xxx(取消过期时间)

选择数据库（0-15库）

Select 0

选择数据库

move age 1

把age 移动到1库

Randomkey

随机返回一个key

Rename

重命名

Type

返回数据类型



## 使用过Redis分布式锁么，它是怎么实现的？

先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。

**如果在setnx之后执行expire之前进程意外crash或者要重启维护了，那会怎么样？**

set指令有非常复杂的参数，这个应该是可以同时把setnx和expire合成一条指令来用的！



## 使用过Redis做异步队列么，你是怎么用的？有什么缺点？

一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。

缺点：

在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。

**能不能生产一次消费多次呢？**

使用pub/sub主题订阅者模式，可以实现1:N的消息队列。



## 什么是缓存穿透？如何避免？什么是缓存雪崩？何如避免？

**缓存穿透**

一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。

如何避免？

1：对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清理缓存。

2：对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该bitmap过滤。

**缓存雪崩**

当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。

如何避免？

1：在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

2：做二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期

3：不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。



## Redis的用途是什么？

**计数器** 可以对 String 进行自增自减运算，从而实现计数器功能。Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

**缓存**将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

**会话缓存** 可以使用 Redis 来统一存储多台应用服务器的会话信息。当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

**全页缓存（FPC）** 除基本的会话 token 之外，Redis 还提供很简便的 FPC 平台。以 Magento 为例，Magento 提供一个插件来使用 Redis 作为全页缓存后端。此外，对 WordPress 的用户来说，Pantheon 有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

**查找表** 例如 DNS 记录就很适合使用 Redis 进行存储。查找表和缓存类似，也是利用了Redis快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

**消息队列(发布/订阅功能)** List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。不过最好使用 Kafka、RabbitMQ 等消息中间件。

**分布式锁实现** 在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

**其它** Set 可以实现交集、并集等操作，从而实现共同好友等功能。ZSet可以实现有序性操作，从而实现排行榜等功能。



## 如何与Redis连接？

安装服务器后，可以运行 redis 安装时提供的 Redis 客户端，也可以打开命令提示符并使用以下命令：

 redis-cli

通过使用其中任何一个，您可以与 Redis 交互。



## Redis的主要特点是什么？

以下是 Redis 的主要功能：

- 读写性能优异， Redis 能读的速度是 110000 次/s，写的速度是 81000 次/s。
- 支持数据持久化，支持 AOF 和 RDB 两种持久化方式。
- 支持事务，Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作合并后的原子性执行。
- 数据结构丰富，除了支持 string 类型的 value 外还支持 hash、set、zset、list 等数据结构。
- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。



## 解释Redis的复制功能？

Redis 可以使用主从同步，从从同步。第一次同步时，主节点做一次 bgsave，并同时将后续修改操作记录到内存 buffer，待完成后将 rdb 文件全量同步到复制节点，复制节点接受完成后将 rdb 镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。



## Redis和RDBMS有什么区别？

Redis 和 RDBMS 之间存在很多差异：

- Redis 是 NoSQL 数据库，而 RDBMS 是 SQL 数据库。
- Redis 遵循键值结构，而 RDBMS 遵循表结构。
- Redis 非常快，而 RDBMS 相对较慢。
- Redis 将所有数据集存储在主存储器中，而 RDBMS 将其数据集存储在辅助存储器中。
- Redis 通常用于存储小型和常用文件，而 RDBMS 用于存储大文件。
- Redis 仅为 Linux，BSD，Mac OS X，Solaris 提供官方支持。它目前没有为 Windows 提供官方支持，而 RDBMS 提供对两者的支持。



## 为什么Redis不同于其他的键值存储数据库？

有两个主要原因：

- Redis发展方向不同与其他键值数据库，**它能包含很多复杂数据类型，对这些数据类型操作都是原子的**。Redis数据类型与基本数据结构强相关，直接暴露给程序员，没有增加抽象层。
- Redis是一个基于内存的，能够持久化到硬盘的数据库，**因此为了实现高速读写，数据集大小不能超过内存**。内存数据库另一个优点是，内存数据库相对于硬盘数据库非常容易操作复杂数据结构，因此Redis的可以做很多事情，内部复杂性低。与此同时两款磁盘存储格式（RDB和AOF）不需要支持随机访问，因此他们是紧凑的，而且总是以追加形式生成（甚至AOF日志轮换也是一个追加操作，因为新版本是由内存中的副本生成）。比起基于磁盘的数据存储， Redis 用来处理重要数据时需要确保数据集及时落盘刷新。



## Redis内存使用情况？

举几个例子（所有数据基于64位实例）

- 一个空实例大约占用3M内存
- 1百万简单字符串键值对大约占用85M内存
- 1百万哈希表键值对，每个对象有5个属性，大约占用160M内存

为了测试你的用例，使用redis-benchmark工具生成随机数据集，使用INFO memory命令检查使用内存空间。

存储相同的键，64位系统比32位系统使用更多的内存，键值很小情况下更明显。这是因为64位系统指针占用8字节。但是64位系统优点是可以配置更多内存（校对注：32位操作系统支持的内存最多为2的32次方，就是4G），因此为了运行大型Redis服务器，64位系统尤佳。另一种方案是使用分片。



## 我喜欢Redis的高性能操作和特性，但是不喜欢所有内容都在内存中，我不能创建一个比内存更大数据集。有计划改变吗？

过去为了允许数据集超过RAM大小，Redis开发人员尝试使用虚拟内存和其他系统，但是我们非常高兴可以把一件事情做好：数据服务由内存提供，磁盘用于存储数据。所以现在没有计划为Redis创建磁盘后端，毕竟Redis大部分特性都是基于其当前架构设计的。

你的真正问题并不是所需的总内存，而是你需要划分你的数据集到多个Redis实例上，为了获取更多信息请阅读本文档中的分区页面。



## 同时使用Redis和磁盘数据库，是不是一个好想法？

是的，一个通用的设计方案是，在非常频繁的写小的数据时采用Redis（并且你需要使用Redis数据结构给你的问题建立高效模型），以及将大数据存储到SQL数据库或者最终一致性磁盘数据库中。



## 有没有方法降低Redis内存使用率？

如果可以的话使用Redis 32位实例。另外，还要善于使用哈希表，列表，有序集合和整数集，因为在特殊情况下Redis使用这些数据类型可以更紧凑存储一些元素。可以在内存优化页面获取更多信息。



## Redis内存不足时会发生什么？

Redis要么被Linux内核OOM杀掉，抛出错误崩溃，要么开始变得卡顿。随着现代操作系统malloc方法通常都不返回NULL，而是服务器开始交换，因此Redis性能降低，因此你可能会观察到一些错误现象。

INFO命令返回Redis使用内存总量，因此你可以编写脚本监控Redis服务器内存临界值。

Redis内置保护措施允许用户在配置文件中使用`maxmemory`选项，设置Redis最大占用内存。如果达到此限制，Redis将开始返回错误给写命令（但是将继续接受只读命令），或者当最大内存限制达到时也可以配置为键淘汰，在这种情况下Redis作为缓存使用。

我们有文档描述Redis作为LRU缓存使用。



## 在Linux系统中，即使我有很多空闲内存，后台保存失败报fork()错误！

精简回答：`echo 1 > /proc/sys/vm/overcommit_memory`

详细回答：

Redis后台保存模式依赖现代操作系统的写时拷贝技术。Redis fork（创建一个子进程）是父进程精确拷贝。子进程存储数据到磁盘并且最终退出。从理论上讲，子进程应该和父进程使用同样多内存，作为父进程副本，但是得益于多数现代操作系统实现的写时复制技术，父进程和子进程共享内存页。内存页在父进程或子进程改变时将被复制。当子进程保存时，理论上所有页面都可能改变，Linux无法提前告知子进程需要多少内存，因此如果overcommit_memory设置为0，fork将会失败除非有足够的空闲RAM真正复制父进程内存页.结果是，如果你有3G Redis数据集，只有2G可用内存将会失败。

overcommit_memory设置为1，意味着Linux 使用更乐观方式fork，这确实是你所期望的Redis。

“理解虚拟机内存 ”是红帽经典文章，可以了解Linux虚拟内存怎么工作，overcommitmemory和overcommitratio的替代品。这篇文章校正了proc(5)用户手册对overcommit_memory1和2配置正确含义。



## Redis磁盘快照是不是原子操作？

是的，当服务器不在执行命令时，Redis后台保存进程总是被创建，因此每个命令在RAM中是原子的，并且在磁盘快照过程也是原子的。



## Redis是单线程的，我怎么利用多CPU/核?

CPU基本不可能成为的Redis的瓶颈，因为通常Redis受限于内存或网络。例如使用Pipelining，Redis运行在普通的Linux系统上，每秒可以处理50万请求，所以如果你的应用程序主要使用O(N) 或者 O(log(N))命令，几乎不会使用太多的CPU。

然而为了最大限度利用CPU，你可以在一台机器上启动多个Redis实例，并把它们设置为不同服务器。某些时候单个机器是不够的，所以如果你想使用多个CPU，你可以提前考虑使用分片。

关于使用多Redis实例，你可以在Partitioning page找到更多的信息



## 单个Redis实例最多可以存储多少键？哈希表、列表、集合和有序集合最大可以包含多少元素？

Redis最大可以处理232键，实践测试每个实例最少可以处理2.5亿键。

每个哈希表、列表、集合和有序集合可以容纳232元素。

换句话说，Redis极限容量就是系统可用内存。



## 为什么从实例与主实例拥有不同数量键？

如果你使用有生存周期的键，这就是正常现象。这就导致主从实例键的数量不一致原因。

- 主实例在第一次与从实例同步时生成RDB文件。
- RDB文件不包含已经过期的键，但是已经过期的键仍然在内存中。
- 尽管这些键从逻辑上说已经过期失效，但是还在Redis主实例内存中，他们并不被识别为存在的，当增量或访问这些键时这些键会被回收。尽管从逻辑上说这些键不是数据集一部分，但是INFO和DBSIZE命令结果包含这些信息。
- 当从实例读取主实例生成的RDB文件时，过期键不会被载入。

为很多键设置过期属性，通常为用户提供了在从实例上存储更少键，但是实际上实例内容没有逻辑区别。



## *Redis实际含义是什么？

Redis是远程数据字典服务器（REmote DIctionary Server）。



## 为什么启动Redis项目？

最初启动Redis，是为了扩大LLOOGG。但是当我完成了基本服务端工作后，我喜欢把这个想法分享给其他人，然后Redis转变成开源项目。



## Redis 如何发音？

Redis 读作颜色的"red"，然后是 "iss"。
