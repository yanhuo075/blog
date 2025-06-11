---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 连接
order: 14
---

# Redis连接

Redis 连接命令用于控制和管理到 Redis Server 的客户端连接。



### 例

以下示例说明客户端如何向 Redis 服务器验证自身并检查服务器是否正在运行。

```
redis 127.0.0.1:6379> AUTH "password"  
(error) ERR Client sent AUTH, but no password is set  
redis 127.0.0.1:6379>  
redis 127.0.0.1:6379> PING  
PONG  
redis 127.0.0.1:6379>  
```

![Redis Connections](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-connections1-1.png)



#### *注意：在这里您可以看到未设置“密码”，因此您可以直接访问任何命令。

------



## Redis 连接命令

下表列出了用于 Redis 连接相关的命令

| 命令          | 描述               |
| :------------ | :----------------- |
| AUTH password | 验证密码是否正确   |
| ECHO message  | 打印字符串         |
| PING          | 查看服务是否运行   |
| QUIT          | 关闭当前连接       |
| SELECT index  | 切换到指定的数据库 |
