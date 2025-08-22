---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.4 Ansible 模块使用
order: 3
---

# 使用 Ansible 模块

欢迎来到 Ansible 使用模块、插件和集合的指南。

Ansible 模块是可以控制系统资源或执行系统命令的代码单元。Ansible 提供了一个模块库，您可以直接在远程主机上或通过 playbook 执行这些模块。您还可以编写自定义模块。

与模块类似的是插件，它们是扩展 Ansible 核心功能的代码片段。Ansible 使用插件架构来实现丰富、灵活和可扩展的功能集。Ansible 自带多个插件，并允许您轻松使用自己的插件。

- 模块简介
- 模块维护和支持
  - 维护
  - 问题报告
  - 支持
- 拒绝模块
- 使用插件
  - 动作插件
  - Become 插件
  - 缓存插件
  - 回调插件
  - Cliconf 插件
  - 连接插件
  - 文档片段
  - 过滤器插件
  - Httpapi 插件
  - 资源清单插件
  - 查找插件
  - 模块
  - 模块实用程序
  - Netconf 插件
  - Shell 插件
  - 策略插件
  - 终端插件
  - 测试插件
  - 变量插件
- 模块和插件索引



## 模块介绍

模块（也称为“任务插件”或“库插件”）是可以从命令行或在 playbook 任务中使用的离散代码单元。Ansible 通常在远程托管节点上执行每个模块，并收集返回值。在 Ansible 2.10 及更高版本中，大多数模块都托管在集合中。

您可以从命令行执行模块。

```
ansible webservers -m service -a "name=httpd state=started"
ansible webservers -m ping
ansible webservers -m command -a "/sbin/reboot -t now"
```



每个模块都支持参数。几乎所有模块都接受 key=value 参数，以空格分隔。有些模块不接受参数，而 command/shell 模块只接受您要运行的命令字符串。

从 playbook 中，Ansible 模块的执行方式非常相似。

```
- name: reboot the servers
  command: /sbin/reboot -t now
```



另一种将参数传递给模块的方式是使用 YAML 语法，也称为“复杂参数”。

```
- name: restart webserver
  service:
    name: httpd
    state: restarted
```



所有模块都返回 JSON 格式的数据。这意味着模块可以用任何编程语言编写。模块应该是幂等的，如果它们检测到当前状态与期望的最终状态匹配，则应避免进行任何更改。当在 Ansible playbook 中使用时，模块可以以通知 handlers 运行其他任务的形式触发“更改事件”。

您可以使用 ansible-doc 工具从命令行访问每个模块的文档。

```
ansible-doc yum
```



