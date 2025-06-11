---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 安全
order: 3
---

# redis 设置密码

对于数据库来说，安全性是非常必要的，以确保数据的安全性。它提供身份验证，因此如果客户端想要建立连接，则需要在执行命令之前进行身份验证。

您需要在配置文件中设置密码以保护 Redis 数据库。



### 例

我们来看看如何保护您的 Redis 实例。

使用`config get command`

```
config get requirepass
```

![Redis安全1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-security1-1.png)

您可以看到上面的属性为空，表示我们没有此实例的任何密码。您可以通过执行以下命令来更改此属性并为此实例设置密码。

```
config set requirepass "rediscomcn123"
```

![Redis安全2](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-security2-1.png)

```
CONFIG get requirepass
```

![Redis安全3](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-security3-1.png)

当您设置此密码时，如果客户端在未经身份验证的情况下运行该命令，则会收到错误"NOAUTH Authentication required"。因此，客户端需要使用 AUTH 命令来验证自己。

------



## AUTH命令的用法

```
127.0.0.1:6379> AUTH "rediscomcn123"  
OK  
127.0.0.1:6379> SET mykey "hindi100" 
OK  
127.0.0.1:6379> GET mykey  
"hindi100"  
127.0.0.1:6379>  
```

![Redis安全4](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-security4-1.png)



## Redis 设置密码

如何设置Redis密码并使用密码连接到Redis服务器？

Redis服务器默认不受密码保护，也就是没有设置密码。

有两种方法可以设置redis服务器的密码。

- 使用命令行
- 直接修改redis.conf文件



## Redis设置或更改密码

首先，检查是否使用`auth-Password`命令设置了密码

```
127.0.0.1:6379> auth password
(error) ERR Client sent AUTH, but no password is set
```

返回错误表示没有为Redis服务器设置密码。

- 使用命令行设置密码

使用交互式cli，您可以使用`CONFIG SET` 更改密码

```
CONFIG SET requirepass "12345"
```

requirepass 是用于更改密码的配置参数，设置密码实时生效。 重启失效，用永久生效需要保存设置到配置文件。

要保存这些更改,运行以下命令

```
CONFIG REWRITE
```

重新启动Redis后生效

- 使用redis.conf文件

Redis.conf文件包含与安全相关的带注释配置，如下所示

```
# requirepass foobared
```

需求注释,设置密码未1234567

```
requirepass 123467
```

保存文件。

重新启动或停止并启动服务器以重新加载更改。

现在，Redis服务器是受密码保护的。

任何想要与服务器通信的客户端都需要提供`-a 密码`选项。

```
redis-cli -h 127.0.0.1 -p 6379 -a 1234567
```
