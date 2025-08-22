---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.2 Ansible 剧本简介
order: 1
---

# Ansible 剧本简介

Ansible 剧本提供了一个可重复、可重用、简单的配置管理和多机器部署系统，非常适合部署复杂的应用程序。如果您需要多次使用 Ansible 执行任务，请编写一个剧本并将其放在源代码管理下。然后，您可以使用该剧本来部署新的配置或确认远程系统的配置。

剧本可以

- 声明配置
- 协调任何手动排序过程的步骤，在多组机器上按定义的顺序执行
- 同步或异步启动任务

- 剧本语法
- 剧本执行
  - 任务执行
  - 期望状态和“幂等性”
  - 运行剧本
  - 以检查模式运行剧本
- Ansible-Pull
- 验证剧本
  - ansible-lint



## 剧本语法

剧本使用 YAML 格式表达，语法简洁。如果您不熟悉 YAML，请查看我们关于YAML 语法 的概述，并考虑为您的文本编辑器安装一个附加组件（参见其他工具和程序），以帮助您在剧本中编写清晰的 YAML 语法。

一个剧本由一个或多个按顺序排列的“剧目”组成。“剧本”和“剧目”这两个术语是体育类比。每个剧目执行剧本的总体目标的一部分，运行一个或多个任务。每个任务调用一个 Ansible 模块。

## 剧本执行

剧本从上到下按顺序运行。在每个剧目中，任务也从上到下按顺序运行。具有多个“剧目”的剧本可以协调多机器部署，在一个剧目中运行您的 Web 服务器，然后在另一个剧目中运行您的数据库服务器，然后在第三个剧目中运行您的网络基础设施，依此类推。至少，每个剧目定义两件事

- 使用模式来定位要管理的节点
- 要执行的至少一项任务

在这个例子中，第一个剧目定位 Web 服务器；第二个剧目定位数据库服务器。

```
---
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest

  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest

  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started
```



您的剧本可以包含的内容不仅仅是 hosts 行和任务。例如，上面的剧本为每个剧目设置了一个remote_user。这是 SSH 连接的用户帐户。您可以在剧本、剧目或任务级别添加其他剧本关键词 来影响 Ansible 的行为。剧本关键词可以控制连接插件、是否使用权限提升、如何处理错误等等。为了支持各种环境，Ansible 允许您将许多这些参数设置为命令行标志、在 Ansible 配置中或在清单中。了解这些数据源的优先级规则 将有助于您扩展 Ansible 生态系统。



### 任务执行

默认情况下，Ansible 按顺序一次执行一项任务，针对主机模式匹配的所有机器。每个任务都使用特定参数执行模块。当一项任务在所有目标机器上执行完毕后，Ansible 将继续执行下一项任务。您可以使用策略来更改此默认行为。在每个剧目中，Ansible 将相同的任务指令应用于所有主机。如果一项任务在一台主机上失败，Ansible 将把该主机从剧本的其余部分的轮换中移除。

运行剧本时，Ansible 将返回有关连接、所有剧目和任务的name 行、每个任务在每台机器上是否成功或失败以及每个任务在每台机器上是否进行了更改的信息。在剧本执行的底部，Ansible 提供了目标节点及其性能的摘要。常规故障和致命的“无法访问”通信尝试在计数中被分开。



### 期望状态和“幂等性”

大多数 Ansible 模块都会检查是否已经达到所需的最终状态，如果已经达到该状态，则在不执行任何操作的情况下退出，以便重复该任务不会改变最终状态。表现出这种行为的模块通常被称为“幂等”。无论您运行剧本一次还是多次，结果都应该相同。但是，并非所有剧本和所有模块都具有这种行为。如果您不确定，请在沙箱环境中测试您的剧本，然后再在生产环境中多次运行它们。



### 运行剧本

要运行您的剧本，请使用ansible-playbook 命令。

```
ansible-playbook playbook.yml -f 10
```



运行剧本时使用--verbose标志可以查看来自成功模块和不成功模块的详细输出。

### 以检查模式运行剧本

Ansible 的检查模式允许您执行剧本而不会对您的系统进行任何更改。您可以在生产环境中实施剧本之前使用检查模式来测试剧本。

要以检查模式运行剧本，您可以将-C或--check标志传递给ansible-playbook命令

```
ansible-playbook --check playbook.yaml
```



执行此命令将正常运行剧本，但是 Ansible 不会实施任何修改，而只是提供它将进行的更改的报告。此报告包含文件修改、命令执行和模块调用的详细信息。

检查模式提供了一种安全且实用的方法来检查剧本的功能，而不会冒着意外更改系统的风险。此外，它也是一个宝贵的工具，用于对无法按预期运行的剧本进行故障排除。



## Ansible-Pull

如果您想反转 Ansible 的架构，以便节点向中心位置检入，而不是将配置推送到它们，您可以这样做。

ansible-pull是一个小型脚本，它将从 git 中检出配置指令的仓库，然后针对该内容运行ansible-playbook。

假设您对检出位置进行负载均衡，ansible-pull 的扩展性基本上是无限的。

运行ansible-pull --help以获取详细信息。

## 验证剧本

在运行 playbook 之前，您可能需要验证您的 playbook 以捕获语法错误和其他问题。 ansible-playbook 命令提供了多种验证选项，包括 --check、--diff、--list-hosts、--list-tasks 和 --syntax-check。验证 playbook 的工具 描述了其他用于验证和测试 playbook 的工具。



### ansible-lint

您可以使用 ansible-lint 在执行 playbook 之前获得关于 playbook 的详细、Ansible 特定的反馈。例如，如果您在位于本页顶部的名为 verify-apache.yml 的 playbook 上运行 ansible-lint，您应该会得到以下结果。

```
$ ansible-lint verify-apache.yml
[403] Package installs should not use latest
verify-apache.yml:8
Task/Handler: ensure apache is at the latest version
```



ansible-lint 默认规则 页面描述了每个错误。对于 [403]，建议的修复方法是将 playbook 中的 state: latest 更改为 state: present。
