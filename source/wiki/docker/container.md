---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 5. Docker 容器使用
order: 4
---

## 5. Docker 容器

容器是 Docker 又一核心概念。

简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。本章节着重介绍了容器的基础使用方法，以及如何管理容器的数据、如何管理容器的网络，相信你读完本章节将会有一个大致的了解。



### 5.1 容器使用入门

使用info命令检查docker安装程序是否正常运行：

```
# 检查是否安装好docker
$ docker info
```

如果提示如下信息: command not found 或者 类似于/var/lib/docker/repositories: permission denied 可能安装有问题或者尝试在前面加上sudo。

此外，基于docker系统配置，命令行前面应该加上sudo，来确保正常执行命令，并且此时系统会为Administrar创建一个名叫docker的Unix 用户组来让其他用户加入该组。

**5.1.1 下载一个已经制作好的镜像**

```
$ docker pull ubuntu
```

这条命令会从Docker Hub上搜索ubuntu镜像，然后下载到本里镜像缓存中去。

***Note:When the image is successfully downloaded, you see a 12 character hash 539c0211cd76: Download complete which is the short form of the image ID. These short image IDs are the first 12 characters of the full image ID - which can be found using docker inspect or docker images --no-trunc=true.\***

**5.1.2 创建一个带有交互窗口的container**

```
$ docker run -i -t ubuntu /bin/bash
```

—i参数表示启动了一个可以交互的容器，—t参数表示创建了一个附带标准输入和输出的pseudo－TTY窗口

如果想要退出tty窗口，使用 Ctrl-p + Ctrl-q指令，容器将会退出并且会持续保持一个停止的状态。如果想查看所有状态的容器，可以使用docker ps －a 指令。

**5.1.3 绑定容器到另外一个主机/端口或者一个Unix Socket**

***Warning: Changing the default docker daemon binding to a TCP port or Unix docker user group will increase your security risks by allowing non-root users to gain root access on the host. Make sure you control access to docker. If you are binding to a TCP port, anyone with access to that port has full Docker access; so it is not advisable on an open network.\***

使用 -H 能够让Docker deamon监听特殊的IP和端口。默认情况下，它将监听unix:///var/run/docker.sock以仅允许root进行本地连接。你可以将它设置为0.0.0.0:2375或者是指定的主机IP以供所有人连接，但是并不建议这么做，因为这将使某些无聊的人也获得deamon运行主机root的访问权限。

类似的，Docker客户端也可以使用 —H 连接一个指定端口

-H 使用以下格式来分配主机和端口 :

tcp://host[path] or unix://path

例如:

- tcp://host:2375 -> TCP connection on host:2375
- tcp://host:2375/path -> TCP connection on host:2375 and prepend path to all requests
- unix://path/to/socket -> Unix socket located at path/to/socket

当—H参数为空时，将被默认认为没有—H参数传进来

-H also accepts short form for TCP bindings:

host[:port] or :port

Docker以daemon模式运行:

```
$ sudo <path to>/docker daemon -H 0.0.0.0:5555 &
```

下载一个ubuntu镜像:

```
$ docker -H :5555 pull ubuntu
```

如果你想同时监听TCP和Unix Socket 你可以添加多个 －H

```
# Run docker in daemon mode
$ sudo <path to>/docker daemon -H tcp://127.0.0.1:2375 -H unix:///var/run/docker.sock &
# Download an ubuntu image, use default Unix socket
$ docker pull ubuntu
# OR use the TCP port
$ docker -H tcp://127.0.0.1:2375 pull ubuntu
```

**5.1.4 起一个持续工作的进程**

```
# Start a very useful long-running process
$ JOB=$(docker run -d ubuntu /bin/sh -c "while true; do echo Hello world; sleep 1; done")

# Collect the output of the job so far
$ docker logs $JOB

# Kill the job
$ docker kill $JOB
```

**5.1.5 监听容器**

```
$ docker ps # Lists only running containers
$ docker ps -a # Lists all containers
```

**5.1.6 管理容器**

```
# Start a new container
$ JOB=$(docker run -d ubuntu /bin/sh -c "while true; do echo Hello world; sleep 1; done")

# Stop the container
$ docker stop $JOB

# Start the container
$ docker start $JOB

# Restart the container
$ docker restart $JOB

# SIGKILL a container
$ docker kill $JOB

# Remove a container
$ docker stop $JOB # Container must be stopped to remove it
$ docker rm $JOB
```

