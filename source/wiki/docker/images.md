---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 4. Docker 镜像管理
order: 3
---

## 4. Docker 镜像管理

镜像是 Docker 的三大组件之一。

Docker 运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker 会从镜像仓库下载（默认是 Docker Hub 公共注册服务器中的仓库），我们也可以搭建一个本地的镜像仓库，但这不是本文的重点。本文将以镜像为中心介绍：

- 如何构建基础镜像
- Dockerfile的基本结构以及详解
- 利用Dockerfile构建镜像



### 4.1 构建基础镜像

我将应用打包到镜像中形成我们所需的镜像，往往需要一个基础的镜像作为我们应用服务的外部环境，那么问题来了，基础镜像从何而来？官方推荐的是直接从官网仓库pull一个，但由于官网被墙的比较厉害，所以这里介绍一些官方提供以及个人方法。

1.使用Debootstrap来创建Ubuntu的base image

```
$ sudo debootstrap raring raring > /dev/null
$ sudo tar -C raring -c . | docker import - raring
a29c15f1bf7a
$ docker run raring cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=13.04
DISTRIB_CODENAME=raring
DISTRIB_DESCRIPTION="Ubuntu 13.04"
```

在docker github上有更多有关基础镜像的介绍

- BusyBox
- CentOS / Scientific Linux CERN (SLC) on Debian/Ubuntu or on CentOS/RHEL/SLC/etc.
- Debian / Ubuntu

2.使用scratch创建base image 在Docker registry中有一个scratch，你可以pull拉取下来，

```
$ sudo docker pull scratch
```

甚至可以自己制作

```
$ tar cv --files-from /dev/null | docker import - scratch
```

Scratch镜像很赞，它简洁、小巧而且快速， 它没有bug、安全漏洞、延缓的代码或技术债务。这是因为它基本上是空的。除了Docker添加了点的metadata (译注：元数据为描述数据的数据)。总之它是非常小的一个Docker镜像。 为Scratch镜像创建内容，具体Dockerfile命令如下:

```
FROM scratch
ADD hello /
CMD ["/hello"]
```

