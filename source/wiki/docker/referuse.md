---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 7. Docker 参数用法详解
order: 6
---

# 7. Docker 参数用法详解


## 7.1 deamon 参数

Docker对使用者来讲是一个`C/S`模式的架构，而Docker的后端是一个非常松耦合的架构，模块各司其职，并有机组合，支撑Docker的运行,如下图所示：

![image](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/docker_struct.jpg)

不难看出，用户是使用Docker Client与Docker Daemon建立通信，并发送请求给后者。 而Docker Daemon作为Docker架构中的主体部分，首先提供Server的功能使其可以接受Docker Client的请求；而后Engine执行Docker内部的一系列工作，每一项工作都是以一个Job的形式的存在。

Job的运行过程中，当需要容器镜像时，则从Docker Registry中下载镜像，并通过镜像管理驱动graphdriver将下载镜像以Graph的形式存储；当需要为Docker创建网络环境时，通过网络管理驱动networkdriver创建并配置Docker容器网络环境；当需要限制Docker容器运行资源或执行用户指令等操作时，则通过execdriver来完成。

而libcontainer是一项独立的容器管理包，networkdriver以及execdriver都是通过libcontainer来实现具体对容器进行的操作。

当执行完运行容器的命令后，一个实际的Docker容器就处于运行状态，该容器拥有独立的文件系统，独立并且安全的运行环境等。

#### Docker Client

Docker Client是Docker架构中用户用来和Docker Daemon建立通信的客户端。用户使用的可执行文件为docker，通过docker命令行工具可以发起众多管理container的请求。

Docker Client可以通过以下三种方式和Docker Daemon建立通信：tcp://host:port，unix://path_to_socket和fd://socketfd。为了简单起见，本文一律使用第一种方式作为讲述两者通信的原型。与此同时，与Docker Daemon建立连接并传输请求的时候，Docker Client可以通过设置命令行flag参数的形式设置安全传输层协议(TLS)的有关参数，保证传输的安全性。

Docker Client发送容器管理请求后，由Docker Daemon接受并处理请求，当Docker Client接收到返回的请求相应并简单处理后，Docker Client一次完整的生命周期就结束了。当需要继续发送容器管理请求时，用户必须再次通过docker可执行文件创建Docker Client。

#### Docker Daemon

Docker Daemon是Docker架构中一个常驻在后台的系统进程，功能是：接受并处理Docker Client发送的请求。该守护进程在后台启动了一个Server，Server负责接受Docker Client发送的请求；接受请求后，Server通过路由与分发调度，找到相应的Handler来执行请求。

Docker Daemon启动所使用的可执行文件也为docker，与Docker Client启动所使用的可执行文件docker相同。在docker命令执行时，通过传入的参数来判别Docker Daemon与Docker Client。

deamon的参数选项：

```
Usage: docker daemon [OPTIONS]
A self-sufficient runtime for linux containers.
Options:
  --api-cors-header=""                   Set CORS headers in the remote API
  -b, --bridge=""                        Attach containers to a network bridge
  --bip=""                               Specify network bridge IP
  -D, --debug=false                      Enable debug mode
  --default-gateway=""                   Container default gateway IPv4 address
  --default-gateway-v6=""                Container default gateway IPv6 address
  --dns=[]                               DNS server to use
  --dns-search=[]                        DNS search domains to use
  --default-ulimit=[]                    Set default ulimit settings for containers
  -e, --exec-driver="native"             Exec driver to use
  --exec-opt=[]                          Set exec driver options
  --exec-root="/var/run/docker"          Root of the Docker execdriver
  --fixed-cidr=""                        IPv4 subnet for fixed IPs
  --fixed-cidr-v6=""                     IPv6 subnet for fixed IPs
  -G, --group="docker"                   Group for the unix socket
  -g, --graph="/var/lib/docker"          Root of the Docker runtime
  -H, --host=[]                          Daemon socket(s) to connect to
  -h, --help=false                       Print usage
  --icc=true                             Enable inter-container communication
  --insecure-registry=[]                 Enable insecure registry communication
  --ip=0.0.0.0                           Default IP when binding container ports
  --ip-forward=true                      Enable net.ipv4.ip_forward
  --ip-masq=true                         Enable IP masquerading
  --iptables=true                        Enable addition of iptables rules
  --ipv6=false                           Enable IPv6 networking
  -l, --log-level="info"                 Set the logging level
  --label=[]                             Set key=value labels to the daemon
  --log-driver="json-file"               Default driver for container logs
  --log-opt=[]                           Log driver specific options
  --mtu=0                                Set the containers network MTU
  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file
  --registry-mirror=[]                   Preferred Docker registry mirror
  -s, --storage-driver=""                Storage driver to use
  --selinux-enabled=false                Enable selinux support
  --storage-opt=[]                       Set storage driver options
  --tls=false                            Use TLS; implied by --tlsverify
  --tlscacert="~/.docker/ca.pem"         Trust certs signed only by this CA
  --tlscert="~/.docker/cert.pem"         Path to TLS certificate file
  --tlskey="~/.docker/key.pem"           Path to TLS key file
  --tlsverify=false                      Use TLS and verify the remote
  --userland-proxy=true                  Use userland proxy for loopback traffic
```

如果你想要运行守护态进程，你可以输入 `docker -d`（之前版本是 `docker deamon`）。如果想加入Debug模式，输入`docker -d -D`即可。

- Deamon socket 选项

Docker deamon 通过三种不同的socket方式监听`docker remote API`请求，分别是：`unix、tcp、以及fd。`

默认情况下，通过创建在`/var/run/docer.sock`文件内的`unix domain socket`（或者 IPC socket）来接收root或者docker用户组的请求。如果你想远程通信你需要打开tcpSocket。

要注意的是，默认的方式提供了一个未加密未验证直接连接deamon。应该使用内置的HTTPS加密的socket或者在前面使用一个安全的web代理。使用-H `tcp://0.0.0.0:2375`来监听所有ip地址接口的2375端口，或者指定一个主机IP监听`-H 192.168.2.160:2375`。通常情况下2375端口是 未加密的，而2376用于与deamon通信的加密端口。

```
注意：如果你使用HTTPS加密socket ，目前支持TLS1.0或更高级的协议，不支持Protocols SSLv3或者低于此版本的协议。
```

在Systemd基础的系统中，使用`docker -d -H fd://`,通过Systemd socket activation与deamon通信。对于大多数设置，使用fd://将很好的运作，你也可以指定单个socket：`docker -d -H fd://3`。如果没有找到指定的激活的文件，Docker 将会退出进程。

1. **Server端**

   -H参数可以多次指定监听不同的端口：

   例如指定监听主机默认的unix socket以及指定的IP地址：

   ```
    $ sudo docker -d -H unix:///var/run/docker.sock  -H tcp://192.168.2.160:2375
   ```

2. **Client端**

   为客户端设置-H参数，将使客户端监听`DOCKER_HOST`环境变量指定的参数：

   ```
    $ docker -H tcp://0.0.0.0:2375 ps
   ```

   或者

   ```
    $ export DOCKER_HOST="tcp://0.0.0.0:2375"
    $ docker ps
   ```

   设置 `DOCKER_TLS_VERIFY`环境变量相当于设置`--tlsverify`参数：

   ```
    $ docker --tlsverify ps
   ```

   或者

   ```
    $ export DOCKER_TLS_VERIFY=1
    $ docker ps
   ```

   以上设置是等效的

   Docker客户端会遵守`HTTP_PROXY,HTTPS_PROXY以及NO_PROXY`这三个环境变量运行。其中`HTTPS_PROXY`优先权大于`HTTP_PROXY`