**5.1.7 在一个TCP端口上绑定一个服务**

```
# Bind port 4444 of this container, and tell netcat to listen on it
$ JOB=$(docker run -d -p 4444 ubuntu:12.10 /bin/nc -l 4444)

# Which public port is NATed to my container?
$ PORT=$(docker port $JOB 4444 | awk -F: '{ print $2 }')

# Connect to the public port
$ echo hello world | nc 127.0.0.1 $PORT

# Verify that the network connection worked
$ echo "Daemon received: $(docker logs $JOB)"
```

**5.1.8 提交（保存）一个容器的状态到一个镜像文件中**

当提交（commit）容器时，docker仅保存源镜像与当前镜像的差异，如果想列出镜像，请使用docker images指令

```
# Commit your container to a new named image
$ docker commit <container> <some_name>

# List your images
$ docker images
```



### 5.2 管理容器工作

**5.2.1 概览**

我们用docker run指令来运行一个容器:

- 交互容器跑在前端.
- 守护进程跑在后台.

一些常用管理容器的命令:

- docker ps - 列出容器.
- docker logs - 输出容器日志.
- docker stop - 停止运行容器.

Docker客户端非常简单，你只需要需要输入一些带有一系列参数的指令就可以：

```
# Usage:  [sudo] docker [command] [flags] [arguments] ..
# Example:
$ docker run -i -t ubuntu /bin/bash
```

我们以docker version为例查看当前安装的docker客户端以及deamon进程信息

```
$ docker version
```

这条指令不仅提供啦docker客户端以及deamon进程的版本信息，同时也提供了所用go语言的信息。

```
Client:
  Version:      1.8.1
  API version:  1.20
  Go version:   go1.4.2
  Git commit:   d12ea79
  Built:        Thu Aug 13 02:35:49 UTC 2015
  OS/Arch:      linux/amd64

Server:
  Version:      1.8.1
  API version:  1.20
  Go version:   go1.4.2
  Git commit:   d12ea79
  Built:        Thu Aug 13 02:35:49 UTC 2015
  OS/Arch:      linux/amd64
```

**5.2.2 获取docker命令行帮助**

如果你想展示一些帮助信息，可以使用：

```
$ docker --help
```

如果你想了解具体参数的使用方法，可以参照如下使用：

```
$ docker attach --help

Usage: docker attach [OPTIONS] CONTAINER

Attach to a running container

--help=false        Print usage
--no-stdin=false    Do not attach stdin
--sig-proxy=true    Proxy all received signals to the process
```

**5.2.3 在docker中跑一个web应用程序**

现在你已经掌握了一些基础了，来深入一下吧。之前跑的一些应用没什么实际用途，让们来跑一个web应用试试吧。

这个web应用包含里 python 应用：

```
$ docker run -d -P training/webapp python app.py
```

以上指令中包含两个参数：

```
－d 让容器在后台运行
－P 分配一个主机端口到容器端口的映射以供外部访问
```

training/webapp是一个构建好的镜像，里面包含了一个简单的Python Flask web应用程序。

**5.2.4 查看web应用程序状态**

我们利用docker ps 来查看容器状态

```
$ docker ps -l
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
```

其中指令包含了一个－l参数，这将展示最近一次创建的容器的状态。

```
Note: 默认的 docker ps 不加参数将只会展现正在运行对容器，使用docker ps －a 将展现所有状态的容器
```

在POTS栏中，我们可以查看到端口映射。

```
PORTS
0.0.0.0:49155->5000/tcp
```

当我们使用－P参数时，docker将会给主机指定容器暴露的所有端口。

在这个例子中，容器暴露里5000端口，主机随机分配了一个49155端口与之映射。

网络端口绑定在Docker是可配置性非常高的。—P参数是随机指定一个32768～61000的主机端口与容器绑定，而－p则是指定一个端口与容器绑定：

```
$ docker run -d -p 80:5000 training/webapp python app.py
```

这条指令将绑定主机的80端口与容器的5000端口。

我们来看看49155端口上的web应用吧！

![image](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/webapp.jpg)

**5.2.5 快捷查看网络端口**

用docker ps命令可以查看端口映射，此外，docker海提供了另一种便捷的方式，用docker port命令

```
$ docker port nostalgic_morse 5000
0.0.0.0:49155
```

