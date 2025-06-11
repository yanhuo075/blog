---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 安装
order: 1
---

# Linux 安装



## 源码编译安装Redis

本教程使用稳定版本，下载并安装Redis：

```
wget http://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make
```

执行完 make 命令后，src 目录下会出现编译后的 redis 服务程序 `redis-server`，还有用于测试的客户端程序 `redis-cli`，两个程序位于安装目录 src 目录下：

- **redis-server** Redis 服务器程序
- **redis-cli** 与Redis交互的命令行界面程序

安装可执行程序到/usr/local/bin，执行:

---
make install
---



## 在前台启动和停止 Redis

安装后，您可以通过运行来启动 Redis

```
redis-server
```

如果成功，您将看到 Redis 的启动日志，Redis 将在前台运行。

```
30858:C 11 May 2023 20:26:52.870 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
30858:C 11 May 2023 20:26:52.870 # Redis version=7.0.11, bits=64, commit=00000000, modified=0, pid=30858, just started
30858:C 11 May 2023 20:26:52.870 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
30858:M 11 May 2023 20:26:52.871 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 7.0.11 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 30858
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

30858:M 11 May 2023 20:26:52.871 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
30858:M 11 May 2023 20:26:52.871 # Server initialized
30858:M 11 May 2023 20:26:52.871 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
30858:M 11 May 2023 20:26:52.872 * Ready to accept connections
```

要停止 Redis，请输入**Ctrl-C**。

注意这种方式启动 redis 使用的是默认配置。也可以通过启动参数告诉 redis 使用指定配置文件使用下面命令启动。

```
cp redis.conf /etc/redis.conf
redis-server /etc/redis.conf
```

redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。

启动 redis 服务进程后，就可以使用测试客户端程序 redis-cli 和 redis 服务交互了。 比如：

```
redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

关闭停止redis服务

```
redis-cli shutdown
#直接关闭不保存内存
redis-cli shutdown nosave
```



## 配置 Redis 为后台服务

将配置文件中的 daemonize no 改成 daemonize yes，配置 redis 为后台启动。



## Redis 设置访问密码

在配置文件中找到 requirepass，去掉前面的注释，并修改后面的密码。

常用配置文件例子 redis.conf

```
#默认端口6379
port 6379
#绑定ip，如果是内网可以直接绑定 127.0.0.1, 或者忽略, 0.0.0.0是外网
bind 0.0.0.0
#守护进程启动
daemonize yes
#超时
timeout 300
loglevel notice
#分区
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
#存储文件
dbfilename dump.rdb
#密码 abcd123
requirepass abcd123
```



# 在 ubuntu 上安装 Redis

按照下面给出的步骤在 Ubuntu 上安装 Redis：

首先使用 sudo 设置非 root 用户，然后安装构建和测试依赖项：

```
sudo apt update 
sudo apt full-upgrade
sudo apt install build-essential tcl
```

![在Ubuntu               1上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu1-2.png)![在Ubuntu               2上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu2-2.png)

要继续按 Y 键

![在Ubuntu               3上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu3-2.png)



## 安装 Redis 服务器

使用以下命令安装 Redis 服务器：

sudo apt-get install redis-server

![在Ubuntu               4上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu4-2.png)![在Ubuntu               5上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu5-2.png)

现在安装了 Redis Server。您可以启动 Redis 服务器：

------



## 启动 Redis 服务器

您使用以下命令启动 redis 服务器：

```
 redis-server
```

![在Ubuntu               6上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu6-2.png)



## 启动 Redis 客户端

Redis 服务器已启动，因此您可以启动 redis 客户端以在它们之间进行通信。

```
 redis-cli
```

![在Ubuntu               7上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu7-2.png)

------



## 验证 Redis 是否正常工作

执行以下命令：

```
redis-cli
```

这将打开一个 redis 提示符。

**redis 127.0.0.1:6379>**

在上面的提示中，127.0.0.1 是机器的 IP 地址，6379 是 Redis 服务器运行的端口。

现在键入以下 PING 命令。返回 PONG 表示 Redis 已成功安装在您的系统上。

![在Ubuntu               8上安装Redis](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/installation-on-ubuntu8-2.png)



# 在 windows 上安装 Redis

Redis 官方不建议在 windows 下使用 Redis，所以官网没有 windows 版本可以下载。还好微软团队维护了开源的 windows 版本，虽然只有 3.2 版本，对于普通测试使用足够了。



## 安装包方式安装 Redis 服务

下载地址：https://github.com/MicrosoftArchive/redis/releases

点击下载：

![Redis安装4](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation4-1-1.png) ![Redis安装5](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation5-1-1.png)

您可以看到 Redis 现在已下载。

或者您也可以使用下面链接下载。

https://github.com/rgl/redis/downloads

下载完成之后双击按着引导流程安装。

![Redis安装6](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation6-1-1.png) ![Redis安装7](https://www.redis.com.cn/wp-content/uploads/2020/03/redis-installation7-1-1.png) ![Redis安装8](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation8-1-1.png) ![Redis安装9](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation9-1-1.png) ![Redis安装10](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation10-1-1.png) ![Redis安装11](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation11-1-1.png)

------



## 启动 Redis

Redis 现在可以使用了。打开 Redis 程序目录：

![Redis安装12](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation12-1-1.png)

文件介绍：

redis-server.exe：服务端程序，提供 redis 服务

redis-cli.exe: 客户端程序，通过它连接 redis 服务并进行操作

redis-check-dump.exe：RDB 文件修复工具

redis-check-aof.exe：AOF 文件修复工具

redis-benchmark.exe：性能测试工具，用以模拟同时由 N 个客户端发送 M 个 SETs/GETs 查询 (类似于 Apache 的 ab 工具)

redis.windows.conf： 配置文件，将 redis 作为普通软件使用的配置，命令行关闭则 redis 关闭

redis.windows-service.conf：配置文件，将 redis 作为系统服务的配置

单击 redis-server.exe，启动 Redis 服务。

![Redis安装13](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation13-1-1.png)

现在启动 Redis 客户端。

![Redis安装14](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation14-1-1.png) ![Redis安装15](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation15-1-1.png)

检查 Redis 是否已连接。

使用 PING 命令。

![Redis安装16](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation16-1-1.png)

Redis 服务窗口也输出 1 个客户端已连接。

![Redis安装17](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-installation17-1-1.png)



## 直接解压的方式安装 redis

首先下载 redis 安装包：https://github.com/MSOpenTech/redis/releases

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20180530232823964-e1584240924769.png)

解压安装包到相应文件夹，任何盘符都行，例如 E:\tools\redis-3.2.100。



### 使用命令行启动 Redis 服务

运行 cmd，cd 进入对应目录 E:\tools\redis-3.2.100，执行：

```
redis-server.exe redis.windows.conf 
```

*注：可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的参数。

输入之后会显示如下：

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/5d84a6dd5f1b242dce37ab0423388981.png)



### 安装 redis 到 windows 服务

```
redis-server --service-install redis.windows.conf
```

查看 windows 服务是否加入：

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/2a38fe5c35c522f602e4647ef8af3d58.png)

这时候先关闭打开的第一个 cmd 窗口，然后执行以下命令启动再次 redis：

```
redis-server --service-start
```

停止 redis 服务：

```
redis-server --service-stop
```

最后，测试一下 redis 是否能够正常使用：

切换到 redis 目录下：E:\tools\redis-3.2.100 下：

```
redis-cli.exe -h 127.0.0.1 -p 6379 
```
