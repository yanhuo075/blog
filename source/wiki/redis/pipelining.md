---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 流水线
order: 6
---

# Redis Pipelining 流水线

在了解流水线之前，首先要了解 Redis 的概念：

Redis 是一个支持请求/响应协议的 TCP 服务器。在 Redis 中，请求分两步完成：

- 客户端通常以阻塞方式向服务器发送命令。
- 服务器处理该命令并将响应发送回客户端。

------



## 什么是流水线

流水线操作有助于客户端向服务器发送多个请求，而无需等待回复，最后只需一步即可读取回复。

**例**

让我们看一下 Redis 流水线的例子。在这个例子中，我们将向 Redis 提交一次命令，Redis 将在一个步骤中提供所有命令的输出。

打开 Redis 终端并使用以下命令：

```
（echo -en  "PING \r\n SET sssit javatraining \r\n GET sssit \r\n INCR visitor \r\n INCR visitor \r\n INCR visitor \r\n" ; sleep  10 ）|  
 nc localhost  6379  
```

![Redis流水线1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-pipelining1-1.png)

这里：

- PING 命令用于检查 Redis 连接。
- 设置名为“sssit”的字符串，其值为“javatraining”。
- 获得了 key 值并将访问者数量增加了三倍。

每次增加值时都可以看到。

------



## 流水线的优势

Redis 流水线操作的主要优点是提高了 Redis 的性能。由于多个命令同时执行，它极大地提高了协议性能。



## Pipelining vs Scripting

Redis Scripting 可在 Redis 2.6 或更高版本中使用。

脚本的主要优点是它可以以最小的延迟同时读取和写入数据。它使读取，计算，写入等操作变得非常快。

在流水线操作中，客户端在调用 write 命令之前需要 read 命令的回复。
