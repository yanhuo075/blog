---
layout: wiki  # 使用wiki布局模板
wiki: git # 这是项目id，对应 /_data/wiki/git.yml
title: 大文件托管
order: 3
---

# 如何使用Git LFS?

本文主要介绍了为什么要选择使用Git LFS，进而说明了在Codeup中如何使用Git LFS，以及在使用Git LFS时常见问题的处理方式和Git LFS的工作原理。

## **为什么选择使用Git LFS？**

Git LFS为了解决大文件托管的效率问题，提供了五大特性：

- **更大**：支持GB级别的大文件版本控制。
- **更小**：让Git仓库空间占用减小。
- **更快**：仓库的克隆和拉取更快。
- **透明**：Git使用上对用户完全透明。
- **兼容**：权限控制上完全兼容（兼容Codeup权限控制）。

了解更多：[什么是Git LFS大文件存储?](https://help.aliyun.com/zh/yunxiao/user-guide/what-is-git-lfs-large-file-storage#topic-2042154)

## **下载和安装Git LFS**

- 下载：

  - Linux Debian和RPM packages：https://packagecloud.io/github/git-lfs/install。
  - Mac：使用`brew install git-lfs`。
  - Windows：目前lfs已经集成在了[Git for Windows ](https://gitforwindows.org/)中，直接下载和使用最新版本的Windows Git即可。
  - 直接下载二进制包：https://github.com/git-lfs/git-lfs/releases。
  - 依据源码构建：https://github.com/git-lfs/git-lfs。

- 安装：如果您选择使用二进制包下载后安装，直接执行解压后的`./install.sh`脚本即可，这个脚本会做两件事情：

  - 在$PATH中安装Git LFS的二进制可执行文件。

  - 执行`git lfs install`命令，让当前环境支持全局的LFS配置。

     

    ```shell
    # 让仓库支持LFS
    $ git lfs install
    Updated pre-push hook.
    Git LFS initialized.
    ```

## **让本地新仓库支持Git LFS**

以下将以一个通用的场景进行实际的演示和说明（环境为Linux/macOS ）。

### **步骤一：创建一个新的Git空仓库**

1. 在Codeup上创建一个空白的新仓库，名为“git-lfs”，如何创建，请参见[步骤一：新建第一个代码库](https://help.aliyun.com/zh/yunxiao/user-guide/novice-guide-1/#df69fbe063gsr)。

2. 将该仓库克隆到本地，并进入该目录：

    

   ```sh
   # 请注意替换您实际的代码库地址
   $git clone https://codeup.aliyun.com/您的组织分组/git-lfs.git
   Cloning into 'git-lfs'...
   warning: You appear to have cloned an empty repository.
   $cd git-lfs
   $tree .git/hooks/
   .git/hooks/
   ├── applypatch-msg.sample
   ├── commit-msg.sample
   ├── execute-commands.sample
   ├── post-receive.sample
   ├── post-update.sample
   ├── pre-applypatch.sample
   ├── pre-commit.sample
   ├── prepare-commit-msg.sample
   ├── pre-push.sample
   ├── pre-rebase.sample
   ├── pre-receive.sample
   └── update.sample
   
   0 directories, 12 files
   # 此时Git LFS相关的Hook还未替换
   ```

### **步骤二：配置Git LFS**

1. 为了将以示例`.bigfile`后缀结尾的文件使用Git LFS进行存储，需要执行track命令建立追踪：

    

   ```sh
   $git lfs track "*.bigfile"
   Tracking "*.bigfile"
   ```

   **重要**

   使用 lfs track 命令时，"*.bigfile"的双引号非常重要，否则将影响pattern的文件匹配功能。

   同理，如需跟踪其他后缀的文件，如.jpg，可以写为`git lfs track "*.jpg"`。

2. 执行`git lfs track`（不带任何参数），可以查看当前已跟踪的Git LFS File 类型：

    

   ```sh
   $git lfs track
   Listing tracked patterns
       *.bigfile (.gitattributes)
   Listing excluded patterns
   ```

3. track 命令实际上是修改了仓库中的`.gitattributes`文件，将该文件add添加到暂存区。

    

   ```sh
   $git add .gitattributes 
   ```

4. 可以通过以下命令查看文件相关变动：

    

   ```sh
   $git diff --cached
    diff --git a/.gitattributes b/.gitattributes
    new file mode 100644
    index 0000000..c441ad2
    --- /dev/null
    +++ b/.gitattributes
    @@ -0,0 +1 @@
    +*.bigfile filter=lfs diff=lfs merge=lfs -text
   ```

### **步骤三：让Git LFS配置生效**

为了让"***.bigfile"的配置生效，需要将`.gitattributes`文件进行提交：

 

```sh
$git commit -m "Add \"*.bigfile\" LFS config "
[master (root-commit) d052478] Add "*.bigfile" LFS config
 1 file changed, 1 insertion(+)
 create mode 100644 .gitattributes
$git log --oneline
d052478 (HEAD -> master) Add "*.bigfile" LFS config
```

### **步骤四：新建一个.bigfile文件进行测试**

接下来，在工作空间创建一个名为**dyrone.bigfile**的文件，大小为1GB*：*

 

```sh
# mac环境可以使用mkfile命令替换dd命令 
$dd if=/dev/zero of=dyrone.bigfile bs=1G count=1
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 2.41392 s, 445 MB/s
$du -sh dyrone.bigfile 
1.1G    dyrone.bigfile
```

将**dyrone.bigfile**添加到暂存区：

 

```sh
$git add dyrone.bigfile
```

由于**dyrone.bigfile**后缀命中了`.gitattributes`中设置的`"*.bigfile"`的文件格式，所以将做为 LFS 文件处理。

**说明**

1. **什么是Git LFS Pointer**

Git LFS并不会将完整的文件属性和内容进行计算写入INDEX（暂存区的实际文件），而是会生成一个Git LFS Pointer文件，该文件指向了真正的LFS存储对象名称。可以通过执行`git diff --cached`命令查看仓库INDEX和HEAD之间的差异，也就是**dyrone.bigfile**对应的**Pointer**文件的内容：

 

```sh
$git diff --cached
diff --git a/dyrone.bigfile b/dyrone.bigfile
new file mode 100644
index 0000000..9d7c19f
--- /dev/null
+++ b/dyrone.bigfile
@@ -0,0 +1,3 @@
+version https://git-lfs.github.com/spec/v1
+oid sha256:49bc20df15e412a64472421e13fe86ff1c5165e18b2afccf160d4dc19fe68a14
+size 1073741824
```

**Pointer**文件内容所表达的含义：

- version [https://git-lfs.github.com/spec/v1 ](https://git-lfs.github.com/spec/v1)：代表git-lfs协议的版本。
  - version...v1：代表当前git-lfs服务端遵从的协议版本。
  - git-lfs.github.com：此域名地址为git-lfs开源项目的官方地址。
- oid sha256:49bc20df15e412a64472421e13fe86ff1c5165e18b2afccf160d4dc19fe68a14：
  - oid：代表Git LFS object id。
  - sha256：64位16进制，其代表真实文件的名称，名称通过sha256生成，唯一值。
- size: 代表文件实际大小，单位：byte。

1. **本地的Git LFS文件是如何存储的？**

在**Pointer**文件被添加到暂存区的同时，真正的`1GB`的大文件**dyrone.bigfile**被存储在仓库的LFS缓存目录下，名称被修改为**Pointer文件**中所指向的`oid`字符串名称：

 

```sh
$tree .git/lfs
.git/lfs
├── objects
│    └── 49
│         └── bc
│             └── 49bc20df15e412a64472421e13fe86ff1c5165e18b2afccf160d4dc19fe68a14
└── tmp
4 directories, 1 file
```

在本地仓库中，Git LFS是按照这种方式来进行大文件实际存储，原本的**dyrone.bigfile**被表示成了两个文件，**Pointer文件**的blob是对应Commit真正引用的对象，而实际LFS文件被缓存在.git/lfs目录。而此时工作区中的文件没有发生任何变化。

### **步骤五：推送文件到远端**

接下来，将**dyrone.bigfile**的变更提交并推送到远端：

 

```sh
$git commit -m "Add a really big file"
[master 8032589] Add a really big file
 1 file changed, 3 insertions(+)
 create mode 100644 dyrone.bigfile
```

其中，"1 file changed, 3 insertions(+)" 表示**Pointer文件**已经提交，可以执行`git show HEAD`查看提交详情：

 

```sh
$git show HEAD
commit 8032589f47a748171e84da94ce6440fe139e99f9 (HEAD -> master)
Author: dyroneteng <tenglong***@alibaba-inc.com>
Date:   Tue Sep 15 17:25:58 2020 +0800

    Add a really big file

diff --git a/dyrone.bigfile b/dyrone.bigfile
new file mode 100644
index 0000000..9d7c19f
--- /dev/null
+++ b/dyrone.bigfile
@@ -0,0 +1,3 @@
+version https://git-lfs.github.com/spec/v1
+oid sha256:49bc20df15e412a64472421e13fe86ff1c5165e18b2afccf160d4dc19fe68a14
+size 1073741824
```

接着，将提交推送到远端Codeup已准备好的空仓库：

 

```sh
$git push
Uploading LFS objects:   0% (0/1), 3.9 MB | 0 B/s                                                                                                                                                               
Uploading LFS objects:   0% (0/1), 79 MB | 30 MB/s                                                                                                                                                              
Uploading LFS objects:   0% (0/1), 207 MB | 50 MB/s                                                                                                                                                             
Uploading LFS objects:   0% (0/1), 326 MB | 51 MB/s                                                                                                                                                             
Uploading LFS objects:   0% (0/1), 534 MB | 56 MB/s                                                                                                                                                             
Uploading LFS objects: 100% (1/1), 1.1 GB | 58 MB/s, done.                                                                                                                                                      
Counting objects: 3, done.
Delta compression using up to 32 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 410 bytes | 410.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://codeup.aliyun.com/xxxxx/git-lfs.git
   d052478..8032589  master -> master
```

如果存在LFS文件需要上传，在推送过程中将会显示LFS上传进度。

至此，这个仓库中**.bigfile**的文件已经成功使用LFS进行管理，而其他文件使用Git进行管理。

**说明**

简要解读此过程的标准输出：

- 如果推送的文件中，包含Git LFS文件，那么实际上push将被分为两个部分：
  - 第一部分： Git LFS oid文件上传。
  - 第二部分： 当LFS oid文件上传结束后，继续使用Git协议上传Pointer文件的相关对象。
- 第一部分 "Uploading LFS objects: 100% (1/1), 1.1 GB | 58 MB/s, done." ， 会显示*Git LFS oid文件*上传过程中的进度信息，如总上传个数、当前上传、oid文件大小和上传速度等。
- 第二部分 "Writing objects: 100% (3/3), 410 bytes | 410.00 KiB/s, done." ，例子共三个对象：blob(1)、tree(1)、commit(1)。
- 两个部分，任一部分失败，则整个推送将失败。

除了过程的标注输出增多的变化以外，也侧面体现了Git LFS是如何进行推送的：在推送时，Git LFS文件被单独上传到LFS 服务器上，而Pointer文件保持不变推送到Git 服务器上，可以观察仓库目录大小来验证这一点。

可以发现，本地仓库大小仅仅为`188K`（排除LFS缓存目录），这也基本是远端Git仓库的大小，也达到了Git LFS瘦身仓库的目的。

 

```shell
$du -sh --exclude=lfs .git 
188K    .git

$du -sh .git
1.1G    .git
```

## **克隆已使用Git LFS的仓库**

**本地需预先已安装Git LFS工具**，否则克隆后的LFS文件将以指针而非期望的原始文件呈现。当克隆一个已经使用的Git LFS仓库时，`git-lfs`会自动替换仓库中相关hook，进而让Git LFS生效。

 

```sh
# 样例仓库，如尝试，可替换为实际已经 LFS 的仓库地址
$ git clone gi*@codeup.aliyun.com:您的组织分组/dyrone.git
Cloning into 'dyrone'...
remote: Counting objects: 41, done.
remote: Total 41 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (41/41), 2.98 MiB | 5.01 MiB/s, done.
Filtering content: 100% (3/3), 5.01 MiB | 3.83 MiB/s, done.
# Filtering content表示正在将下载下来的LFS文件smudge为实际工作区的文件
$ cd dyrone
$ tree .git/hooks
.git/hooks
├── applypatch-msg.sample
├── commit-msg.sample
├── execute-commands.sample
├── fsmonitor-watchman.sample
├── post-checkout
├── post-commit
├── post-merge
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample
├── pre-merge-commit.sample
├── pre-push
├── pre-push.sample
├── pre-rebase.sample
├── pre-receive.sample
├── prepare-commit-msg.sample
└── update.sample

0 directories, 17 files
```

可以看到，例如pre-push等Git LFS需要用到的相关Hook已经被替换，进而支持Git Flow，对用户Git操作保持兼容和透明。

## **如何将历史文件转换为LFS管理**

更多复杂场景迁移指南，参见[LFS 迁移指南](https://help.aliyun.com/zh/yunxiao/user-guide/lfs-migration-guide)。

## **如何撤销LFS跟踪并使用Git管理**

您可以取消继续跟踪某类文件，并将其从cache中清理：

 

```sh
git lfs untrack "*.bigfile"
git rm --cached "*.bigfile"
```

如果您想将这些文件添加回常规Git跟踪，可以执行以下操作：

 

```sh
git add "*.bigfile"
git commit -m "restore "*.bigfile" to git from lfs"
```

## **Git LFS工作原理**

在前面的一些示例之后，相信大家对Git LFS有了一定的理解，其工作原理如下：

Git LFS场景

Git场景

![Git 2](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/p240000.png)

如图所示，可以针对jpg格式的图片利用Git LFS的存储功能，在推送过程中将其上传至大文件存储服务。同时，与大文件对应的指针文件将与其他普通代码文件一并推送到远端Git仓库中。

**Git LFS处理流程介绍**

![Git 3](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/p240002.png)

## **Git LFS的限制**

- Windows平台，单个文件不支持超过`4G`，[issues 2434](https://github.com/git-lfs/git-lfs/issues/2434)。
- Windows用户必须保证已经安装`Git Credential Manager`，否则可能导致操作被无限挂起，[issues 1763](https://github.com/git-lfs/git-lfs/issues/1763)。
- 不同于Gitlab硬编码的LFS下载token超时时间（30分钟），Codeup会根据将要下载的文件列表动态计算token超时时间，但是如果位于网络环境不好的环境，仍旧可能导致token超时的情况。如果需要根据需求调整，可以联系Codeup系统管理员处理。