3.下载官方提提供的OS的tar文件 到[OPENVZ](http://wiki.openvz.org/Download/template/precreated)上下载基础包然后使用docker limport 加载到本地镜像，这里以ubuntu14.04 为例，从openvz下载一个ubuntu14.04的模板：

```
wget http://download.openvz.org/template/precreated/ubuntu-14.04-x86_64.tar.gz
cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:base
```



### 4.2 Dockerfile文件结构

Dockerfile 由一行行命令语句组成，并且支持以 # 开头的注释行。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

例如:

```
# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..

# Base image to use, this must be set as the first line
FROM ubuntu

# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER docker_user [email protected]

# Commands to update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

# Commands when creating a new container
CMD /usr/sbin/nginx
```

其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。

后面则是镜像操作指令，例如 RUN 指令，RUN 指令将对镜像执行跟随的命令。每运行一条 RUN 指令，镜像添加新的一层，并提交。

最后是 CMD 指令，来指定运行容器时的操作命令。

下面是一个更复杂的例子

```
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
MAINTAINER Victor Vieux <[email protected]>

RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server

# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir /.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]

# Multiple images example
#
# VERSION               0.1

FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```



### 4.3 Dockerfile 操作建议

Docker可以读取一个Dockerfile文件来构建所需的镜像，这个文件里包含所有所需要的指令。Dockerfile文件用特有的格式来设置镜像信息，更多基础知识在 Dockerfile 参数详解 会详细展示。

本文包含Docker官方提供的一些践以及方法，我们强烈建议你去参照这些建议。

官方建议：

1. 一个Dockerfile文件尽量`越简洁越好`，这意味着 它可以被停止销毁，然后被最小化配置安装到另一个地方。

2. 在通常情况下，最好把`Dockerfile文件放到一个空目录`，然后，将构建镜像所需要动文件添加到该目录。为了提高构建性能，你也可以通过添加`.dockerignore`文件到该目录以排除文件和目录，该文件支持排斥的模式类似于.gitignore文件。

3. 为了`减少镜像复杂度、依赖、文件大小和构建时间`，应该尽量避免安装多余的不需要包，例如：你不需要在一个数据库镜像中添加一个文本编辑器。

4. 在绝大多数情况下，`每一个镜像只跑一个process`，应用于多个容器中可以方边应用横向扩展和重复利用容器。如果该服务依赖于其他服务，请使用容器互联。

5. 你需要在Dockerfile的可读性和镜像层次最小化之间取得平衡，要有目的且非常谨慎的控制使用分层的数量

6. 尽可能缓解由字母数字排序的多行参数后的变化。这将帮助你避免包的重复，使列表更容易更新。这也使得PRs更容易审查。在一个空格前面加一个反斜杠能起到帮助。

   下面是来自buildpack-DEPS镜像中的例子：

   ```
    RUN apt-get update && apt-get install -y \
    bzr \
    cvs \
    git \
    mercurial \
    subversion
   ```

7. 构建缓存

   在构建镜像时，进程将为`Dockerfile内每一个指定的执行步骤构建一个镜像` 。由于执行每条指令都会对它缓存内现有镜像进行检查，所以`镜像可以重复利用` ，而不是创建一个重复的镜像。如果你不想使用缓存，请在docker build 时使用`--no-cach=true` 选项。

   但是，如果你让Docker使用构建缓存找到匹配的镜像，你应该了解它什么时候需要，什么时候不需要，所以要遵循以下规则：

   从缓存中启动一个基础镜像，执行下一条指令，比与上一层所有子镜像对比，`查看是否有与已经存在的镜像相同` ，如果不同，缓存将失效。

   在大多数情况下，只需要简单地比较Dockerfile指令与其中一个子镜像就够了，但是，某些指令需要更多解释：

   对于ADD和COPY指令，镜像中的文件`每个的内容将全部检查` 。文件的某些信息是不检查的：最近更新时间和最后访问时间。在查找缓存期间，Docker会与已经存在的镜像文件进行校验，`如果文件被更改，例如内容或元数据，那么缓存也将失效` 。除此之外，ADD和COPY指令缓存将不会查看容器内文件来匹配缓存。例如，当执行一个 RUN apt-get -y 指令时，不会检查容器内文件更新来确定缓存是否存在。在这种情况下，将使用指令字符串本身来查找缓存匹配。

   上文所提到动缓存失效，是指`后续指令将会产生新的镜像文件，缓存将不会被使用` 。

#### 4.3.1 镜像文件的大小

**Dockerfile 与镜像**

Dockerfile 由多条指令构成，随着深入研究 Dockerfile 与镜像的关系，很快大家就会发现，Dockerfile 中的`每一条指令都会对应于 Docker 镜像中的一层` 。

以如下 Dockerfile 为例：

```
FROM ubuntu:14.04
ADD run.sh /
VOLUME /data
CMD ["./run.sh"]
```

通过 docker build 以上 Dockerfile 的时候，会在 ubuntu:14.04 镜像基础上，添加三层独立的镜像，依次对应于三条不同的命令。镜像示意图如下：

![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/dockerfile.jpg)

有了 Dockerfile 与镜像关系的初步认识之后，我们再进一步联系到每一层镜像的大小。

不得不说，在层级化管理的 Docker 镜像中，有不少层大小都为 0。那些镜像层大小不为 0 的情况，归根结底的原因是：`构建 Docker 镜像时，对当前的文件系统造成了修改更新` 。而修改更新的情况主要有两种：

1. ADD或 COPY 命令：ADD 或者 COPY 的作用是在`docker build 构建镜像时向容器中添加内容` ，只要内容添加成功，`当前构建的那层镜像就是添加内容的大小` ，如以上命令 ADD run.sh /，新构建的那层镜像大小为文件 run.sh 的大小。
2. RUN 命令`：RUN 命令的作用是在当前空的镜像层内运行一条命令` ，倘若运行的命令需要更新磁盘文件，那么所有的更新内容都在存储在当前镜像层中。举例说明：`RUN echo Hello world 命令不涉及文件系统内容的修改` ，故命令运行完之后当前镜像层的`大小为 0` ；`RUN wget http://abc.com/def.tar` 命令会将压缩包下载至当前目录下，因此当前这一层镜像的大小为：对文件系统内容的增量修改部分，即 `def.tar 文件的大小` （在成功执行的情况下）。

**联合文件系统**

Dockerfile 中`命令与镜像层一一对应` ，那么是否意味着 docker build 完毕之后，镜像的`总大小是否等于每一层镜像的大小总和` 呢？答案是`肯定的` 。依然以上图为例：如果 ubuntu:14.04 镜像的大小为 200 MB，而 run.sh 的大小为 5 MB，那么以上三层镜像从上到下，每层大小依次为 0、0 以及 5 MB，那么最终构建出的镜像大小的确为 0 + 0 + 5 + 200 = 205 MB。

虽然`最终镜像的大小是每层镜像的累加` ，但是需要额外注意的是：Docker `镜像的大小并不等于容器中文件系统内容的大小` （不包括挂载文件，/proc、/sys 等虚拟文件）。个中缘由，就和联合文件系统有很大的关系了。

首先来看一下这个简单的 Dockerfile 例子（假如在 Dockerfile 当前目录下有一个 100 MB 的压缩文件 compressed.tar）：

```
FROM ubuntu:14.04
ADD compressed.tar /
RUN rm /compressed.tar
ADD compressed.tar /
```

1. FROM ubuntu:14.04：镜像 ubuntu:14.04 的`大小为 200 MB` ；
2. ADD compressed.tar /： compressed.tar 文件为 100 MB，因此`当前镜像层的大小为 100 MB` ，镜像`总大小为 300 MB` ；
3. RUN rm /compressed.tar：删除文件 compressed.tar，此时的删`除并不会删除下一层的 compressed.tar 文件` ，只会在当前层`产生一个 compressed.tar 的删除标记` ，确保通过该层将看不到 compressed.tar，因此`当前镜像层的大小也为 0` ，镜像`总大小为 300 MB` ；
4. ADD compressed.tar /：compressed.tar 文件为 100 MB，因此`当前镜像层的大小为 300 MB + 100 MB` ，镜像`总大小为 400 MB` ；

分析完毕之后，我们发现镜像的总大小为 400 MB，但是如果`运行该镜像` 的话，我们很快可以发现在容器根目录下执行 du -sh 之后，显示的数值并非 400 MB，而是 `300 MB 左右` 。主要的原因还是：联合文件系统的性质保`证了两个拥有 compressed.tar 文件的镜像层，容器仅能看到一个` 。同时这也说明了一个现状，当用户基于一个非常大，甚至好几个 GB 的镜像运行容器时，在容器内部查看根目录大小，发现竟然只有 500 MB 不到，甚至更小。

分析至此，有一点大家需要非常注意：`镜像大小和容器大小有着本质的区别` 。

**镜像共享关系**

Docker 镜像说大不大，说小不小，但是一旦`镜像的总数上来` 之后，岂不是对本地磁盘造成很大的存储压力？平均每个镜像 500 MB，岂不是 100 个镜像就需要准备 50 GB 的存储空间？

结果往往不是我们想象的那样，Docker 在`镜像复用` 方面设计得非常出色，大大节省镜像占用的磁盘空间。Docker 镜像的复用主要体现在：`多个不同的 Docker 镜像可以共享相同的镜像层` 。

假设本地镜像存储中只有一个 ubuntu:14.04 的镜像，我们以两个 Dockerfile 来说明镜像复用：

```
FROM ubuntu:14.04
RUN apt-get update
FROM ubuntu:14.04
ADD compressed.tar /
```

假设最终 docker build 构建出来的镜像名分别为 image1 和 image2，由于两个 Dockerfile 均`基于 ubuntu:14.04` ，因此，image1 和 image2 这两个镜像均`复用了镜像 ubuntu:14.04` 。 假设 `RUN apt-get update` 修改的文件系统内容为 20 MB，最终本地三个镜像的大小关系应该如下：

```
ubuntu:14.04： 200 MB
image1：200 MB（ubuntu:14.04 的大小）+ 20 MB = 220 MB
image2：200 MB（ubuntu:14.04 的大小）+ 100 MB = 300 MB
```

如果仅仅是单纯的累加三个镜像的大小，那结果应该是：`200 + 220 + 300 = 720 MB` ，但是由于镜像复用的存在，实际占用的磁盘空间大小是`：200 + 20 + 100 + 320 MB` ，足足节省了 400 MB 的磁盘空间。在此，足以证明镜像复用的巨大好处。



### 4.4 Dockerfile 参数详解

指令的一般格式为 `INSTRUCTION arguments` ，指令包括 `FROM、MAINTAINER、RUN` 等。

#### FROM

```
格式为 FROM <image>或FROM <image>:<tag>。
```

第一条指令必须为`FROM` 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个`FROM` 指令（每个镜像一次）。

#### MAINTAINER

```
格式为 MAINTAINER <name>，指定维护者信息。
```

#### RUN

```
格式为 RUN <command> 或 RUN ["executable", "param1", "param2"]。
前者将在 shell 终端中运行命令` ，即 /bin/sh -c；`后者则使用 exec 执行` 。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]。
```

每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

#### CMD

支持三种格式

```
CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；
```

指定启动容器时执行的命令，每个 Dockerfile 只能有一条`CMD` 命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

#### EXPOSE

格式为

```
EXPOSE <port> [<port>...]。
```

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 `-P` ，Docker 主机会自动分配一个端口转发到指定的端口。

#### ENV

格式为

```
ENV <key> <value>。
```

指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

例如

```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