有关所有可用模块的列表，请参阅[集合文档](https://docs.ansible.org.cn/ansible/latest/collections/index.html#list-of-collections)，或在命令提示符下运行以下命令。

```
ansible-doc -l
```



## 模块维护和支持

如果您正在使用某个模块并发现错误，您可能想知道在哪里报告该错误，谁负责修复它以及如何跟踪模块的更改。如果您是 Red Hat 订阅者，您可能想知道您是否可以获得对您面临问题的支持。

从 Ansible 2.10 开始，大多数模块都位于集合中。每个集合的发布方式反映了该集合中模块的维护和支持。

- 维护
- 问题报告
- 支持

### 维护

| 集合                     | 代码位置                      | 维护者             |
| ------------------------ | ----------------------------- | ------------------ |
| ansible.builtin          | ansible/ansible 仓库 (GitHub) | 核心团队           |
| 在 Galaxy 上分发         | 各种；请关注repo链接          | 社区或合作伙伴     |
| 在 Automation Hub 上分发 | 各种；请关注repo链接          | 内容团队或合作伙伴 |

### 问题报告

如果您发现影响主 Ansible 仓库（也称为ansible-core）中插件的错误

> 1. 确认您正在运行最新稳定版本的 Ansible 或开发分支。
> 2. 查看Ansible 仓库中的问题跟踪器，查看是否已提交问题。
> 3. 如果不存在问题，请创建一个。尽可能详细地说明您发现的行为。

如果您发现影响 Galaxy 集合中插件的错误

> 1. 在 Galaxy 上找到该集合。
> 2. 找到该集合的问题跟踪器。
> 3. 查看是否已提交问题。
> 4. 如果不存在问题，请创建一个。尽可能详细地说明您发现的行为。

一些合作伙伴集合可能托管在私有仓库中。

如果您不确定您看到的行为是否是错误，如果您有任何疑问，如果您想讨论面向开发的主题，或者如果您只想联系我们，请访问Ansible 通信指南，了解如何加入社区。

如果您发现影响 Automation Hub 集合中模块的错误

> 1. 如果集合在 Automation Hub 上提供了“问题跟踪器”链接，请点击该链接并在集合仓库上打开一个问题。如果没有，请按照Red Hat 客户门户上报告问题的标准流程进行操作。您必须订阅 Red Hat Ansible Automation Platform 才能在门户上创建问题。

### 支持

所有保留在`ansible-core`中的插件以及在 Automation Hub 上托管的所有集合都由 Red Hat 支持。Red Hat 不支持其他插件或集合。如果您订阅了 Red Hat Ansible Automation Platform，您可以在[Red Hat 客户门户](https://access.redhat.com/)上找到更多信息和资源。



## 拒绝模块

如果您想避免使用某些模块，可以将它们添加到拒绝列表中，以防止 Ansible 加载它们。要拒绝插件，请创建一个 yaml 配置文件。此文件的默认位置是 /etc/ansible/plugin_filters.yml。您可以使用 PLUGIN_FILTERS_CFG 设置（位于 ansible.cfg 文件的 defaults 部分）选择拒绝列表的不同路径。这是一个拒绝列表示例：

```
---
filter_version: '1.0'
module_rejectlist:
  # Deprecated
  - docker
  # We only allow pip, not easy_install
  - easy_install
```



该文件包含两个字段：

> - 文件版本，以便您可以在保持向后兼容性的同时更新格式。当前版本应为字符串 "1.0"。
> - 要拒绝的模块列表。当 Ansible 搜索要为任务调用的模块时，它不会加载此列表中的任何模块。



## 模块

- 启用模块
- 使用模块

模块是 Ansible Playbook 的主要构建块。虽然我们通常不谈论“模块插件”，但模块是一种插件。有关模块和其他插件之间差异的以开发者为中心的描述，请参阅模块和插件：有什么区别？。



### 启用模块

您可以通过将自定义模块放入以下位置之一来启用它

- 添加到 ANSIBLE_LIBRARY 环境变量的任何目录（$ANSIBLE_LIBRARY 接受像 $PATH 一样的冒号分隔列表）
- ~/.ansible/plugins/modules/
- /usr/share/ansible/plugins/modules/

有关使用本地自定义模块的更多信息，请参阅在集合之外添加模块或插件。



### 使用模块

有关在即席任务中使用模块的信息，请参阅[即席命令简介](https://docs.ansible.org.cn/ansible/latest/command_guide/intro_adhoc.html#intro-adhoc)。有关在 Playbook 中使用模块的信息，请参阅[Ansible Playbook](https://docs.ansible.org.cn/ansible/latest/playbook_guide/playbooks_intro.html#playbooks-intro)。



## 模块实用程序

- 启用模块实用程序
- 使用模块实用程序

模块实用程序包含多个插件使用的共享代码。您可以编写自定义模块实用程序。



### 启用模块实用程序

您可以通过将自定义模块实用程序放入与您的集合或角色相邻的 module_utils 目录中来添加它，就像任何其他插件一样。



### 使用模块实用程序

有关使用模块实用程序的信息，请参阅[使用和开发模块实用程序](https://docs.ansible.org.cn/ansible/latest/dev_guide/developing_module_utilities.html#developing-module-utilities)。



