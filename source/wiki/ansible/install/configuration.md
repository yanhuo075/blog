---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 2.5 配置 Ansible
order: 4
---

# 配置 Ansible

主题

- [配置 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_configuration.html#configuring-ansible)
  - [配置文件](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_configuration.html#configuration-file)
    - [获取最新配置](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_configuration.html#getting-the-latest-configuration)
  - [环境变量配置](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_configuration.html#environmental-configuration)
  - [命令行选项](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_configuration.html#command-line-options)

本主题介绍如何控制 Ansible 设置。



## 配置文件

Ansible 中的某些设置可以使用配置文件 (`ansible.cfg`) 进行调整。对于大多数用户来说，默认配置应该足够了，但可能有一些原因让你想要更改它们。

搜索配置文件的路径在[参考文档](https://docs.ansible.org.cn/ansible/latest/reference_appendices/config.html#ansible-configuration-settings-locations)中列出。



### 获取最新配置

如果从软件包管理器安装 Ansible，最新的 `ansible.cfg` 文件应位于 `/etc/ansible` 中，在更新的情况下，可能作为 `.rpmnew` 文件（或其他）存在。

如果从 `pip` 或源代码安装 Ansible，你可能需要创建此文件以覆盖 Ansible 中的默认设置。

你可以生成一个 Ansible 配置文件，`ansible.cfg`，其中列出了所有默认设置，如下所示：

```
ansible-config init --disabled > ansible.cfg
```



包括可用的插件以创建更完整的 Ansible 配置，如下所示：

```
ansible-config init --disabled -t all > ansible.cfg
```



有关更多详细信息和可用配置的完整列表，请转到[configuration_settings](https://docs.ansible.org.cn/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)。

你可以使用[ansible-config](https://docs.ansible.org.cn/ansible/latest/cli/ansible-config.html#ansible-config)命令行实用程序列出可用的选项并检查当前值。

有关深入的详细信息，请参阅[Ansible 配置设置](https://docs.ansible.org.cn/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)。



## 环境变量配置

Ansible 还允许使用环境变量配置设置。

如果设置了这些环境变量，它们将覆盖从配置文件加载的任何关联设置。你可以从以下位置获取可用环境变量的完整列表：

- [Ansible 配置设置](https://docs.ansible.org.cn/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)：用于配置核心功能
- [所有集合环境变量的索引](https://docs.ansible.org.cn/ansible/latest/collections/environment_variables.html#list-of-collection-env-vars)：用于配置集合中的插件



## 命令行选项

并非所有配置选项都存在于命令行中，只有那些被认为最有用或最常见的选项。命令行中的设置将覆盖通过配置文件和环境变量传递的设置。

可用的完整选项列表在[ansible-playbook](https://docs.ansible.org.cn/ansible/latest/cli/ansible-playbook.html#ansible-playbook)和[ansible](https://docs.ansible.org.cn/ansible/latest/cli/ansible.html#ansible)中。