- storage-driver 选项

  Docker deamon 支持许多不同的镜像层存储驱动：aufs、devicemapper、btrfs、zfs以及overlay。

  1. aufs是最老的，但是由于它是基于linux 内核patch-set,不太可能被合并到主内核中。这也会导致一些严重的系统崩溃。但是，aufs也是唯一允许容器共享可执行文件以及共享类库内存的存储驱动，所以对于那些需要运行数以千计运行相同程序或类库的容器会非常有用。

  2. devicemapper使用自动精简配置以及Copy on Write(COW)快照。对于每一个graph位置通常是在/var/lib/docker/devicemapper中，通常被分为两块设备，一块给数据，一块给metadata。默认的，这些块设备是通过使用自动创建的零散文件回送挂载来自动创建的。Refer to Storage driver options below for a way how to customize this setup.~jpetazzo/Resizing Docker containers with the Device Mapper plugin article explains how to tune your existing setup without the use of options.

  3. Btrfs 对于docker build构建镜像时会非常快，但是和devicemapper一样不会共享可执行文件以及类库的内存。使用方法：

     ```
      docker -d -s btrfs -g /mnt/btrfs_partition
     ```

  4. Zfs 没有btrfs那么快，但是对相对较长记录有更稳定地支持。由于克隆之间的单一副本ARC共享块将被一次缓存，使用方法：

     ```
      docker -d -s zfs
     ```

     Use docker daemon -s zfs. To select a different zfs filesystem set zfs.fsname option as described in Storage driver options.

  5. Overlay 是一个非常快的联合文件系统，它现在被并入了3.18.0的Linux内核中，使用方法：

     ```
      docker -d -s overlay
     ```

- storage-opt选项

  ***`dm.thinpooldev`\***,指定块存储设备所使用的thin pool。

  ```
    docker -d --storage-opt dm.thinpooldev=/dev/mapper/thin-pool
  ```

  ***`dm.basesize`\*** 指定基础存储大小，同时限制镜像以及容器。默认值时100G。 修改此值需要执行以下操作才生效：

  ```
    $ sudo service docker stop
    $ sudo rm -rf /var/lib/docker
    $ sudo service docker start
  ```

  使用方法：

  ```
    $ docker -d --storage-opt dm.basesize=20G
  ```

  ***`dm.loopdatasize`\*** 这个选项配置devicemapper looback，这不应该在生产中使用。默认值是100G，用于设定thin pool为数据产生的回送的零散文件存储大小，通常不会占用那么多空间。

  使用方法：

  ```
    $ docker -d --storage-opt dm.loopdatasize=200G
  ```

  ***`dm.loopmetadatasize`\*** 与上面类似，只是设定元数据存储大小。

  使用方法

  ```
    $ docker -d --storage-opt dm.loopmetadatasize=4G
  ```

  ***`dm.fs`\*** 设定文件系统基础设备类型，支持的类型是ext4和xfs，默认是ext4

  使用方法：

  $ docker -d --storage-opt dm.fs=xfs

  ***`dm.mkfsarg`\*** 设定在创建基础设备时mkfs所用到的参数

  使用方法：

  ```
    $ docker -d --storage-opt "dm.mkfsarg=-O ^has_journal"
  ```

  ***`dm.mountopt`\*** 挂载设备时设置挂载选项。

  使用方法：

  ```
    $ docker -d --storage-opt dm.mountopt=nodiscard
  ```

  ***`dm.blocksize`\*** 为thin pool 设置块大小。默认是64K

  使用方法：

  $ docker -d --storage-opt dm.blocksize=512K

  ***`dm.blkdiscard`\*** 当删除devicemapper设备时允许或禁止使用blkdiscard 默认是允许（enable）。如果禁止，将会时删除容器更加快速，但是不会返回其中文件的使用空间。

  使用说明：

  ```
    $ docker -d --storage-opt dm.blkdiscard=false
  ```

  ***`dm.override_udev_sync_check`\*** 设置该参数为true，可以协调devicemapper 与 udev的资源利用。当其设置为false时，将会在devicemapper与udev产生竞争，有可能导致错误或者失败。

  使用方法：

  ```
    $ docker -d --storage-opt dm.override_udev_sync_check=true
  ```

- Docker execdriver选项

  目前zfs支持的选项`zfs.fsname`

  使用方法：

  ```
    $ docker daemon -s zfs --storage-opt zfs.fsname=zroot/docker
  ```

  另外，可以使用 `-e lxc` 来启用`lxcexecution` 设备

- Daemon DNS选项

  设置dns 服务器

  ```
    $  docker -d --dns 8.8.8.8
    $  docker -d --dns-search example.com
  ```

- 不安全仓库登记

  一个安全的私有仓库通过使用TLS和CA证书的副本来替换`/etc/docker/certs.d/myregistry:5000/ca.crt`文件。不使用TLS，或者使用未知CA证书的TLS都将是不安全的。如果CA证书验证实效或者在`/etc/docker/certs.d/myregistry:5000/`找不到证书将会报错。使用`--insecure-registry`参数可以标记一个不安全的仓库：

  ```
    --insecure-registry myregistry:5000
  ```

  将告诉 deamon 这个`myregistry:5000`仓库应该标记为不安全状态。

  ```
    --insecure-registry 10.1.0.0/16
  ```

  告诉deamon通过CIDR语法解析出来的IP地址是`10.1.0.0/16`的仓库标记为不安全。

  如果没有使用参数`--insecure-registry`标记，那么`docker pull 、docker push、docker search` 从指定仓库执行时将会报错。

  

  ## 7.2 atttach 参数

  - 用法

  ```
  Usage: docker attach [OPTIONS] CONTAINER
  
  Attach to a running container
  
    --help=false        Print usage
    --no-stdin=false    Do not attach STDIN
    --sig-proxy=true    Proxy all received signals to the proces
  ```

  - 例子

  在使用-d参数时，容器启动后会进入后台。 某些时候需要进入容器进行操作，有很多种方法，包括使用 docker attach命令或 nsenter 工具等。

  ```
  $  sudo docker run -i -t -d centos
  911207826f4da78cb8b8a233dea6120a7d2939eea389a94eef2c0b1320572628
  $  sudo docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
  911207826f4d        centos:latest       "/bin/bash"         46 seconds ago       Up 45 seconds                           silly_poitras
  $ sudo docker attach 911207826f4d
  [root@911207826f4d /]#
  [root@911207826f4d /]#
  ```

  此时，我们以及进入一个正在运行的容器中去执行命令。

  - 总结

  使用 attach 命令有时候并不方便。当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

  ### 扩展工具 nsenter

  - 说明

  nsenter可以访问一个进程大名字空间。 指令时包含在untli-linux（2.23版本之后才会包含）软件包里。这里需要安装一下：

  ```
  $ sudo yum -y install util-linux
  ```

  完成后检验：

  ```
  $ nsenter -V
  nsenter from util-linux 2.23.2
  ```

  如果要进入容器内需要知道进程的pid。

  ```
  $ sudo  docker run -idt --name=test1 centos f629fa879a34af902a259e831de1cbc298db2b0f469aa49f80b80a0f81869943
  $ sudo docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  f629fa879a34        centos              "/bin/bash"         6 seconds ago       Up 5 seconds                            test1
  $ sudo PID=$(docker inspect --format "{{ .State.Pid }}" f629fa879a34)
  $ sudo  echo $PID
  22109
  $ sudo nsenter --target 22109 --mount --uts --ipc --net --pid
  [root@f629fa879a34 /]#
  [root@f629fa879a34 /]#
  [root@f629fa879a34 /]#
  ```

  这样就完成了进入容器内访问的目的。

  此外，为了方便进入容器，牛人已经为我们封装好指令，我们只需利用简单的两行代码就可完成操作。 下载这个脚本.bashrc_docker，并将内容放到 .bashrc 中：

  ```
  $ sudo wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
  $ sudo echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
  ```

  执行完后我们只需：

  ```
  $ sudo echo $(docker-pid f629fa879a34)
  22109
  $ sudo docker-enter f629fa879a34
  [root@f629fa879a34 ~]#
  实际上也是使用nsenter进入容器，只不过更简洁罢了。
  ```



