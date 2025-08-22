---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.3 网络自动化的不同之处
order: 2
---

# 网络自动化的不同之处

网络自动化使用 Ansible 的基本概念，但是网络模块的工作方式存在重要区别。本介绍将帮助你理解本指南中的练习。

- 在控制节点上执行
- 多种通信协议
- 按网络平台组织的集合
- 权限提升：enable 模式、become 和 authorize
  - 使用 become 进行权限提升

## 在控制节点上执行

与大多数 Ansible 模块不同，网络模块不会在被管理的节点上运行。从用户的角度来看，网络模块的工作方式与其他模块一样。它们可以与临时命令、剧本和角色一起使用。但是，在幕后，网络模块使用与其他（Linux/Unix 和 Windows）模块不同的方法。Ansible是用Python编写并执行的。由于大多数网络设备无法运行 Python，因此 Ansible 网络模块在 Ansible 控制节点上执行， ansible 或 ansible-playbook 在此处运行。

对于那些提供 backup 选项的模块，网络模块也使用控制节点作为备份文件的目标位置。对于 Linux/Unix 模块，如果被管理节点上已存在配置文件，则默认情况下会在与新的更改后的文件相同的目录中写入备份文件。网络模块不会更新被管理节点上的配置文件，因为网络配置不是写入文件中的。网络模块会在控制节点上写入备份文件，通常位于剧本根目录下的 backup 目录中。

当使用连接插件（例如，ansible.netcommon.network_cli）进行网络模块连接时，诸如 ansible.builtin.file 和 ansible.builtin.copy 之类的 Unix/Linux 模块也会在控制节点上运行。

## 多种通信协议

由于网络模块在控制节点而不是在被管理的节点上执行，因此它们可以支持多种通信协议。为每个网络模块选择的通信协议（通过 SSH 的 XML、通过 SSH 的 CLI、通过 HTTPS 的 API）取决于平台和模块的目的。一些网络模块仅支持一种协议；一些模块提供选择。最常见的协议是通过 SSH 的 CLI。您可以使用 ansible_connection 变量设置通信协议。

| ansible_connection 的值       | 协议                   | 需求            | 持久性？ |
| ----------------------------- | ---------------------- | --------------- | -------- |
| ansible.netcommon.network_cli | 通过 SSH 的 CLI        | network_os 设置 | 是       |
| ansible.netcommon.netconf     | 通过 SSH 的 XML        | network_os 设置 | 是       |
| ansible.netcommon.httpapi     | 通过 HTTP/HTTPS 的 API | network_os 设置 | 是       |
| 本地                          | 取决于提供程序         | 提供程序设置    | 否       |

ansible_connection: local 已被弃用。请改用上面列出的持久性连接类型之一。使用持久性连接，你只需定义一次主机和凭据，而无需在每个任务中都定义。你还需要为要与其通信的特定网络平台设置 network_os 变量。有关在各种平台上使用每种连接类型的更多详细信息，请参阅 特定于平台的 页面。

## 按网络平台组织的集合

网络平台是一组具有通用操作系统并可由 Ansible 集合管理的网络设备，例如

- Arista：arista.eos
- Cisco：cisco.ios、cisco.iosxr、cisco.nxos
- Juniper：junipernetworks.junos
- VyOS vyos.vyos

一个网络平台内的所有模块都共享某些需求。一些网络平台存在特定差异 - 有关详细信息，请参阅 特定于平台的 文档。



## 权限提升：enable 模式、become 和 authorize

一些网络平台支持权限提升，其中某些任务必须由特权用户执行。在网络设备上，这称为 enable 模式（相当于 *nix 管理中的 sudo）。Ansible 网络模块为支持它的网络设备提供权限提升。有关哪些平台支持 enable 模式的详细信息以及如何使用它的示例，请参阅 特定于平台的 文档。

### 使用 become 进行权限提升

使用顶级 Ansible 参数 become: true 和 become_method: enable 在任何支持权限提升的网络平台上以提升的权限运行任务、剧本或剧本集。你必须将 connection: network_cli 或 connection: httpapi 与 become: true 和 become_method: enable 一起使用。如果你使用 network_cli 将 Ansible 连接到你的网络设备，则 group_vars 文件将如下所示

```
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: cisco.ios.ios
ansible_become: true
ansible_become_method: enable
```