**5.2.6 查看web应用日志**

使用docker logs 可以查案容器内部的日志记录这样可以让我们清楚地看到容器运行的状态和历史纪录：

```
$ docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -
```

这里加了一个－f参数，可以查看容器标准输出。这里展现了该web应用在5000端口上的日志信息。

**5.2.7 查看web应用容器中的进程**

根据容器的日志信息，我们可以利用 docker top 指令查看容器内的进程信息

```
$ docker top nostalgic_morse
PID                 USER                COMMAND
854                 root                python app.py
```

这里我们在容器里看到了刚才我们运行的 python app.py命令。

**5.2.8 审查（inspect）web应用容器**

利用docker inspect 查看一个容器的详细信息（包括运行状态）或者镜像的配置信息，展现出来的是一个json文件。

```
$ docker inspect nostalgic_morse
Let’s see a sample of that JSON output.

[{
    "ID": "bc533791f3f500b280a9626688bc79e342e3ea0d528efe3a86a51ecb28ea20",
    "Created": "2014-05-26T05:52:40.808952951Z",
    "Path": "python",
    "Args": [
       "app.py"
    ],
    "Config": {
       "Hostname": "bc533791f3f5",
       "Domainname": "",
       "User": "",
    . . .
```

我们可以使用－f参数里筛选出需要的信息

```
$ docker inspect -f '{{ .NetworkSettings.IPAddress }}' nostalgic_morse
172.17.0.5
```

**5.2.9 停止web应用容器**

现在我们尝试停止我们刚才启用的容器，容器名称: nostalgic_morse.

```
$ docker stop nostalgic_morse
nostalgic_morse
```

我们可以使用docker ps 来查看是否成功

```
$ docker ps -l
```

**5.2.10 重启web应用容器**

重启一下试试：

```
$ docker start nostalgic_morse
nostalgic_morse
```

用docker ps －l命令或者打开浏览器产看变化

**5.2.11 删除web应用容器**

我们可以利用docker rm来删除一个容器，但有时候会出现以下错误：

```
$ docker rm nostalgic_morse
Error: Impossible to remove a running container, please stop it first or use -f
2014/05/24 08:12:56 Error: failed to remove one or more containers
```

怎么回事? 原来删除的是一个正在运行的容器. 所以我们要做的是stop it&remove it

```
$ docker stop nostalgic_morse
nostalgic_morse
$ docker rm nostalgic_morse
nostalgic_morse
```

官方建议

```
Note: Always remember that removing a container is final!
```



### 5.3 管理容器数据

到目前为止，我们已经介绍了一些基本的docker概念，如何管理docker 镜像，以及了解网络和容器之间的联系。

在这一节中，我们将介绍如何管理容器数据。

docker管理数据的两种主要方式。

数据卷，以及数据卷容器。

数据卷是在一个或多个容器，它绕过Union File System的一个专门指定的目录。数据卷为持续共享数据提供了一些有用的功能：

- 在创建容器时，卷被初始化。如果容器的基础映像包含指定的数据装入点，现有的数据复制到在卷初始化新卷。
- 数据卷可以共享和容器之间重复使用。
- 改变数据卷将立刻生效（在所有挂载该容器中）
- 改变数据卷数据不会影响到容器。
- 即使容器本身被删除。但是数据卷依然存在。
- 数据卷的目的是持久化数据，独立于容器的生命周期。Docker因此不会自动删除卷，当你删除一个容器，也不会“垃圾回收”直到没有容器再使用。

**5.3.1 添加一个数据卷**

你可以在docker run时加上－v参数来添加一个数据卷，－v参数也可以使用多次，以挂载多个数据卷。

```
$ docker run -d -P --name web -v /webapp training/webapp python app.py
```

这条命令将在容器中的／webapp文件夹创建一个数据卷存储数据。

你也可以在构建镜像时在Dockerfile里面定义。

数据卷默认的权限时读写，你也可以定义成只读。

```
$ docker run -d -P --name web -v /opt/webapp:ro training/webapp python app.py
```

查找数据卷路径用docker inspect命令定位

```
$ docker inspect web
```

我们可以看到保存在主机上数据卷的路径：

```
...
Mounts": [
    {
        "Name": "fac362...80535",
        "Source": "/var/lib/docker/volumes/fac362...80535/_data",
        "Destination": "/webapp",
        "Driver": "local",
        "Mode": "",
        "RW": true
    }
]
```