## 7.3 build 参数

- 用法

```
Usage: docker build [OPTIONS] PATH | URL | -

Build a new image from the source code at PATH

 -f, --file=""            Name of the Dockerfile (Default is 'PATH/Dockerfile')
 --force-rm=false         Always remove intermediate containers
 --no-cache=false         Do not use cache when building the image
 --pull=false             Always attempt to pull a newer version of the image
 -q, --quiet=false        Suppress the verbose output generated by the containers
 --rm=true                Remove intermediate containers after a successful build
 -t, --tag=""             Repository name (and optionally a tag) for the image
 -m, --memory=""          Memory limit for all build containers
 --memory-swap=""         Total memory (memory + swap), `-1` to disable swap
 -c, --cpu-shares         CPU Shares (relative weight)
 --cpuset-mems=""         MEMs in which to allow execution, e.g. `0-3`, `0,1`
 --cpuset-cpus=""         CPUs in which to allow execution, e.g. `0-3`, `0,1`
 --cgroup-parent=""       Optional parent cgroup for the container
 --ulimit=[]              Ulimit options
```

- 例子

使用该命令，将会从参数指定的路径中的 Dockerfile的文件执行构建镜像，文件的指向可以是一个本地文件PATH或者是一个URL。

例如：

```
$ sudo docker build https://github.com/docker/rootfs.git#container:docker
```

或者用标准输入：

```
$ sudo docker build - < Dockerfile
```

如果你采用以上两种方式构建镜像，-f 或者－file参数将失效。

默认情况下，docker build 指令将会在指定根目录下查找Dockerfile文件，如果指定-f/-file参数，将指定该构建目录文件，这样的好处是可以多次构建。需要注意的是，路径必须包含构建信息的文件。

在多数情况下，最好保证构建目录为空。然后添加所需要的软件包到该文件夹。为了提高构建效率，可以加入 .dockerignore 文件排除一些不需要的文件。

返回值

如果构建成功，将会返回0，当失败时，将会返回相应错误返回值：

```
$ docker build -t fail .
Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon
Step 0 : FROM busybox
 ---> 4986bf8c1536
Step 1 : RUN exit 13
 ---> Running in e26670ec7a0a
INFO[0000] The command [/bin/sh -c exit 13] returned a non-zero code: 13
$ echo $?
1
```

一般例子：

```
$ docker build .
Uploading context 10240 bytes
Step 1 : FROM busybox
Pulling repository busybox
 ---> e9aa60c60128MB/2.284 MB (100%) endpoint: https://cdn-registry-1.docker.io/v1/
Step 2 : RUN ls -lh /
 ---> Running in 9c9e81692ae9
total 24
drwxr-xr-x    2 root     root        4.0K Mar 12  2013 bin
drwxr-xr-x    5 root     root        4.0K Oct 19 00:19 dev
drwxr-xr-x    2 root     root        4.0K Oct 19 00:19 etc
drwxr-xr-x    2 root     root        4.0K Nov 15 23:34 lib
lrwxrwxrwx    1 root     root           3 Mar 12  2013 lib64 -> lib
dr-xr-xr-x  116 root     root           0 Nov 15 23:34 proc
lrwxrwxrwx    1 root     root           3 Mar 12  2013 sbin -> bin
dr-xr-xr-x   13 root     root           0 Nov 15 23:34 sys
drwxr-xr-x    2 root     root        4.0K Mar 12  2013 tmp
drwxr-xr-x    2 root     root        4.0K Nov 15 23:34 usr
 ---> b35f4035db3f
Step 3 : CMD echo Hello world
 ---> Running in 02071fceb21b
 ---> f52f38b7823e
Successfully built f52f38b7823e
Removing intermediate container 9c9e81692ae9
Removing intermediate container 02071fceb21b
```

上面例子中，指定路径是 .,这个路径告诉docker构建的目录为当前目录，里面包含构建文件的信息，以及所要添加的文件。如果想保留构建过程中的容器，可以使用--rm=false ，这样操作不会影响构建缓存。

下面这个例子使用了.dockerignore文件来排除.git文件的使用方法，将会影响上下文文件大小。

```
$ docker build .
Uploading context 18.829 MB
Uploading context
Step 0 : FROM busybox
 ---> 769b9341d937
Step 1 : CMD echo Hello world
 ---> Using cache
 ---> 99cc1ad10469
Successfully built 99cc1ad10469
    $ echo ".git" > .dockerignore
$ docker build .
Uploading context  6.76 MB
Uploading context
Step 0 : FROM busybox
 ---> 769b9341d937
Step 1 : CMD echo Hello world
 ---> Using cache
 ---> 99cc1ad10469
Successfully built 99cc1ad10469
```

使用-t参数指定name以及tag：

```
$ docker build -t vieux/apache:2.0 .
```

从标准输入读取Dockerfile：

```
$ docker build - < Dockerfile
```

使用压缩文件，目前支持的格式是bzip2, gzip and xz

```
$ docker build - < context.tar.gz
```

从克隆的GitHub仓库作为上下文构建镜像，在仓库根目录下的Dockerfile文件将作为构建文件。

```
$ docker build github.com/creack/docker-firefox
```

注意，若要加前缀必须是 git:// 或者 git@ 。

使用-f参数指定文件构建

```
$ docker build -f Dockerfile.debug .
```

在.目录下从不同文件构建镜像：

```
$ docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
$ docker build -f dockerfiles/Dockerfile.prod  -t myapp_prod .
```

我们在观察下面例子：

```
$ cd /home/me/myapp/some/dir/really/deep
$ docker build -f /home/me/myapp/dockerfiles/debug /home/me/myapp
$ docker build -f ../../../../dockerfiles/debug /home/me/myapp
```

这个例子执行的两次构建操作所做事情是一模一样的，都会寻找debug文件作为Dockerfile来构建镜像。



## 7.4 commit 参数

- 用法

```
Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

-a, --author=       Author (e.g., "John Hannibal Smith <[email protected]>")
-c, --change=[]     Apply Dockerfile instruction to the created image
--help=false        Print usage
-m, --message=      Commit message
-p, --pause=true    Pause container during commit
```

- 例子

```
$ sudo docker ps
ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
$ sudo docker commit c3f279d17e0a  SvenDowideit/testimage:version3
f5283438590d
$ sudo docker images | head
REPOSITORY                        TAG                 ID                  CREATED             VIRTUAL SIZE
SvenDowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 M
```

提交一个重新配置过的容器到镜像

```
$ sudo docker ps
ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
$ sudo docker inspect -f "{{ .Config.Env }}" c3f279d17e0a
[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
$ sudo docker commit --change "ENV DEBUG true" c3f279d17e0a SvenDowideit/testimage:version3
f5283438590d
$ sudo docker inspect -f "{{ .Config.Env }}" f5283438590d
[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin DEBUG=true]
```

- 总结

这个命令的用处在于把有修改的container提交成新的Image，然后导出此Imange分发给其他场景中调试使用。Docker官方的建议是，当你在调试完Image的问题后，应该写一个新的Dockerfile文件来维护此Image。commit命令仅是一个临时创建Imange的辅助命令。



## 7.5 cp 参数

- 用法

```
Usage: docker cp [OPTIONS] CONTAINER:PATH HOSTDIR|-

Copy files/folders from a PATH on the container to a HOSTDIR on the host running the command. Use '-' to write the data as a tar file to STDOUT.

  --help=false       Print usage
```

- 例子

```
$ sudo docker cp hopeful_feynman:/etc /home
```

这将会在主机的/home目录下多一个etc文件夹，该文件夹就是从容器中复制出来的。

