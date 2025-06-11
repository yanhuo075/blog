---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: PHP 使用 Redis
order: 9
---

# php 操作 Redis

开始在PHP 中连接 Redis 前， 需要确保你的机器上已经安装了 redis PHP 驱动并配置了 PHP 环境。

接下来让我们安装 PHP redis驱动：下载地址为:[**https://github.com/phpredis/phpredis/releases**](https://github.com/phpredis/phpredis/releases)。



## 安装

PHP 如何安装配置 Redis PHP 驱动。

PHP redis 驱动下载地址为:[**https://github.com/phpredis/phpredis/releases**](https://github.com/phpredis/phpredis/releases)。下载完成后，解压文件到 phpredis 目录。

```
cd phpredis 
sudo phpize 
sudo ./configure 
sudo make 
sudo make install 
```

拷贝“modules” 目录中的内容到 PHP extension 目录，并修改 **php.ini** 增加

```
extension = redis.so
```

Redis PHP 驱动安装完成。



## 连接到 Redis 服务

```
<?php 
   //Connecting to Redis server on localhost 
   $redis = new Redis(); 
   $redis->connect('127.0.0.1', 6379); 
   echo "Connection to server sucessfully"; 
   //check whether server is running or not 
   echo "Server is running: ".$redis->ping(); 
?>
```

执行，输出如下：

```
Connection to server sucessfully 
Server is running: PONG 
```



## Redis PHP String 实例

```
<?php 
   //Connecting to Redis server on localhost 
   $redis = new Redis(); 
   $redis->connect('127.0.0.1', 6379); 
   echo "Connection to server sucessfully"; 
   //set the data in redis string 
   $redis->set("tutorial-name", "Redis tutorial"); 
   // Get the stored data and print it 
   echo "Stored string in redis:: " .$redis->get("tutorial-name"); 
?>
```

执行，输出如下：

```
Connection to server sucessfully 
Stored string in redis:: Redis tutorial 
```



## Redis php List 实例

```
<?php 
   //Connecting to Redis server on localhost 
   $redis = new Redis(); 
   $redis->connect('127.0.0.1', 6379); 
   echo "Connection to server sucessfully"; 
   //store data in redis list 
   $redis->lpush("tutorial-list", "Redis"); 
   $redis->lpush("tutorial-list", "Mongodb"); 
   $redis->lpush("tutorial-list", "Mysql");  

   // Get the stored data and print it 
   $arList = $redis->lrange("tutorial-list", 0 ,5); 
   echo "Stored string in redis:: "; 
   print_r($arList); 
?>
```

执行，输出如下：

```
Connection to server sucessfully 
Stored string in redis:: 
Redis 
Mongodb 
Mysql 
```



## Redis PHP Keys 实例

```
<?php 
   //Connecting to Redis server on localhost 
   $redis = new Redis(); 
   $redis->connect('127.0.0.1', 6379); 
   echo "Connection to server sucessfully"; 
   // Get the stored keys and print it 
   $arList = $redis->keys("*"); 
   echo "Stored keys in redis:: " 
   print_r($arList); 
?>
```

执行，输出如下：

```
Connection to server sucessfully 
Stored string in redis:: 
tutorial-name 
tutorial-list 
```
