---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 8. Docker run 运行容器
order: 7
---

# 8. docker run 运行容器

Docker run 作为运行容器的直接入口，命令参数相当丰富，使用它可以启动容器，使容器拥有自己的文件系统、网络以及关系进程树。

Docker run 命令基本结构：

```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

## docker run 命令示例

以下是一个比较常见的 Docker run 命令示例，用于创建一个 NGINX 容器：

```
docker run -d \
  --name my_nginx \
  --restart=always \
  -p 80:80 \
  -v /path/to/nginx/conf:/etc/nginx/conf.d \
  -v /path/to/nginx/html:/usr/share/nginx/html \
  nginx:latest
```

- `-d`: 在后台以守护进程模式运行容器。

- `--name my_nginx`: 为容器指定一个名称（可以根据需要更改为您喜欢的名称）。

- `--restart=always`：指定容器在退出时的重启策略。

- `-p 80:80`: 将主机的端口 80 映射到容器的端口 `80`。这样，您可以通过访问 `http://localhost` 来访问 NGINX 容器中的网站。

- `-v /path/to/nginx/conf:/etc/nginx/conf.d`: 将主机上的 NGINX 配置文件目录挂载到容器中的 `/etc/nginx/conf.d` 目录，以便使用自定义的 NGINX 配置。

- `-v /path/to/nginx/html:/usr/share/nginx/html`: 将主机上的 HTML 文件目录挂载到容器中的 `/usr/share/nginx/html` 目录，以便在容器中提供自定义的静态网页内容。

- `nginx:latest`: 指定要使用的 NGINX 镜像及其标签（可以根据实际情况替换为您自己的镜像名称和标签）。

  

`Docker run` 命令是在 `Docker` 中创建和运行容器的主要命令之一。它允许您根据需要配置容器的各种属性。以下是 `Docker run` 命令的30个常用参数的详细解释和示例用法，帮助您更好地理解和使用这些参数。

```
`-d` 或 `--detach`：以后台模式运行容器，将容器放置在后台运行，作为守护进程。
示例：`docker run -d image_name`

`-it`：以交互模式运行容器，允许与容器进行交互。
示例：docker run -it image_name
   
--name`：为容器指定一个名称。
示例：docker run --name container_name image_name
 
`-p`：将容器的端口映射到主机上的一个端口。
示例：docker run -p host_port:container_port image_name

`-v`：挂载主机上的文件或目录到容器内部。
示例：docker run -v host_path:container_path image_name

`-e`：设置容器的环境变量。
示例：docker run -e ENV_VARIABLE=value image_name

`--restart`：指定容器在退出时的重启策略。
示例：docker run --restart=always image_name
   
`--link`：将容器连接到另一个容器，在两个容器之间建立网络连接。
示例：docker run --link container_name:image_alias image_name

`--dns`：指定容器使用的自定义 DNS 服务器。
示例：docker run --dns 8.8.8.8 image_name

`--dns-search`：指定容器的 DNS 搜索域。
示例：docker run --dns-search example.com image_name

`--cap-add` 和 `--cap-drop`：增加或删除容器的 `Linux` 能力，用于控制容器的权限。
示例：docker run --cap-add=SYS_ADMIN image_name
 
`--privileged`：给容器赋予特权，可以访问主机的设备。
示例：docker run --privileged image_name
 
`--tmpfs`：在容器内创建临时文件系统，用于存储临时数据。
示例：docker run --tmpfs /tmp image_name
 
`--ulimit`：设置容器的资源限制，如最大打开文件数、最大进程数等。
示例：docker run --ulimit nofile=1024:1024 image_name
 
`--security-opt`：设置容器的安全选项，如 `AppArmor` 配置、`Seccomp` 配置等。
示例：docker run --security-opt seccomp:unconfined image_name
 
`--cpu-shares`：设置容器的 CPU 份额，用于控制 CPU 资源的分配。
示例：docker run --cpu-shares 512 image_name

`--memory`：设置容器可使用的内存限制。
示例：docker run --memory 1g image_name

`--network`：指定容器使用的网络模式。
示例：`docker run --network bridge image_name`

`--hostname`：设置容器的主机名。
示例：docker run --hostname my_container image_name