- 总结

使用cp可以把容器內的文件复制到Host主机上。这个命令在开发者开发应用的场景下，会需要把运行程序产生的结果复制出来的需求，在这个情况下就可以使用这个cp命令。



## 7.6 diff 参数

- 用法

```
Usage: docker diff [OPTIONS] CONTAINER

Inspect changes on a container's filesystem

  --help=false       Print usage
```

- 例子

```
$ sudo docker diff b448f729a0b0
C /run
A /run/secrets
```

- 总结

diff会列出3种容器内文件状态变化（A - Add, D - Delete, C - Change ）的列表清单。构建Image的过程中需要的调试指令。



## 7.7 events 参数

- 用法

```
Usage: docker events [OPTIONS]

Get real time events from the server

-f, --filter=[]    Filter output based on conditions provided
--help=false       Print usage
--since=           Show all events created since timestamp
--until=           Stream events until this timestamp
```

- 例子

第一个窗口用来监听事件

```
$ docker events
```

第二个窗口 起停容器

```
$ docker start 4386fb97867d
$ docker stop 4386fb97867d
$ docker stop 7805c1d35632
```

执行完后，shell窗口会同步打印如下信息：

```
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) start
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
```

使用since参数按时间筛选

```
$ sudo docker events --since 1378216169
2014-03-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-03-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ sudo docker events --since '2013-09-03'
2014-09-03T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) start
2014-09-03T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-09-03T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ sudo docker events --since '2013-09-03T15:49:29'
2014-09-03T15:49:29.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-09-03T15:49:29.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
```

只保留三分钟内的事件

```
$ sudo docker events --since '3m'
2015-05-12T11:51:30.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2015-05-12T15:52:12.999999999Z07:00 4 4386fb97867d: (from ubuntu-1:14.04) stop
2015-05-12T15:53:45.999999999Z07:00  7805c1d35632: (from redis:2.8) die
2015-05-12T15:54:03.999999999Z07:00  7805c1d35632: (from redis:2.8) stop
```

也可以使用过滤器筛选

```
$ docker events --filter 'event=stop'
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-09-03T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ docker events --filter 'image=ubuntu-1:14.04'
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) start
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
$ docker events --filter 'container=7805c1d35632'
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-09-03T15:49:29.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ docker events --filter 'container=7805c1d35632' --filter 'container=4386fb97867d'
2014-09-03T15:49:29.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-09-03T15:49:29.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ docker events --filter 'container=7805c1d35632' --filter 'event=stop'
2014-09-03T15:49:29.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
$ docker events --filter 'container=container_1' --filter 'container=container_2'
2014-09-03T15:49:29.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) die
2014-05-10T17:42:14.999999999Z07:00 4386fb97867d: (from ubuntu-1:14.04) stop
2014-05-10T17:42:14.999999999Z07:00 7805c1d35632: (from redis:2.8) die
2014-09-03T15:49:29.999999999Z07:00 7805c1d35632: (from redis:2.8) stop
```

- 总结

  打印容器实时的系统事件。



## 7.8 export 参数

- 用法

```
Usage: docker export [OPTIONS] CONTAINER

Export a filesystem as a tar archive (streamed to STDOUT by default)

  --help=false       Print usage
  -o, --output=      Write to a file, instead of STDOUT
```

- 例子

```
$ sudo docker export b448f729a0b0 > centos.tar
```

- 总结

把容器系统文件打包并导出来，方便分发给其他场景使用。



## 7.9 import 参数

- 用法

```
Usage: docker import [OPTIONS] URL|- [REPOSITORY[:TAG]]

Create an empty filesystem image and import the contents of the
tarball (.tar, .tar.gz, .tgz, .bzip, .tar.xz, .txz) into it, then
optionally tag it.

 -c, --change=[]    Apply Dockerfile instruction to the created image
  --help=false       Print usage
```

- 例子

从网络上导入：

```
$ sudo docker import http://example.com/exampleimage.tgz
```

从本地文件导入:

通过标准输入和pipe导入到docker.

```
$ cat exampleimage.tgz | sudo docker import - exampleimagelocal:new
```

从本地目录导入：

```
$ sudo tar -c . | docker import - exampleimagedir
```

带配置信息从本地目录导入：

```
$ sudo tar -c . | docker import --change "ENV DEBUG true" - exampleimagedir
```



## 7.10 history 参数

- 用法

```
Usage: docker history [OPTIONS] IMAGE

Show the history of an image

-H, --human=true     Print sizes and dates in human readable format
--help=false         Print usage
--no-trunc=false     Don't truncate output
-q, --quiet=false    Only show numeric IDs
```

- 例子

```
$ sudo docker history postgres
IMAGE               CREATED             CREATED BY                                      SIZE                 COMMENT
730d1d72bda2        4 weeks ago         /bin/sh -c #(nop) CMD ["postgres"]              0 B
3e840dbb5474        4 weeks ago         /bin/sh -c #(nop) EXPOSE 5432/tcp               0 B
4df8a54cf33a        4 weeks ago         /bin/sh -c #(nop) ENTRYPOINT &{["/docker-entr   0 B
09e02a9f8afe        4 weeks ago         /bin/sh -c #(nop) COPY file:090d83d34addb45c3   2.761 kB
39172f8b90f2        4 weeks ago         /bin/sh -c #(nop) VOLUME [/var/lib/postgresql   0 B
3fa84fbfdec9        4 weeks ago         /bin/sh -c #(nop) ENV PGDATA=/var/lib/postgre   0 B
c5d75e7f9094        4 weeks ago         /bin/sh -c #(nop) ENV PATH=/usr/lib/postgresq   0 B
a95070c23e86        4 weeks ago         /bin/sh -c mkdir -p /var/run/postgresql && ch   0 B
64957633c267        4 weeks ago         /bin/sh -c apt-get update &&  apt-get install   116.4 MB
a814508841fa        4 weeks ago         /bin/sh -c echo 'deb http://apt.postgresql.or   66 B
49915906faae        4 weeks ago         /bin/sh -c #(nop) ENV PG_VERSION=9.4.4-1.pgdg   0 B
b41b53da5fba        4 weeks ago         /bin/sh -c #(nop) ENV PG_MAJOR=9.4              0 B
02fa71f1fa38        4 weeks ago         /bin/sh -c apt-key adv --keyserver ha.pool.sk   3.212 kB
0b82f508e063        4 weeks ago         /bin/sh -c mkdir /docker-entrypoint-initdb.d    0 B
e07b5a739ed9        4 weeks ago         /bin/sh -c #(nop) ENV LANG=en_US.utf8           0 B
c783ebe7a1d4        4 weeks ago         /bin/sh -c apt-get update && apt-get install    19.54 MB
8b6b2a3b7f9c        4 weeks ago         /bin/sh -c apt-get update && apt-get install    3.758 MB
22ed955cce18        5 weeks ago         /bin/sh -c gpg --keyserver pool.sks-keyserver   98.87 kB
26a84c436db4        5 weeks ago         /bin/sh -c groupadd -r postgres && useradd -r   330.4 kB
9a61b6b1315e        5 weeks ago         /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B
902b87aaaec9        5 weeks ago         /bin/sh -c #(nop) ADD file:e1dd18493a216ecd0c   125.2 MB
```

- 总结

打印指定Image中每一层Image命令行的历史记录。



## 7.11 images 参数

- 使用方法

```
docker images [OPTIONS] [REPOSITORY]

List images

 -a, --all=false      Show all images (default hides intermediate images)
  --digests=false      Show digests
  -f, --filter=[]      Filter output based on conditions provided
  --help=false         Print usage
  --no-trunc=false     Don't truncate output
  -q, --quiet=false    Only show numeric IDs
```

- 例子：

查询本里存储的镜像

