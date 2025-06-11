---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 客户端连接
order: 5
---

# Redis客户端连接

Redis 可以在配置的监听 TCP 端口和 Unix 套接字上接受不同类型的客户端连接。

接受新客户端连接时，它将执行以下操作：

- 由于 Redis 使用多路复用和非阻塞 I/O，因此客户端套接字处于非阻塞状态。
- 设置 TCP_NODELAY 选项是为了确保我们的连接没有延迟。
- 创建可读文件事件，以便一旦可以在套接字上读取新数据，Redis 就能够收集客户端查询。

------



## 最大客户端数

在 Redis config（redis.conf）中，有一个名为 `maxclients` 的属性，它指定可以连接到 Redis 的客户端数量。

以下是命令的基本语法。

```
Config get maxclients  
"maxclients"  
"4064"  
```

![Redis客户端连接1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-client-connection1-1.png)

最大客户端数取决于 OS 的最大文件描述符数限制。它的默认值为 10000，但您可以更改此属性。



### 例

我们举一个例子，在启动服务器时将最大客户端数设置为 100000。

```
redis-server --maxclients 100000  
```

------



## Redis 客户端命令

| 命令           | 描述                                       |
| :------------- | :----------------------------------------- |
| CLIENT LIST    | 返回连接到 redis 服务的客户端列表          |
| CLIENT SETNAME | 设置当前连接的名称                         |
| CLIENT GETNAME | 获取通过 CLIENT SETNAME 命令设置的服务名称 |
| CLIENT PAUSE   | 挂起客户端连接，指定挂起的时间以毫秒计     |
| CLIENT KILL    | 关闭客户端连接                             |
