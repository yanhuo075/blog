---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 发布订阅
order: 11
---

# Redis 发布订阅

Redis 发布/订阅是一种消息传模式，其中发送者（在Redis术语中称为发布者）发送消息，而接收者（订阅者）接收消息。传递消息的通道称为channel。

在Redis中，客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/pubsub2.png)



## 实例

以下实例演示了发布订阅是如何工作的，需要开启两个 redis-cli 客户端。

在我们实例中我们创建了订阅频道名为 rediscomcnChat:



## 第一个 redis-cli 客户端

```
redis 127.0.0.1:6379> SUBSCRIBE rediscomcnChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

现在，我们先重新开启个 redis 客户端，然后在同一个频道 rediscomcnChat 发布两次消息，订阅者就能接收到消息。



## 第二个 redis-cli 客户端

```
redis 127.0.0.1:6379> PUBLISH rediscomcnChat "Redis PUBLISH test"
(integer) 1

redis 127.0.0.1:6379> PUBLISH rediscomcnChat "Learn redis by redis.com.cn"
(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "rediscomcnChat"
3) "Redis PUBLISH test"
1) "message"
2) "rediscomcnChat"
3) "Learn redis by redis.com.cn"
```



## Redis 发布订阅命令

下表列出了列表相关命令：

| 命令         | 描述                               |
| :----------- | :--------------------------------- |
| PSUBSCRIBE   | 订阅一个或多个符合给定模式的频道。 |
| PUBSUB       | 查看订阅与发布系统状态。           |
| PUBLISH      | 将信息发送到指定的频道。           |
| PUNSUBSCRIBE | 退订所有给定模式的频道。           |
| SUBSCRIBE    | 订阅给定的一个或多个频道的信息。   |
| UNSUBSCRIBE  | 指退订给定的频道。                 |