```
$ sudo docker imgaes
REPOSITORY           TAG       IMAGE ID        CREATED      VIRTUAL SIZE
docker.io/ubuntu     latest    63e3c10217b8    7 days ago    188.3 MB
docker.google/etcd   2.1.1     2c319269dd15    8 days ago    23.32 MB
docker.io/postgres   latest    730d1d72bda2    2 weeks ago   265.3 MB
centos               latest    770327a1e9e7    2 weeks ago   418.9 MB
…
```

将ID完整展现

```
$ sudo docker images --no-trunc
REPOSITORY                  TAG                 IMAGE ID                                                           CREATED             VIRTUAL SIZE
scratch1                    latest              dc869bfd3085af05a1a070c7409193e8be88de00ff4560e2e9af80ffa9d2041d   58 minutes ago      0 B
registry.liugang/centos     latest              770327a1e9e746cf8d4449a7134e87917982b33c7f5cea584d941350f5ead7ac   4 weeks ago         418.9 MB
registry.liugang/busybox    latest              8c2e06607696bd4afb3d03b687e361cc43cf8ec1a4a725bc96e39f05ba97dd55   4 months ago        2.43 MB
docker.io/scratch           latest              511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158   2 years ago         0 B
```

使用该命令将展现没有tag的镜像

```
$ sudo docker images --filter "dangling=true"
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              b133995b6291        About an hour ago   0 B
<none>              <none>              6fae83243a01        About an hour ago   0 B
<none>              <none>              4c6412305cfa        About an hour ago   0 B
```

- 总结

其中第一字段是image镜像的名称；TAG一般表示为版本号，也可以自己定义 ；IMAGE ID 表示镜像的唯一ID ，这也是判断两个镜像文件是否为同一个的判断标准。



## 7.12 info 参数

- 用法

```
Usage: docker info [OPTIONS]

Display system-wide information

  --help=false       Print usage
```

- 例子

```
$ sudo docker -D info
Containers: 6
Images: 30
Storage Driver: devicemapper
 Pool Name: docker-8:3-28326-pool
 Pool Blocksize: 65.54 kB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 1.37 GB
 Data Space Total: 107.4 GB
 Data Space Available: 44.49 GB
 Metadata Space Used: 2.245 MB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.145 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: false
 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
 Library Version: 1.02.93-RHEL7 (2015-01-28)
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.10.0-229.el7.x86_64
Operating System: CentOS Linux 7 (Core)
CPUs: 1
Total Memory: 979.7 MiB
Name: localhost.localdomain
ID: PRVB:3SDE:YL4E:JT5P:5BIR:BUC5:PHXI:HG4B:P753:Y2BI:U7OU:YPGC
```

- 总结

这个命令在开发者报告Bug时会非常有用，结合docker vesion一起，可以随时使用这个命令把本地的配置信息提供出来，方便Docker的开发者快速定位问题。



## 7.13 inspect 参数

- 用法

```
Usage: docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]

Return low-level information on a container or image

-f, --format=         Format the output using the given go template
--help=false          Print usage
-r, --remote=false    Inspect remote images
```

- 例子

```
$ sudo docker inspect centos
[
{
    "Id": "770327a1e9e746cf8d4449a7134e87917982b33c7f5cea584d941350f5ead7ac",
    "Parent": "67c02c69a0fc420e781b9a1c676f19306e999aac2cf3ba24dfa4e0b9a5e34b5e",
    "Comment": "",
    "Created": "2015-07-24T01:06:38.020790544Z",
    "Container": "9163378b9f7fe2887bce56cb726c3845ce9af8ebc9cbef30d3af6315429a27ad",
"ContainerConfig": {
    "Hostname": "9163378b9f7f",
    "Domainname": "",
    "User": "",
    "AttachStdin": true,
     ...
     ...
     ...
    }
```

取出某一个值：

```
$ sudo docker inspect --format="{{.Id}}" centos
770327a1e9e746cf8d4449a7134e87917982b33c7f5cea584d941350f5ead7ac
```

- 总结

查看容器运行时详细信息的命令。了解一个Image或者Container的完整构建信息就可以通过这个命令实现。



## 7.14 login 参数

- 用法

```
Usage: docker login [OPTIONS] [SERVER]

Register or log in to a Docker registry server, if no server is
specified "https://index.docker.io/v1/" is the default.

-e, --email=       Email
--help=false       Print usage
-p, --password=    Password
-u, --username=    Username
```

- 例子

```
root@liugang:~# docker login
Username: username
Password: ****
Email: [email protected]
Login Succeeded
```

如果你有一个自己的仓库，你也可以连接到指定主机：

```
$ sudo docker login localhost:8080
```



## 7.15 logout 参数

- 用法

```
Usage: docker logout [OPTIONS] [SERVER]

Log out from a Docker registry, if no server is
specified "https://index.docker.io/v1/" is the default.

--help=false       Print usage
```

- 例子

```
$ sudo docker logout localhost:8080
```



## 7.16 logs 参数

- 用法

```
Usage: docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

-f, --follow=false        Follow log output
--help=false              Print usage
--since=                  Show logs since timestamp
-t, --timestamps=false    Show timestamps
--tail=all                Number of lines to show from the end of the logs
```

- 例子：打印某个容器日志

```
$ sudo docker logs 60095325e584
```

- 例子：实时监控容器某个容器日志输出

```
$ sudo docker logs -f 60095325e584
```

- 总结

批量打印出容器中进程的运行日志。



## 7.17 network 参数

### network connect

你可以使用容器名称或者ID，将一个正在运行的容器介入网络。连接成功后容器酒可以与处于同一个网络中的容器通信。

```
docker network connect multi-host-network coantainer1
```

你也可以使用 docker run --net= 选项来启动一个容器直接连接到一个已知网络。

```
docker run -idt --net=multi-host-network busybox
```

你可以暂停，重启甚至终止已连接网络的容器。暂停容器将保持网络连接以及网络发现（ by a network inspect）。终止容器，将使容器在该网络上消失，直到重启才可以被发现。当容器重启以后，容器重新加入该网络将不保证IP地址保持与原来一致。

使用 docker network inspect 命令来验证容器的网络是否已连接，而使用 docker network disconnect 来从网络上断开与容器的连接、

当容器连接到网络时，容器职能使用容器IP地址或者容器name来通信。对于overlay网络或者其它通过插件配置的跨主机环境的网络，然可以使用这种方式运行。

你可以使容器连接一个活多个网络，这些网络不要是相同类型的。例如：你可以连接一个容器网桥和overlay网络。

### network create

该命令用于创建一个网络。使用 -d参数允许使用bridge或者overlay类型的网络构建与网络驱动中。如果你有第三种网络结构或者其它通用网络驱动，你也可以使用该参数特别说明。

如果不指定 --driver 参数，该命令将自动为你创建一个bridge类型的网络。该网络对应与传统的docker0网桥。当使用 docker run 启动一个容器时，它将自动连接到这个bridge网络。你不能删除这个默认的网络但是可以使用docker network create 命令创建一个新的：

```
docker network create -d bridge my-bridge-network
```

birdge网络是单docker引擎的隔离网络（Bridge networks are isolated networks on a single Engine installation）。如果你想创建一个跨越多个docker主机引擎的网络，你必须创建一个overlay类型的网络。与birdge网络不同，overlay网络创建需要提前准备一些配置：

```
1、需要连接一个 key－value 存储，目前支持Consul，Etcd以及Zookeeper（分布式存储）key－value 存储。

2、一个连接到 key－value 存储的云主机

3、每一台机器上的 docker deamon 都要配置相同参数
```

docker deamon 支持overlay选项的参数有：

```
--cluster-store
--cluster-store-opt
--cluster-advertise
```

想要了解更多有关配置以上参数的信息，请阅读 “Get started with multi-host network“

