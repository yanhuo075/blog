---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Java 使用 Redis
order: 8
---

# Java 使用 Redis

开始在 Java 中使用 Redis 前， 我们需要确保已经安装了 redis 服务及 Java redis 驱动，且你的机器上能正常使用 Java。



## 安装

首先，安装 Redis 的 java 驱动。

- 首先你需要下载驱动包 [**下载 jedis.jar**](https://mvnrepository.com/artifact/redis.clients/jedis)，确保下载最新驱动包。
- 在你的 classpath 中包含该驱动包。



## 连接到 Redis 服务

```
import redis.clients.jedis.Jedis; 

public class RedisJava { 
   public static void main(String[] args) { 
      //Connecting to Redis server on localhost 
      Jedis jedis = new Jedis("localhost"); 
      System.out.println("Connection to server sucessfully"); 
      //check whether server is running or not 
      System.out.println("Server is running: "+jedis.ping()); 
   } 
} 
```

编译并运行程序。可以根据需要修改你的路径。我们假设 **jedis.jar** 在当前路径中。

```
$javac RedisJava.java 
$java RedisJava 
Connection to server sucessfully 
Server is running: PONG
```



## Redis Java String 实例

```
import redis.clients.jedis.Jedis; 

public class RedisStringJava { 
   public static void main(String[] args) { 
      //Connecting to Redis server on localhost 
      Jedis jedis = new Jedis("localhost"); 
      System.out.println("Connection to server sucessfully"); 
      //set the data in redis string 
      jedis.set("tutorial-name", "Redis tutorial"); 
      // Get the stored data and print it 
      System.out.println("Stored string in redis:: "+ jedis.get("tutorial-name")); 
   } 
}
```

编译和运行上面的程序

```
$javac RedisStringJava.java 
$java RedisStringJava 
Connection to server sucessfully 
Stored string in redis:: Redis tutorial 
```



## Redis Java List 实例

```
import redis.clients.jedis.Jedis; 

public class RedisListJava { 
   public static void main(String[] args) { 

      //Connecting to Redis server on localhost 
      Jedis jedis = new Jedis("localhost"); 
      System.out.println("Connection to server sucessfully"); 

      //store data in redis list 
      jedis.lpush("tutorial-list", "Redis"); 
      jedis.lpush("tutorial-list", "Mongodb"); 
      jedis.lpush("tutorial-list", "Mysql"); 
      // Get the stored data and print it 
      List<String> list = jedis.lrange("tutorial-list", 0 ,5); 

      for(int i = 0; i<list.size(); i++) { 
         System.out.println("Stored string in redis:: "+list.get(i)); 
      } 
   } 
} 
```

编译和运行程序

```
$javac RedisListJava.java 
$java RedisListJava 
Connection to server sucessfully 
Stored string in redis:: Redis 
Stored string in redis:: Mongodb 
Stored string in redis:: Mysql
```



## Redis Java Keys 实例

```
import redis.clients.jedis.Jedis; 

public class RedisKeyJava { 
   public static void main(String[] args) { 

      //Connecting to Redis server on localhost 
      Jedis jedis = new Jedis("localhost"); 
      System.out.println("Connection to server sucessfully"); 
      //store data in redis list 
      // Get the stored data and print it 
      List<String> list = jedis.keys("*"); 

      for(int i = 0; i<list.size(); i++) { 
         System.out.println("List of stored keys:: "+list.get(i)); 
      } 
   } 
}
```

编译和运行程序

```
$javac RedisKeyJava.java 
$java RedisKeyJava 
Connection to server sucessfully 
List of stored keys:: tutorial-name 
List of stored keys:: tutorial-list 
```
