---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.1 使用 Ansible 命令行工具
order: 0
---

# 使用 Ansible 命令行工具

欢迎来到 Ansible 命令行工具使用指南。Ansible 提供了 ad hoc（临时命令） 和多个实用程序，用于执行各种操作和自动化任务。



## 临时命令简介

Ansible 临时命令使用 /usr/bin/ansible 命令行工具在一个或多个受管节点上自动化单个任务。临时命令快速且容易，但它们不可重用。那么为什么要学习临时命令呢？临时命令展示了 Ansible 的简单性和强大功能。您在这里学到的概念将直接移植到 Playbook 语言。在阅读和执行这些示例之前，请阅读 如何构建清单。

- 为什么使用临时命令？
- 临时任务的使用场景
  - 重启服务器
  - 管理文件
  - 管理软件包
  - 管理用户和组
  - 管理服务
  - 收集事实
  - 检查模式
  - 模式和临时命令



## 为什么使用临时命令？

临时命令非常适合不经常重复的任务。例如，如果您想在圣诞假期关闭实验室中的所有机器，您可以在 Ansible 中执行一个快速的单行命令，而无需编写 Playbook。临时命令看起来像这样

```
$ ansible [pattern] -m [module] -a "[module options]"
```



-a 选项通过 key=value 语法或以 { 开头并以 } 结尾的 JSON 字符串接受选项，以用于更复杂的选项结构。您可以在其他页面上了解有关 模式 和 模块 的更多信息。

## 临时任务的使用场景

临时任务可用于重启服务器、复制文件、管理软件包和用户等等。您可以在临时任务中使用任何 Ansible 模块。像 Playbook 一样，临时任务使用声明式模型，计算并执行达到指定最终状态所需的操作。它们通过在开始之前检查当前状态，并且除非当前状态与指定的最终状态不同，否则不执行任何操作，从而实现某种形式的幂等性。



### 重启服务器

ansible 命令行实用程序的默认模块是 ansible.builtin.command 模块。您可以使用临时任务调用命令模块，并一次重启亚特兰大所有 Web 服务器的 10 台。在 Ansible 可以执行此操作之前，您必须在清单中有一个名为 [atlanta] 的组中列出亚特兰大的所有服务器，并且您必须拥有该组中每台机器的有效 SSH 凭据。要重启 [atlanta] 组中的所有服务器

```
$ ansible atlanta -a "/sbin/reboot"
```



默认情况下，Ansible 仅使用五个并发进程。如果您拥有的主机多于为 fork 计数设置的值，则可能会增加 Ansible 与主机通信所需的时间。要以 10 个并行 fork 重启 [atlanta] 服务器

```
$ ansible atlanta -a "/sbin/reboot" -f 10
```



/usr/bin/ansible 将默认以您的用户帐户运行。要以不同的用户身份连接

```
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username
```



重启可能需要特权提升。您可以使用 become 关键字以 username 连接到服务器并以 root 用户身份运行命令

```
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]
```



如果添加 --ask-become-pass 或 -K，Ansible 会提示您输入用于特权提升（sudo/su/pfexec/doas/etc）的密码。

到目前为止，我们所有的示例都使用了默认的“command”模块。要使用其他模块，请传递 -m 作为模块名称。例如，要使用 ansible.builtin.shell 模块

```
$ ansible raleigh -m ansible.builtin.shell -a 'echo $TERM'
```



在使用 Ansible ad hoc CLI（与 Playbook 相反）运行任何命令时，请特别注意 shell 引号规则，以便本地 shell 保留变量并将其传递给 Ansible。例如，在上面的示例中使用双引号而不是单引号将在您所在的框上评估变量。



### 管理文件

临时任务可以利用 Ansible 和 SCP 的强大功能，并行地将多个文件传输到多台机器。要直接将文件传输到 [atlanta] 组中的所有服务器

```
$ ansible atlanta -m ansible.builtin.copy -a "src=/etc/hosts dest=/tmp/hosts"
```



如果您计划重复执行类似的任务，请在 Playbook 中使用 ansible.builtin.template 模块。

ansible.builtin.file 模块允许更改文件的所有权和权限。这些相同的选项也可以直接传递给 copy 模块

```
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/a.txt mode=600"
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
```



```
file` 模块还可以创建目录，类似于 `mkdir -p
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
```



以及删除目录（递归地）和删除文件

```
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c state=absent"
```





### 管理软件包

您还可以使用临时任务，使用诸如 yum 之类的软件包管理模块在受管节点上安装、更新或删除软件包。软件包管理模块支持安装、删除和常规管理软件包的常用功能。软件包管理器的某些特定功能可能不存在于 Ansible 模块中，因为它们不属于常规软件包管理。

确保安装软件包而不更新它

```
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=present"
```



确保安装特定版本的软件包

```
$ ansible webservers -m ansible.builtin.yum -a "name=acme-1.5 state=present"
```



确保软件包处于最新版本

```
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=latest"
```



确保未安装软件包

```
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=absent"
```



Ansible 具有用于管理多种平台下软件包的模块。如果没有适用于您的软件包管理器的模块，则可以使用 command 模块安装软件包或为您的软件包管理器创建模块。



### 管理用户和组

您可以使用临时任务在受管节点上创建、管理和删除用户帐户

```
$ ansible all -m ansible.builtin.user -a "name=foo password=<encrypted password here>"

$ ansible all -m ansible.builtin.user -a "name=foo state=absent"
```

有关所有可用选项的详细信息，包括如何操作组和组成员身份，请参阅 ansible.builtin.user 模块文档。



### 管理服务

确保在所有 Web 服务器上启动服务

```
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=started"
```



或者，在所有 Web 服务器上重新启动服务

```
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=restarted"
```



确保服务已停止

```
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=stopped"
```



### 收集事实

事实表示有关系统的已发现变量。您可以使用事实来实现任务的条件执行，也可以仅获取有关系统的临时信息。要查看所有事实

```
$ ansible all -m ansible.builtin.setup
```



您还可以筛选此输出以仅显示某些事实，有关详细信息，请参阅 ansible.builtin.setup 模块文档。

### 检查模式

在检查模式下，Ansible 不会对远程系统进行任何更改。Ansible 仅打印命令。它不会运行命令。

```
$  ansible all -m copy -a "content=foo dest=/root/bar.txt" -C
```



在上面的命令中启用检查模式（-C 或 --check）意味着 Ansible 实际上不会在任何远程系统上创建或更新 /root/bar.txt 文件。



### 模式和临时命令

有关所有可用选项的详细信息，包括如何使用临时命令中的模式进行限制，请参阅 模式 文档。

现在您已经了解了 Ansible 执行的基本要素，您已准备好学习使用 Ansible Playbook 来自动化重复性任务。



## Ansible CLI 速查表

接下来显示每个 Ansible 命令行实用程序的一个或多个示例，其中添加了一些常用标志，以及指向该命令完整文档的链接。此页面仅提供一些常见用例的快速提醒 - 它可能已过时或不完整，或两者兼有。对于规范文档，请点击 CLI 页面的链接。