你可以安装 docker swarm 来管理集群建立自己的网络，这是一个不错的想法，但不是必须的。 Swarm提供先进的服务发现以及节点管理来帮助你实现。

当你准备好你的overlay网络时，你只需选择集群中的一个docker主机并执行以下命令：

```
docker network create -d overlay my-multihost-network
```

网络名称必须是唯一的，docker deamon 会尝试验证命名冲突，但并不保证（我就呵呵了）。用户有责任去避免命名冲突（呵呵呵。。。）。

### connect containers

当你使用 --net 参数连接到一个网络时，这将会使目标容器连接到自定义网络中去：

```
docker run -idt --net=mynet busybox
```

如果你想在一个容器运行之后添加到一个网络中去，可以使用 docker network connect 命令。

你可以将多个容器连接到相同的网络中去。一旦连接，容器将只能通过IP或者容器名称进行通信。对于overlay网络或者其它通过插件配置的跨主机环境的网络，然可以使用这种方式运行。

使用 docker network disconnect 可以断开容器与网络的连接

### Specifying advanced options

当创建一个网络时，docker Engine 会默认为该网络创建一个非重叠子网。这个子网并不是已存在子网的划分，它纯粹为了IP寻址（It is purely for ip-addressing purposes）。你可以覆盖这个默认的，然后使用 --subnet 选项来特别定义。在bridge网络上你可以这样定义：

```
docker network create -d --subnet=192.168.0.0/16
```

此外，你还可以指定 --gateway --ip-range 以及 --aux-addressoptions。

```
docker network create --driver=bridge  --subnet=172.28.0.0/16  --ip-range=172.28.5.0 --gateway=172.28.5.154
```

如果你省略了 --gateway 选项，docker Engine 将会从内置的 preferred pool 为你选择一个。

对于overlay网络，你可以创建多个子网：

```
docker network create -d overlay
--subnet=192.168.0.0/16 --subnet=192.170.0.0/16
--gateway=192.168.0.100 --gateway=192.170.0.100
--ip-range=192.168.1.0/24
--aux-address a=192.168.1.5 --aux-address b=192.168.1.6
--aux-address a=192.170.1.5 --aux-address b=192.170.1.6
my-multihost-newtork
```

但是确保你的子网不要重叠，否则，创建网络就会失败，Engine 将会反悔错误。

### network disconnect

断开容器与网络的连接。

```
 docker network disconnect multi-host-network container1
```

### network ls

列出deamon知道的所有的网络，包括跨多主机的集群网络。

```
 sudo docker network ls
NETWORK ID          NAME                DRIVER
7fca4eb8c647        bridge                 bridge
9f904ee27bf5        none                    null
cf03ee007fb4        host                      host
78b03ee04fc4        multi-host          overlay
```

使用 --no-trunc 选项来显示整个网络的ID

```
docker network ls --no-trunc
NETWORK ID                                                         NAME                DRIVER
18a2866682b85619a026c81b98a5e375bd33e1b0936a26cc497c283d27bae9b3   none                null                
c288470c46f6c8949c5f7e5099b5b7947b07eabe8d9a27d79a9cbf111adcbf47   host                host                
7b369448dccbf865d397c8d2be0cda7cf7edc6b0945f77d2529912ae917a0185   bridge              bridge              
95e74588f40db048e86320c6526440c504650a1ff3e9f7d60a497c4d2163e5bd   foo                 bridge    
```

### network rm

删除一个网络，在删除该网络之前，必须断开与该网络连接的任何容器。

```
docker network rm my-network
```



## 7.18 search 参数

说明：搜索镜像仓库

- 用法

```
Usage: docker search [OPTIONS] TERM

Search the Docker Hub for images

  --automated=false    Only show automated builds
  --help=false         Print usage
  --no-index=false     Don't prepend index to output
  --no-trunc=false     Don't truncate output
  -s, --stars=0        Only displays with at least x stars
```

- 例子

```
$ sudo docker search ubuntu
```

从官方仓库中搜索出含有关键字ubuntu的镜像：

```
INDEX       NAM                                       DESCRIPTION                                       STARS         OFFICIAL            AUTOMATED
docker.io   docker.io/ubuntu                          Ubuntu is a Debian-based Linux operating s...     2046           [OK]
docker.io   docker.io/ubuntu-upstart                  Upstart is an event-based replacement for ...     30             [OK]
docker.io   docker.io/torusware/speedus-ubuntu        Always updated official Ubuntu docker imag...     25                                       [OK]
docker.io   docker.io/dorowu/ubuntu-desktop-lxde-vnc  Ubuntu with openssh-server and NoVNC on po...     20                                       [OK]
docker.io   docker.io/sequenceiq/hadoop-ubuntu        An easy way to try Hadoop on Ubuntu               19                                       [OK]
docker.io   docker.io/tleyden5iwx/ubuntu-cuda         Ubuntu 14.04 with CUDA drivers pre-installed      16                                       [OK]
docker.io   docker.io/ubuntu-debootstrap              debootstrap --variant=minbase --components...     12             [OK]
…
```

- 总结

在使用docker创建容器时，必然要用到镜像文件。这时我们就得从仓库中拉取我们所需要的image文件。



## 7.19 pull 参数

说明：拉取镜像仓库

- 用法

```
Usage: docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from the registry

-a, --all-tags=false    Download all tagged images in the repository
--help=false            Print usage
```

- 例子

找到所需要的镜像：

```
$ sudo docker pull docker.io/ubuntu:12.04
```

这里是从官方仓库中拉取下来版本号(TAG)为12.04的镜像，其中“docker.io” 可以不写，默认是从官方仓库下载。版本号(TAG)不写的话默认会拉取一个版本号为latest的镜像文件：

```
Trying to pull repository docker.io/ubuntu ...
d0e008c6cf02: Download complete
a69483e55b68: Download complete
bc99d1f906ec: Download complete
3c8e79a3b1eb: Download complete
Status: Downloaded newer image for docker.io/ubuntu:12.04
```

见到如上类似结果说明镜像拉取成功。现在看一下自己的仓库，多了一个12.04的镜像。

```
$ sudo docker images
REPOSITORY         TAG      IMAGE ID       CREATED      VIRTUAL SIZE
docker.io/ubuntu   latest   63e3c10217b8   7 days ago   188.3 MB
docker.io/ubuntu   12.04    d0e008c6cf02   7 days ago   134.7 MB
docker.google/etcd 2.1.1    2c319269dd15   8 days ago   23.32 MB
docker.io/postgres latest   730d1d72bda2   2 weeks ago  265.3 MB
…
```



## 7.20 push 参考

说明：将镜像上传到仓库

- 用法

```
Usage: docker push [OPTIONS] NAME[:TAG]

Push an image or a repository to the registry

  -f, --force=false    Push to public registry without confirmation
  --help=false         Print usage
```

- 例子

```
$ sudo docker push docker.io/ubuntu:latest
```

- 总结

镜像的上传，push 默认是向官方仓库上传，由于服务器在国外，传输速度非常慢，就没试验成功过。注意的是，需要在docker hub上注册过后才可以上传镜像哦。关于私有仓库的上传将在后面章节详细讲解。



## 7.21 ps 参数

- 用法

  Usage: docker ps [OPTIONS]

  List containers

  ```
  -a, --all=false       Show all containers (default shows just running)
  --before=             Show only container created before Id or Name
  -f, --filter=[]       Filter output based on conditions provided
  --help=false          Print usage
  -l, --latest=false    Show the latest created container, include non-running
  -n=-1                 Show n last created containers, include non-running
  --no-trunc=false      Don't truncate output
  -q, --quiet=false     Only display numeric IDs
  -s, --size=false      Display total file sizes
  --since=              Show created since Id or Name, include non-running
  ```

- 例子

