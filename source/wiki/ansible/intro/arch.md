---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 1.2 Ansible 架构
order: 1
---

# Ansible 架构

Ansible 自动化远程系统的管理并控制其期望状态。

[![Basic components of an Ansible environment include a control node, an inventory of managed nodes, and a module copied to each managed node.](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250814114405869.svg)

如上图所示，大多数 Ansible 环境具有三个主要组件：

- **控制节点**

  安装 Ansible 的系统。您可以在控制节点上运行 Ansible 命令，例如 `ansible` 或 `ansible-inventory`。

- **清单**

  以逻辑方式组织的受管节点列表。您在控制节点上创建清单以向 Ansible 描述主机部署。

- **受管节点**

  Ansible 控制的远程系统或主机。
