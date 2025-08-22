---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 2.1 Ansible 安装
order: 0
---

# Ansible 安装

Ansible 是一款无需代理的自动化工具，您需要将其安装在单个主机上（称为控制节点）。

从控制节点，Ansible 可以通过 SSH、Powershell 远程处理和许多其他传输方式远程管理整个机器群和其他设备（称为被管理节点），所有这些都来自简单的命令行界面，无需数据库或守护进程。

## 控制节点要求

对于您的*控制*节点（运行 Ansible 的机器），您可以使用几乎任何安装了 Python 的类 UNIX 机器。这包括 Red Hat、Debian、Ubuntu、macOS、BSDs 以及在 [Windows 子系统 Linux (WSL) 发行版](https://docs.microsoft.com/en-us/windows/wsl/about) 下的 Windows。不带 WSL 的 Windows 本身不被支持作为控制节点；请参阅 [Matt Davis 的博客文章](http://blog.rolpdog.com/2020/03/why-no-ansible-controller-for-windows.html) 以了解更多信息。



## 被管理节点要求

*被管理*节点（Ansible 管理的机器）不需要安装 Ansible，但需要 Python 来运行 Ansible 生成的 Python 代码。被管理节点还需要一个用户帐户，该帐户可以通过 SSH 使用交互式 POSIX shell 连接到该节点。



## 节点需求总结

您可以在 [ansible-core 控制节点 Python 支持](https://docs.ansible.org.cn/ansible/latest/reference_appendices/release_and_maintenance.html#support-life) 和 [ansible-core 支持矩阵](https://docs.ansible.org.cn/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix) 部分找到有关每个 Ansible 版本的控制节点和被管理节点要求（包括 Python 版本）的详细信息。



## 选择要安装的 Ansible 软件包和版本

Ansible 的社区软件包以两种方式分发

- `ansible-core`：一个包含一组 [内置模块和插件](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin) 的极简语言和运行时软件包。
- `ansible`：一个更大的“包含电池”软件包，它添加了社区精选的 [Ansible 集合](https://docs.ansible.org.cn/ansible/latest/collections_guide/index.html#collections)，用于自动化各种设备。

选择适合您需求的软件包。以下说明使用 `ansible` 作为软件包名称，但如果您希望从最小软件包开始并单独安装所需的 Ansible 集合，则可以将其替换为 `ansible-core`。

`ansible` 或 `ansible-core` 软件包可能在您的操作系统包管理器中可用，您可以自由地使用您首选的方法安装这些软件包。有关更多信息，请参阅 [在特定操作系统上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-distros) 指南。这些安装说明仅涵盖使用 `pip` 安装 Python 软件包的官方支持方法。

请参阅 [Ansible 软件包发行状态表](https://docs.ansible.org.cn/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-changelogs) 以了解软件包中包含的 `ansible-core` 版本。