```
$ sudo docker ps -a
CONTAINER ID  IMAGE    COMMAND       CREATED       STATUS                      PORTS    NAMES
b448f729a0b0  centos   "/bin/bash"   4 days ago    Exited (137) 4 days ago              pensive_wilson
54c7b6d6632e  centos   "/bin/bash"   4 days ago    Exited (0) 3 days ago                adoring_wozniak
```

利用筛选器筛选出exied状态时0的容器：

```
$ sudo docker ps -a --filter 'exited=0'
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS                  PORTS               NAMES
8d92293a65e9        registry.liugang/centos   "/bin/bash"         7 days ago          Exited (0) 5 days ago                       web
8410f389ea65        registry.liugang/centos   "/bin/bash"         7 days ago          Exited (0) 7 days ago                       test_link
```

- 总结

-a参数列出所有状态的容器， -l列出最新创建的容器，包括停止运行状态的容器。



## 7.22 kill 参数

- 用法

```
Usage: docker kill [OPTIONS] CONTAINER [CONTAINER...]

Kill a running container using SIGKILL or a specified signal

  --help=false         Print usage
  -s, --signal=KILL    Signal to send to the container
```

- 例子

```
$ sudo docker kill pensive_wilson
pensive_wilson
```

这将停止该容器

- 总结

结合ps命令，可以做到kill所有正在运行的容器：

```
$ sudo docker kill $(sudo docker ps -a -q)
```



## 7.23 rm 参数

- 用法

```
Usage: docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

  -f, --force=false      Force the removal of a running container (uses SIGKILL)
  --help=false           Print usage
  -l, --link=false       Remove the specified link
  -v, --volumes=false    Remove the volumes associated with the container
```

- 例子

```
$ sudo docker rm pensive_wilson
pensive_wilson
```

这将删除一个已经停止运行的容器，若容器正在运行，则将会使docker报错，停止容器再删除，或者加上-f参数强制删除（不建议）。

- 总结

类似的我们也结合ps删除所有容器：

```
$ sudo docker kill $(sudo docker ps -a -q)
...
...
...
$ sudo docker rm $(sudo docker ps -a -q)
...
...
...
```

要清空容器，首先要保证没有容器在运行。



## 7.24 rmi 参数

- 用法

```
Usage: docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

  -f, --force=false    Force removal of the image
  --help=false         Print usage
  --no-prune=false     Do not delete untagged parents
```

- 例子

```
$ sudo docker rmi centos:6.5
...
...
...
```

- 总结

要区分rm于rmi多用法。 与docker images命令配合来清空镜像：

```
$ sudo docker rmi $(sudo docker images -a -q)
```



## 7.25 port 参数

- 用法

```
Usage: docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]

List port mappings for the CONTAINER, or lookup the public-facing port that
is NAT-ed to the PRIVATE_PORT

  --help=false       Print usage
```

- 例子

```
$ sudo docker port 60095325e584
```

- 总结

打印出Host主机端口与容器暴露出的端口的NAT映射关系



## 7.26 pause 参数

- 用法

```
Usage: docker pause [OPTIONS] CONTAINER [CONTAINER...]

Pause all processes within a container

  --help=false       Print usage
```

- 例子

```
$ sudo docker pauese hopeful_feynman
hopeful_feynman
CONTAINER ID    IMAGE     COMMAND      CREATED         STATUS                 PORTS       NAMES
c9a12157fed7    centos    "/bin/bash"  9 minutes ago   Up 9 minutes (Paused)              hopeful_feynman
```

使容器内进程暂停



## 7.27 unpause 参数

- 用法

```
Usage: docker unpause [OPTIONS] CONTAINER [CONTAINER...]

Unpause all processes within a container

  --help=false       Print usage
```

- 例子

```
$ sudo docker pauese hopeful_feynman
hopeful_feynman
```

恢复暂停



## 7.28 create 参数

- 用法

```
Usage: docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

Create a new container

      -a, --attach=[]             Attach to STDIN, STDOUT or STDERR
      --add-host=[]               Add a custom host-to-IP mapping (host:ip)
      --blkio-weight=0            Block IO (relative weight), between 10 and 1000
      -c, --cpu-shares=0          CPU shares (relative weight)
      --cap-add=[]                Add Linux capabilities
      --cap-drop=[]               Drop Linux capabilities
      --cgroup-parent=            Optional parent cgroup for the container
      --cidfile=                  Write the container ID to the file
      --cpu-period=0              Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota=0               Limit the CPU CFS quota
      --cpuset-cpus=              CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems=              MEMs in which to allow execution (0-3, 0,1)
      --device=[]                 Add a host device to the container
      --dns=[]                    Set custom DNS servers
      --dns-search=[]             Set custom DNS search domains
      -e, --env=[]                Set environment variables
      --entrypoint=               Overwrite the default ENTRYPOINT of the image
      --env-file=[]               Read in a file of environment variables
      --expose=[]                 Expose a port or a range of ports
      -h, --hostname=             Container host name
      --help=false                Print usage
      -i, --interactive=false     Keep STDIN open even if not attached
      --init=                     Run container following specified init system container method (systemd)
      --ipc=                      IPC namespace to use
      -l, --label=[]              Set meta data on a container
      --label-file=[]             Read in a line delimited file of labels
      --link=[]                   Add link to another container
      --log-driver=               Logging driver for container
      --log-opt=[]                Log driver options
      --lxc-conf=[]               Add custom lxc options
      -m, --memory=               Memory limit
      --mac-address=              Container MAC address (e.g. 92:d0:c6:0a:29:33)
      --memory-swap=              Total memory (memory + swap), '-1' to disable swap
      --name=                     Assign a name to the container
      --net=bridge                Set the Network mode for the container
      --oom-kill-disable=false    Disable OOM Killer
      -P, --publish-all=false     Publish all exposed ports to random ports
      -p, --publish=[]            Publish a container's port(s) to the host
      --pid=                      PID namespace to use
      --privileged=false          Give extended privileges to this container
      --read-only=false           Mount the container's root filesystem as read only
      --restart=no                Restart policy to apply when a container exits
      --security-opt=[]           Security Options
      -t, --tty=false             Allocate a pseudo-TTY
     -u, --user=                 Username or UID (format: <name|uid>[:<group|gid>])
      --ulimit=[]                 Ulimit options
      --uts=                      UTS namespace to use
      -v, --volume=[]             Bind mount a volume
      --volumes-from=[]           Mount volumes from the specified container(s)
      -w, --workdir=              Working directory inside the container
```

- 例子

```
$ sudo docker create ubuntu /bin/echo 'Hello world'
a637c1d67506951928be296f2db02fa3e2b6e974ef371b181d9c26d1c8995963
$...
```

若有如上输出则代表容器创建成功

```
$ sudo docker ps -a
CONTAINER ID        IMAGE                    COMMAND                     CREATED             STATUS              PORTS               NAMES
a637c1d67506        ubuntu:latest            "/bin/echo 'Hello wo        4 minutes ago                                               mad_hopper
...
```

然后在启动它

```
$ sudo docker start a637c1d67506
a637c1d67506
$...
```

启动成功后会反回容器ID。

```
$ docker ps -a
CONTAINER ID        IMAGE                    COMMAND                   CREATED                STATUS                                 PORTS           NAMES
a637c1d67506        ubuntu:latest           "/bin/echo 'Hello wo       10 minutes ago         Exited (0) 2 minutes ago                               mad_hopper
```

- 总结

当我们去查看容器状态时，容器没有在运行，这时因为我们在创建容器的时候，让容器执行的命令是/bin/echo 'Hello world'，当容器执行完命令的时候就终止结束了。



## 7.29 run 参数

- 用法