`--user`：指定容器运行时的用户名或 UID。
示例：docker run --user username image_name

`--volume-driver`：指定容器使用的卷驱动程序。
示例：docker run --volume-driver my_driver image_name

`--shm-size`：设置容器的共享内存大小。
示例：docker run --shm-size 2g image_name

`--add-host`：向容器的 `/etc/hosts` 文件添加自定义主机名和 IP 映射。
示例：docker run --add-host myhost:192.168.0.100 image_name

`--read-only`：将容器的文件系统设置为只读模式。
示例：docker run --read-only image_name

`–cpu-quota`：设置容器的 CPU 配额，以微秒为单位。
示例：docker run --cpu-quota=50000 image_name

`--cpu-period`：设置容器的 CPU 周期，以微秒为单位。
示例：docker run --cpu-period=100000 image_name

`--dns-option`：为容器的 DNS 配置添加自定义选项。
示例：docker run --dns-option=timeout:5 image_name

`--sysctl`：设置容器的内核参数。
示例：docker run --sysctl net.ipv4.ip_forward=1 image_name

`--label`：为容器添加标签，用于识别和组织容器。
示例：docker run --label env=production image_name

`--workdir`：设置容器的工作目录。
示例：docker run --workdir /app image_name
```

为了更好理解，我们将参数分为以下几类：

1. 容器管理：
   - 后台程序和前台交互程序
   - 器的定义
2. 网络设置
3. CPU和内存的runtime
4. 权限和LXC配置



## 8.1 容器管理

### 8.1.1 守护态运行 Detached

当我们启动一个container时，首先需要确定这个container是运行在前台模式还是运行在后台模式。

如果在docker run 后面追加-d=true或者-d，则containter将会运行在后台模式(Detached mode)。此时所有I/O数据只能通过网络资源或者共享卷组来进行交互。因为container不再监听你执行docker run的这个终端命令行窗口。正如之前的例子：

```
$ sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
61f37c1940c8ec9f08b107e99655b8a5181ded340415e3c15cf413069d556b73
$...
```

但你可以通过执行docker attach 来重新挂载这个container里面。

```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
0409679f511a        ubuntu              "/bin/sh -c 'while t   5 seconds ago       Up 3 seconds                            thirsty_perlman
$ sudo docker attach 0409679f511a
hello world
hello world
hello world
…
```

*需要注意的是，如果你选择执行-d使container进入后台模式，那么将无法配合"--rm"参数。*

### 8.1.2 带控制窗口运行

与Detached（－d）对应的是Foregroud

如果在docker run后面没有追加-d参数，则container将默认进入前台模式(Foregroud mode)。Docker会启动这个container，同时将当前的命令行窗口挂载到container的标准输入，标准输出和标准错误中。也就是container中所有的输出，你都可以再当前窗口中查看到。甚至docker可以虚拟出一个TTY窗口，来执行信号中断。这一切都是可以配置的：

```
-a=[]               : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t=false            : Allocate a pseudo-tty
--sig-proxy=true    : Proxify all received signal to the process (non-TTY mode only)
-i=false            : Keep STDIN open even if not attached
```

如果在执行run命令时没有指定-a，那么docker默认会挂载所有标准数据流，包括输入输出和错误。你可以特别指定挂载哪个标准流。

```
$ sudo docker run -a stdin -a stdout -i -t ubuntu /bin/bash (只挂载标准输入输出)
```

对于执行容器内的交互式操作，例如shell脚本。我们必须使用 -i -t来申请一个控制台同容器进行数据交互。但是当通过管道与容器进行交互时，就不能使用-t. 例如下面的命令：

```
$ echo test | docker run -i busybox cat
```

若强行加上-t 就会报出cannot enable tty mode on non tty input错误。

### 8.1.3 给容器命名

给container 命名有三种方式：

1. 使用UUID长命 ("f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778")，在创建容器时返回的id就是这个。

2. 使用UUID短命令("f78375b1c487")，当执行查询时，查到的dockerID就是这个。

3. 使用--name=evil_ptolemy",若不加此指令，docker会自动给新创建出来的容器分配一个唯一的name

   ```
    $ sudo docker run -d --name=test_name registry.liugang/centos:latest
    f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778
    $ sudo docker ps -a
    CONTAINER ID        IMAGE                            COMMAND       CREATED             STATUS                      PORTS       NAMES
    f78375b1c487        registry.liugang/centos:latest   "/bin/bash"   17 seconds ago      Exited (0) 17 seconds ago               test_name
   ```

### 8.1.4 清除容器

Clean up (--rm) 指在容器运行完之后自动清除

```
--rm=false: Automatically remove the container when it exits (incompatible with -d)
```

默认情况下，每个container在退出时，它的文件系统也会保存下来。这样一方面调试会方便些，因为你可以通过查看日志等方式来确定最终状态。另外一方面，你也可以保存container所产生的数据。但是当你仅仅需要短期的运行一个前台container，这些数据同时不需要保留时。你可能就希望docker能在container结束时自动清理其所产生的数据。 这个时候你就需要--rm这个参数了。

```
$ sudo docker run --rm centos:latest
$ docker ps -a
CONTAINER ID       IMAGE       COMMAND    CREATED     STATUS       PORTS       NAMES
(无)
```

*注意：--rm 和 -d不能共用！*



## 8.2 数据管理

### 8.2.1 数据卷

参数的作用就是挂载一个文件目录到指定容器中去，实现容器中数据持久化。

- 数据卷是一个可以供一个或多个使用的特殊目录，它绕过UFS，可以提供很多有用的特性
  - 数据卷可以在容器之间共享和重用
  - 对数据卷的修改会立马生效
  - 对数据卷的更新，不会影响镜像
  - 卷会一直存在，直到没有容器使用

### 8.2.2 挂载目录

在使用docker run时，加上-v参数可以创建一个数据卷挂载到目标容器中去，也可以多次使用该参数挂载多个数据卷。下面创建一个容器，挂载一个数据卷。

```
$ sudo docker run --name=test -it -v /home/test_volume/:/home/test centos
[root@6a7818f6290b /]# cd /home/
[root@6a7818f6290b home]# ls
test
```

发现容器中已经成功挂载数据卷，但是如果你对系统是CentoOS7系统，你会发现，无法访问test，说明权限不够，是因为CentOS7中的安全模块selinux把权限禁掉了，所以要在运行的时候加上特权：

```
$ sudo docker run --name=test -it --privileged=true -v /home/test_volume/:/home/test centos
```

这样我们就有了对容器的读写权利。（解决方法还有好多种，在后面问题总结中有所介绍。）

Docker 挂载数据卷的默认权限是读写，用户也可以通过 :ro 指定为只读。

```
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro
training/webapp python app.py
```

*注意：也可以在 Dockerfile 中使用 VOLUME 来添加一个或者多个新的卷到由该镜像创建的任意容器。这将在仓库服务中详细讲解。*

### 8.2.3 挂载文件

-v 标记也可以从主机挂载单个文件到容器中

```
$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```

这样就可以记录在容器输入过的命令了。

*注意：如果直接挂载一个文件，很多文件编辑工具，包括 vi 或者 sed --in-place，可能会造成文件 inode 的改变，从 Docker 1.1 .0起，这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。*



## 8.3 资源配置

### 8.3.1 内存资源设置

设置内存我们可以有四种方式：

- **memory=inf, memory-swap=inf (default)**

  *默认的方式设置最低值，容器可以使用大于此最低值的内存数*

- **memory=L<inf, memory-swap=inf**

  *设置memory不能使用超过L的值。*

- **memory=L<inf, memory-swap=2\*L**

- **memory=L<inf, memory-swap=S<inf, L<=S**

  *memory不能超过L，swap＋memory总使用量不能超过S*

例子： $ sudo docker run -ti centos /bin/bash

默认不设置任何限制。(第一种情况)

```
$ sudo docker run -ti -m 300M --memory-swap -1 centos  /bin/bash
```

memory最多使用300M，swap没有限制

```
$ docker run -ti -m 300M centos  /bin/bash
```

我们只设置了memory限制时300M，swap没有指定，默认被设置为与memory一样的值。memory＋swap一共是600M

```
$ docker run -ti -m 300M --memory-swap 1G centos  /bin/bash
```

这里我们同时设置了memory和swap ，对应第四种情况

如果发生内存溢出错误，内核讲kill掉容器中的进程。如果你想控制，可以配合使用- -oom-kill-disable参数。如果没有制定-m参数，可能导致当内存溢出时内核会杀死主机进程。 例子： 设置容器内存限制100M，并且阻止 OOM killer

```
$ docker run -ti -m 100M --oom-kill-disable centos /bin/bash
```

如果不使用-m参数制定限制，官方说很危险！

### 8.3.2 CPU资源设置

默认情况下，所有容器获得CPU周期的比例相同。可以通过改变容器的CPU加权占有率相对于其他正在运行容器的加权占有率的比例来调整。

修改1024的比例，使用-c或--cpu-sharesflag的权重设置为2或更高。 该比例只适用在CPU密集型进程运行时。当在一个容器中的任务处于空闲状态，其他容器可以使用剩余空闲CPU时间。实际CPU时间将根据在系统上运行的容器的数目而变化。

例如，考虑三个容器的情况，一个拥有cpu的1024和另外两个有512 CPU共享时间，三个容器进程都尝试使用100％的CPU，第一个容器将获得的50％总的CPU时间。如果您添加CPU值为1024的第四个容器中，第一个容器只得到了CPU的33％。剩余的容器将分别占用CPU的16.5％，16.5％和33％。

在多核心系统中，CPU时间的份额分布在所有CPU核心。即使容器被限制为CPU时间小于100％时，它可以使用每个单独的CPU核心的100％。例如，在一个拥有超过三个核心的系统中，

如果启动一个容器设置-c＝512跑一个进程，另外一个设置-c=1024,跑2个进程，内存分配将会如下配置：

```
PID    container    CPU CPU share
100    {C0}     0   100% of CPU0
101    {C1}     1   100% of CPU1
102    {C1}     2   100% of CPU2
```

**--cpu-period参数**

默认设置为100ms，当然我们也可以自己设置cpu周期，限制容器CPU用量。通常该参数伴随--cpu-quota参数使用。

**--cpu-quota参数**

限制CPU用量。默认值0，意味着允许容器获得1个CPU的100%的资源量。设置50000限制CPU资源的50%。

```
$ sudo docker run -ti --cpu-period=50000 --cpu-quota=25000 centos /bin/bash
```

如果是单核心系统，将意味着容器将每50ms获得50%运行周期。

**--cpuset参数**

*设置容器允许运行的cpu号（在多核心系统中）：*

设置容器在CPU1和CPU3上运行

```
$ sudo docker run -ti --cpuset-cpus="1,3" centos /bin/bash
```

设置容器在CPU0、CPU1、CPU2上运行

```
$ sudo docker run -ti --cpuset-cpus="0-2" centos /bin/bash
```

*设置容器在指定mems上执行（只在NUMA系统中有效）：*

容器只能在memory nodes 1和2上运行

```
$ sudo docker run -ti --cpuset-mems="1,3" centos /bin/bash
```

**--bkio-weight参数**

默认情况下，所有容器获得相同比例的blokIO带宽，这个比例值是500。要修改此比例，使用--blkio-weight设置容器的blkio相对于其他运行容器权重。它的取值范围是10～1000。 下面的例子中，设置了两个不同blkio:

```
$ sudo docker run -ti --name c1 --blkio-weight 300 centos /bin/bash
$ sudo docker run -ti --name c2 --blkio-weight 600 centos /bin/bash
```

如果同时设置两个容器blockIO，利用如下指令：

```
$ time dd if=/mnt/zerofile of=test.out bs=1M count=1024 oflag=direct
```

You’ll find that the proportion of time is the same as the proportion of blkio weights of the two containers.

```
Note: The blkio weight setting is only available for direct IO. Buffered IO is not currently supported.
```



## 8.4 访问互联

### 8.4.1 外部访问容器

有时候，容器要运行一些网络应用，需要外部能访问到这些应用，就需要使用-p/P 参数指定一个主机端口，映射到容器端口中。其中使用P系统会分配一个随机的端口到内部容器开放的网络端口。

就拿仓库服务镜像来做例子：

```
$ sudo docker run -d -P registry
b89fc89e061dee24ac532af1890cd26e6e016545e0978b01d3d4eadca67119aa
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
b89fc89e061d        registry:latest     "docker-registry"   5 seconds ago       Up 4 seconds        0.0.0.0:32768->5000/tcp   focused_brown
$ curl 192.168.4.100:32768/v1/search
{"num_results": 0, "query": "", "results": []}[root@registry liugang]#
$ sudo docker logs b89fc89e061d
[2015-08-18 00:11:41 +0000] [1] [INFO] Starting gunicorn 19.1.1
[2015-08-18 00:11:41 +0000] [1] [INFO] Listening at: http://0.0.0.0:5000 (1)
```

我们可以看到，当我们加上-P时，docker会任意指定一个一个端口指定到容器的开放端口5000上。从容器到运行日志也是可以看出，在容器的5000端口会有一个监听。当我们通过外网的，也就是宿主机的IP 和端口就可以访问到该容器内提供的服务，这里是仓库服务。

-p（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有

```
ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort。
```

### 映射所有接口地址

我们用到的是 hostPort:containerPort，也就是将制定端口映射到主机地址的任意地址的指定端口：

```
$ sudo docker run -d -p 80:5000 registry
a7abe89606427e3cb90698a6d302e8
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
a7abe8960642        registry:latest     "docker-registry"   4 seconds ago       Up 3 seconds        0.0.0.0:80->5000/tcp   ecstatic_wright
2abb0f04066999adff36b13be2e380c3de
```

我们将主机80端口映射到容器，这样我们直接用主机地址就可以访问到容器了。

### 映射到指定地址的指定端口

可以使用 ip:hostPort:containerPort 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```
$ sudo docker run -d -p 127.0.0.1:5000:5000 registry
9f11390c1e9d048f7d82ff6bfb7e65f5531865343a5a2e6b660c0634e90eda26
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                      NAMES
9f11390c1e9d        registry:latest     "docker-registry"   6 seconds ago       Up 5 seconds        127.0.0.1:5000->5000/tcp   romantic_carson
```

### 映射到指定地址的任意端口

使用 ip::containerPort 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。

```
$ sudo sudo docker run -d -p 127.0.0.1::5000 registry
d8cd77ecc45f434ab9edc0a7e83514ef7cb019fabc9bdbc0b522bb916b309789
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                       NAMES
d8cd77ecc45f        registry:latest     "docker-registry"   4 seconds ago       Up 3 seconds        127.0.0.1:32768->5000/tcp   grave_carson
```

从上面看出，此时docker默认的传输协议是tcp方式，我们也可以改为其他方式标记：

```
$ sudo docker run --name=test_port -d -p 127.0.0.1:5000:5000/udp registry
```

### 使用 docker port 查看端口信息

```
$ docker port test_port
5000/udp -> 127.0.0.1:5000
```

注意：

*容器有自己的内部网络和 ip 地址（使用 docker inspect 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）*

*-p 标记可以多次使用来绑定多个端口*

### 8.4.2 容器间通信

容器在使用Docker的时候我们会常常碰到这么一种应用，就是我需要两个或多个容器，其中某些容器需要使用另外一些容器提供的服务。所以，我们要考虑的问题时如何建立两个容器间通信。

**容器的连接（linking）**

系统是除了端口映射外，另一种跟容器中应用交互的方式。 该系统会在源和接收容器之间创建一个隧道，接收容器可以看到源容器指定的信息。

首先我们先创建一个容器(这里我只是用作示范,没有使用官方示例的镜像，所谓但数据容器内并没有提供数据服务，官方例子我举出来也没啥意思)

创建数据访问容器db：

```
$ sudo docker run -idt --name=db centos
600886c7c69dc4979bdfee19d82331879d71835f794db110eb3b5ea3c164bd30
```

使用--link=name:alias name就是要访问的目标机器，alias就是自定的别名

```
$ sudo docker run -it --name=web --link=db:test_link centos
[root@8d92293a65e9 /]# cat /etc/hosts
172.17.0.12    8d92293a65e9
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.11    test_link 600886c7c69d db
```

我们看到容器内的hosts 内多了一条信息

```
172.17.0.11    test_link 600886c7c69d db
```

这就意味着我们可以访问到db容器进行通信。
