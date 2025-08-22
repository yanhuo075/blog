---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 3.3 清单远程连接方式
order: 2
---

# 清单远程连接方式

本节向您展示如何扩展和改进 Ansible 用于您的清单的连接方法。

## ControlPersist 和 paramiko

默认情况下，Ansible 使用原生的 OpenSSH，因为它支持 ControlPersist（一种性能特性）、Kerberos 和 ~/.ssh/config 中的选项，例如 Jump Host 设置。如果您的控制机器使用不支持 ControlPersist 的旧版本 OpenSSH，Ansible 将回退到名为“paramiko”的 OpenSSH 的 Python 实现。



## 设置远程用户

默认情况下，Ansible 使用您在控制节点上使用的用户名连接到所有远程设备。如果该用户名在远程设备上不存在，您可以为连接设置不同的用户名。如果您只需要以不同的用户身份执行一些任务，请查看 理解权限提升：become。您可以在 Playbook 中设置连接用户

```
---
- name: update webservers
  hosts: webservers
  remote_user: admin

  tasks:
  - name: thing to do first in this playbook
  . . .
```



作为清单中的主机变量

```
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```



或作为清单中的组变量

```
cloud:
  hosts:
    cloud1: my_backup.cloud.com
    cloud2: my_backup2.cloud.com
  vars:
    ansible_user: admin
```



## 设置 SSH 密钥

默认情况下，Ansible 假设您正在使用 SSH 密钥连接到远程计算机。建议使用 SSH 密钥，但如果需要，您可以使用 --ask-pass 选项使用密码验证。如果您需要为 权限提升（sudo、pbrun 等）提供密码，请使用 --ask-become-pass。

要设置 SSH agent 以避免重新输入密码，您可以执行以下操作

```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
```



根据您的设置，您可能希望使用 Ansible 的 --private-key 命令行选项来指定 pem 文件。您还可以添加私钥文件

```
$ ssh-agent bash
$ ssh-add ~/.ssh/keypair.pem
```



在不使用 ssh-agent 的情况下添加私钥文件的另一种方法是在清单文件中使用 ansible_ssh_private_key_file，如此处解释的：如何构建您的清单。

## 针对 localhost 运行

您可以通过使用“localhost”或“127.0.0.1”作为服务器名称来对控制节点运行命令

```
$ ansible localhost -m ping -e 'ansible_python_interpreter="/usr/bin/env python"'
```



您可以通过将其添加到您的清单文件中来显式指定 localhost

```
localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python"
```



## 管理主机密钥检查

默认情况下，Ansible 启用主机密钥检查。检查主机密钥可以防止服务器欺骗和中间人攻击，但这确实需要一些维护。

如果重新安装主机并且在“known_hosts”中具有不同的密钥，这将导致错误消息，直到更正为止。如果新主机不在“known_hosts”中，您的控制节点可能会提示确认密钥，如果在 Ansible 中使用，例如从 cron 中使用，这将导致交互式体验。您可能不希望这样。

如果您了解其含义并希望禁用此行为，您可以通过编辑 /etc/ansible/ansible.cfg 或 ~/.ansible.cfg 来执行此操作

```
[defaults]
host_key_checking = False
```



或者，可以通过 ANSIBLE_HOST_KEY_CHECKING 环境变量来设置此项

```
$ export ANSIBLE_HOST_KEY_CHECKING=False
```



另请注意，在 paramiko 模式下进行主机密钥检查相当慢，因此，当使用此功能时，也建议切换到“ssh”。

## 其他连接方法

Ansible 可以使用除 SSH 之外的多种连接方法。您可以选择任何连接插件，包括在本地管理事物以及管理 chroot、lxc 和 jail 容器。一种名为“ansible-pull”的模式也可以反转系统，并让系统通过计划的 Git 检出“电话回家”以从中央存储库中提取配置指令。