#### ADD

格式为

```
ADD <src> <dest>。
```

该命令将复制指定的 到容器中的 。 其中 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

#### COPY

格式为

```
COPY <src> <dest>。
```

复制本地主机的 （为 Dockerfile 所在目录的相对路径）到容器中的 。

当使用本地目录为源目录时，推荐使用 COPY。

#### ENTRYPOINT

两种格式：

```
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2（shell中执行）。
```

配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。

每个 Dockerfile 中只能有一个 `ENTRYPOINT` ，当指定多个时，只有最后一个起效。

#### VOLUME

格式为

```
VOLUME ["/data"]。
```

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

#### USER

格式为

```
USER daemon。
```

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres` 。`要临时获取管理员权限可以使用 gosu，而不推荐 sudo` 。

#### WORKDIR

格式为

```
WORKDIR /path/to/workdir。
```

为后续的 `RUN、CMD、ENTRYPOINT` 指令配置工作目录。

可以使用多个`WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

则最终路径为 `/a/b/c` 。

#### ONBUILD

格式为

```
ONBUILD [INSTRUCTION]。
```

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile 使用如下的内容创建了镜像 image-A。

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

如果基于 image-A 创建新的镜像时，新的Dockerfile中使用 FROM image-A指定基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

```
FROM image-A
#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
```

使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 `ruby:1.9-onbuild` 。



### 4.5 创建镜像

编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。

基本的格式为 docker build [选项] 路径，该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。也可以通过 .dockerignore 文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 -t 选项，例如

```
$ sudo docker build -t myrepo/myapp /tmp/test1/
```