... 并切看到权限RW是ture

**5.3.2 挂载一个主机目录作为数据卷**

挂载主机目录为数据卷，必须参照 －v hostPATH:containerPATH 这种格式 路径必须为绝对路径，以保证容器的 可移植性。

```
$ docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

上面的命令加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录

docker数据卷的权限是读写，你也可以指定只读：

```
$ docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py
```

**5.3.3 挂载一个数据文件作为数据卷**

-v 标记也可以从主机挂载单个文件到容器中

```
$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```

这样就可以记录在容器输入过的命令了。

如果直接挂载一个文件，很多文件编辑工具，包括 vi 或者 sed --in-place，可能会造成文件 inode 的改变，从 Docker 1.1 .0起，这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。

**5.3.4 创建和挂载数据卷容器**

如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。

数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。

首先，创建一个命名的数据卷容器 dbdata：

```
$ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres
```

然后，在其他容器中使用 --volumes-from 来挂载 dbdata 容器中的数据卷。

```
$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres
$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres
```

还可以使用多个 --volumes-from 参数来从多个容器挂载多个数据卷。 也可以从其他已经挂载了数据卷的容器来挂载数据卷。

```
$ sudo docker run -d --name db3 --volumes-from db1 training/postgres
```

*注意：使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态。

如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 docker rm -v 命令来指定同时删除关联的容器。 这可以让用户在容器之间升级和移动数据卷。

**5.3.5 备份、存储、移动数据卷**

另一个非常有用大功能是利用数据卷容器进行备份、存储以及迁移操作。

备份

```
$ docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

然后新创建一个新的容器

```
$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

然后解压数据卷挂载到容器

```
$ docker run --volumes-from dbdata2 -v $(pwd):/backup ubuntu cd /dbdata && tar xvf /backup/backup.tar
```



### 5.4 容器间的通信

容器的通信相当重要，这里讲解了一通信方式。

**5.4.1 容器与宿主机器采用端口映射的方式通信**

之前的例子

```
$ docker run -d -P training/webapp python app.py
```

我们可以看到端口映射状态：

```
$ docker ps nostalgic_morse
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
```

我们也可以指定端口映射：

```
$ docker run -d -p 80:5000 training/webapp python app.py
```

－p参数还有其他指定方法：

```
ip:hostPort:containerPort  映射指定IP的指定端口

ip::containerPort          映射指定IP任意端口