```
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

     -a, --attach=[]             Attach to STDIN, STDOUT or STDERR
      --add-host=[]               Add a custom host-to-IP mapping (host:ip)
      --blkio-weight=0            Block IO (relative weight), between 10 and 1000
      -c, --cpu-shares=0          CPU shares (relative weight)
      --cap-add=[]                Add Linux capabilities
      --cap-drop=[]               Drop Linux capabilities
      --cgroup-parent=            Optional parent cgroup for the container
      --cidfile=                  Write the container ID to the file
      --cpu-period=0              Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota=0               Limit the CPU CFS quota
      --cpuset-cpus=              CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems=              MEMs in which to allow execution (0-3, 0,1)
      -d, --detach=false          Run container in background and print container ID
      --device=[]                 Add a host device to the container
      --dns=[]                    Set custom DNS servers
      --dns-search=[]             Set custom DNS search domains
      -e, --env=[]                Set environment variables
      --entrypoint=               Overwrite the default ENTRYPOINT of the image
      --env-file=[]               Read in a file of environment variables
      --expose=[]                 Expose a port or a range of ports
      -h, --hostname=             Container host name
      --help=false                Print usage
      -i, --interactive=false     Keep STDIN open even if not attached
      --init=                     Run container following specified init system container method (systemd)
      --ipc=                      IPC namespace to use
      -l, --label=[]              Set meta data on a container
      --label-file=[]             Read in a line delimited file of labels
      --link=[]                   Add link to another container
      --log-driver=               Logging driver for container
      --log-opt=[]                Log driver options
      --lxc-conf=[]               Add custom lxc options
      -m, --memory=               Memory limit
      --mac-address=              Container MAC address (e.g. 92:d0:c6:0a:29:33)
      --memory-swap=              Total memory (memory + swap), '-1' to disable swap
      --name=                     Assign a name to the container
      --net=bridge                Set the Network mode for the container
      --oom-kill-disable=false    Disable OOM Killer
      -P, --publish-all=false     Publish all exposed ports to random ports
      -p, --publish=[]            Publish a container's port(s) to the host
      --pid=                      PID namespace to use
      --privileged=false          Give extended privileges to this container
      --read-only=false           Mount the container's root filesystem as read only
      --restart=no                Restart policy to apply when a container exits
      --rm=false                  Automatically remove the container when it exits
      --security-opt=[]           Security Options
      --sig-proxy=true            Proxy received signals to the process
      -t, --tty=false             Allocate a pseudo-TTY
      -u, --user=                 Username or UID (format: <name|uid>[:<group|gid>])
      --ulimit=[]                 Ulimit options
      --uts=                      UTS namespace to use
      -v, --volume=[]             Bind mount a volume
      --volumes-from=[]           Mount volumes from the specified container(s)
      -w, --workdir=              Working directory inside the container
```

- 例子

用法与create类似，只是在创建容器后不需要进行start操作就可以运行。

```
$  sudo docker run ubuntu /bin/echo 'Hello world'
Hello world
$...
```

与上面一样，在运行完Hello world 之后也会退出容器。

**Daemonized（守护态）**

往往我们需要容器在后台一致执行，这时我们就需要在创建镜像的时候让容器以守护台方式(-d 参数)运行。

```
$ sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
61f37c1940c8ec9f08b107e99655b8a5181ded340415e3c15cf413069d556b73
$...
```

这时，我们查看一下容器状态：

```
$ sudo  docker ps -a
CONTAINER ID        IMAGE                  COMMAND                CREATED               STATUS                      PORTS               NAMES
61f37c1940c8        ubuntu:latest          "/bin/sh -c 'while t   4 seconds ago         Up 3 seconds                                    prickly_galileo
...
```

查看容器输出的信息

```
$ sudo docker logs 61f37c1940c8
hello world
hello world
hello world
hello world
...
```

- 总结

  让容器以后台方式运行，并不是加一个 -d 参数就可以，命令行COMMAND所执行的动作必须为持续运行的状态。



## 7.30 save 参数

- 用法

```
Usage: docker save [OPTIONS] IMAGE [IMAGE...]

Save an image(s) to a tar archive (streamed to STDOUT by default)

  --help=false       Print usage
  -o, --output=      Write to an file, instead of STDOUT
```

- 例子

载出镜像到文件

```
$ sudo docker save -o /home/ubuntu.tar  docker.io/ubuntu:latest
```

这样我们就在/home目录下找到ubuntu.tar 文件了

- 总结

在国内docker.io的下载速度奇慢，基本上下一个500M的image就可以搞你半天时间，这时我们就可以利用载入载出，从好朋友那里获取我们需要的镜像啦！



## 7.31 load 参数

- 用法

```
Usage: docker load [OPTIONS]

Load an image from a tar archive on STDIN

  --help=false       Print usage
  -i, --input=       Read from a tar archive file, instead of STDIN
```

- 例子

从文件载入镜像到本地

```
$ sudo docker load --input /home/ubuntu.tar
```

或者

```
$ sudo docker load < /home/ubuntu.tar
```

- 总结

这种方式将导入镜像以及其相关的元数据信息（包括标签等）。



## 7.32 start 参数

说明：启动容器参数

- 用法

```
Usage: docker start [OPTIONS] CONTAINER [CONTAINER...]

Start one or more stopped containers

-a, --attach=false         Attach STDOUT/STDERR and forward signals
--help=false               Print usage
-i, --interactive=false    Attach container's STDIN
```



## 7.33 stop 参数

说明：停止容器参数

- 用法

```
Usage: docker stop [OPTIONS] CONTAINER [CONTAINER...]

Stop a running container by sending SIGTERM and then SIGKILL after a
grace period

--help=false       Print usage
-t, --time=10      Seconds to wait for stop before killing it
```



## 7.34 restart 参数

说明：重启容器参数

- 用法

```
Usage: docker restart [OPTIONS] CONTAINER [CONTAINER...]

Restart a running container

--help=false       Print usage
-t, --time=10      Seconds to wait for stop before killing the container
```



## 7.34 stats 参数

- 用法

```
Usage: docker stats [OPTIONS] CONTAINER [CONTAINER...]

Display a live stream of one or more containers' resource usage statistics

--help=false         Print usage
--no-stream=false    Disable streaming stats and only pull the first result
```

- 例子

```
$ docker stats redis1 redis2
CONTAINER           CPU %               MEM USAGE/LIMIT     MEM %               NET I/O
redis1              0.07%               796 KB/64 MB        1.21%               788 B/648 B
redis2              0.07%               2.746 MB/64 MB      4.29%               1.266 KB/648 B
```

- 总结

该指令将只返回运行状态容器的数据流情况，停止状态的容器将不会返回任何数据。



## 7.35 tag 参数

说明：常用参数，主要用于镜像标签重命名

- 用法

```
Usage: docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
           docker tag -l [REGISTRYHOST/][USERNAME/]NAME...

Tag an image or list remote tags

-f, --force=false     Force
--help=false          Print usage
-l, --list=false      List repository tags
-r, --remote=false    Force listing of remote repositories only
```

- 例子

```
$ sudo docker tag docker.io/scratch:latest  local/scratch:my
```

- 总结

组合使用用户名，Image名字，标签名来组织管理Image。



## 7.36 top 参数

- 用法

```
Usage: docker top [OPTIONS] CONTAINER [ps OPTIONS]

Display the running processes of a container

--help=false       Print usage
```

- 例子

运行一个之前的例子：

```
$ sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

然后执行docker top 命令，可以查看到容器内进程

```
$ sudo docker top 52c058ff716d
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                6326                1489                0                   23:58               ?                   00:00:00            /bin/sh -c while true; do echo hello world; sleep 1; done
root                6410                6326                0                   23:59               ?          
```



## 7.37 wait 参数

说明：阻塞对指定容器的其他调用方法，直到容器停止后退出阻塞。

- 用法

```
Usage: docker wait [OPTIONS] CONTAINER [CONTAINER...]

Block until a container stops, then print its exit code.

--help=false       Print usage
```