hostPort:containerPort。    映射所有主机IP的指定端口
```

**5.4.2 采用link方式通信**

名字的重要性 为了方便建立连接，通常需要为容器起一个name ，如果不起，你会发现系统会自动分配一个名字。

此外，起一个自定义的名字让你很容易地记住它并切可以方便的管理

```
$ docker run -d -P --name web training/webapp python app.py
```

查看容器

```
$ docker ps -l
CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web
```

你也可以使用docker inspect 命令查看容器名称。

```
Note: Container names have to be unique. That means you can only call one container web. If you want to re-use a container name you must delete the old container (with docker rm) before you can create a new container with the same name. As an alternative you can use the --rm flag with the docker run command. This will delete the container immediately after it is stopped.
```

Links 允许容器发现另一个容器，并在期间建立一个安全的通道以便交换数据。 使用－－link参数来创建一个连接。

```
$ docker run -d --name db training/postgres
```

创建一个web连接到db数据库容器.

```
$ docker run -d -P --name web --link db:db training/webapp python app.py
```

参数格式如下:

```
--link <name or id>:alias
```

alias代表你 为这个链接起的一个别名。

```
--link <name or id>
```

该参数将匹配容器name 并建立连接

```
$ docker run -d -P --name web --link db training/webapp python app.py
```

使用docker inspect 定位:

```
$ docker inspect -f "{{ .HostConfig.Links }}" web
[/db:/web/db]
```

你可以看到web容器已经与db容器连接

连接容器docker究竟做了什么？目前已经知道，一个创建在源容器与目标容器间的连接允许源容器提供信息给目标容器。在上述例子中，目标容器web可以获取源容器db提供的信息。为了实现这项功能，docker在这两个容器之间创建了一个稳定的通道，并且不会暴露任何端口，你不需要在创建容器时加上－p或－P参数。这就是link方式的最大好处。

docker通过以下两种方式完成此项工作

##### 环境变量

当link容器时，docker会创建许多环境变量. Docker会自动创建环境变量到目标容器中去. 它也将通过docker暴露源容器的所有环境变量. 这些变量来自:

Dockerfile中ENV命令定义的环境变量

在启动源容器中使用 －e 、--env以及 --env－file参数附加的环境变量。

这些环境变量使程序从相关的目标容器中发现源容器。

```
Warning: It is important to understand that all environment variables originating from Docker within a container are made available to any container that links to it. This could have serious security implications if sensitive data is stored in them.
```

Docker为每一个在--link参数中的容器设置了一个 _NAME 环境变量 。例如，一个web容器通过--link db：webdb连 接db容器，将会在web容器中创建一个WEBDB_NAME=/web/webdb环境变量

Docker为源容器暴露的端口限定了一组环境变量，每一个环境变量具有唯一前缀形式：

```
<name>_PORT_<port>_<protocol>
```

前缀的构成:

- 是--link :后面的参数(例如, webdb)
- 就是暴露的端口号
- TCP／UDP

Docker 利用这前缀格式定义了三个不同的环境变量:

prefix_ADDR 变量包含了来自URL的IP地址, for example

```
WEBDB_PORT_5432_TCP_ADDR=172.17.0.82.
```

prefix_PORT 变量仅包含了URL的端口号, for example

```
WEBDB_PORT_5432_TCP_PORT=5432.
```

prefix_PROTO 参数包含URL的传输协议, for example

```
WEBDB_PORT_5432_TCP_PROTO=tcp.
```

如果容器暴露多个端口，Docker将会为每个端口创建三个环境变量。算术题：如果容器暴露4个端口，将会创建多少个环境变量？答对了，是12个哦！每个端口三个环境变量。

另外，Docker也要创建一个叫 _PORT 的环境变量。这个变量包含源容器URL首次暴露的IP和端口。该端口的“首次”定义为最低级数字的端口。例如，思考WEBDB_PORT=tcp://172.17.0.82:5432 变量，如果该端口同时用语TCP和UDP，则TCP将会被指定。（原文：*Additionally, Docker creates an environment variable called _PORT. This variable contains the URL of the source container’s first exposed port. The ‘first’ port is defined as the exposed port with the lowest number. For example, consider the WEBDB_PORT=tcp://172.17.0.82:5432 variable. If that port is used for both tcp and udp, then the tcp one is specified.*）

最后，Docker会把源容器中的环境变量暴露给目标容器作为环境变量。并且Docker会在目标容器为每个变量创建一个*ENV*变量。这个变量的值被设置为启动源容器Docker所用到的值。（原文：*Finally, Docker also exposes each Docker originated environment variable from the source container as an environment variable in the target. For each variable Docker creates an \*ENV\* variable in the target container. The variable’s value is set to the value Docker used when it started the source container.*）

回到之前的例子 database ,你可以使用env命令列出具体的容器环境变量：

```
$ docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5432_TCP=tcp://172.17.0.5:5432
DB_PORT_5432_TCP_PROTO=tcp
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_ADDR=172.17.0.5
. . .
```

你可以看到Docker利用许多有关源容器的信息创建了一些列的环境变量。每一环境变量都会带有指点定义的别名，DB*前缀。如果别名是db1，那么变量前缀也会变成DB1*。利用这线环境变量来配置应用用来在db容器上连接数据库。这样的连接方式稳定安全私有化。只有已获得连接的web容器才会有对db容器的访问权限。

关于环境变量的一些注意事项：

与修改/etc/hosts文件不同，在环境变量中存储的IP地址信息不回随着容器的重启而更新，建议利用hosts文件来解决连接容器的IP地址问题。

这些环境变量只是为容器的第一个process设置，某些deamon后台服务，例如sshd，只有当产生连接需求时才会设置。（原文：These environment variables are only set for the first process in the container. Some daemons, such as sshd, will scrub them when spawning shells for connection.）

##### 更新 /etc/hosts 文件

处理环境变量, docker在源文件中追加了host信息，这里向web容器追加:

```
$ docker run -t -i --rm --link db:webdb training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  webdb 6e5cdeb2d300 db
```

我们可以在容器中使用ping命令测试链接：

```
root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping
root@aed84ee21bde:/opt/webapp# ping webdb
PING webdb (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms
```

如果你重启源容器，连接依然存在：

```
$ docker restart db
db
$ docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.9  db
```
