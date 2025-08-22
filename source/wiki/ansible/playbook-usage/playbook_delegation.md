---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.8 控制任务的运行位置
order: 7
---

# 控制任务的运行位置：委托和本地操作

默认情况下，Ansible 会在与您的 Playbook 的 hosts 行匹配的机器上收集 facts 并执行所有任务。此页面向您展示如何将任务委托给不同的机器或组，将 facts 委托给特定的机器或组，或者在本地运行整个 Playbook。使用这些方法，您可以精确且高效地管理相互关联的环境。例如，在更新 Web 服务器时，您可能需要暂时将它们从负载均衡池中移除。您无法在 Web 服务器本身上执行此任务。通过将任务委托给 localhost，您可以将所有任务保留在同一个 play 中。

- 无法委托的任务
- 委托任务
- 委托上下文中的模板
- 委托和并行执行
- 委托 facts
- 本地 Playbook

## 无法委托的任务

某些任务始终在控制节点上执行。这些任务（包括 include、add_host 和 debug）无法委托。您可以从 connection 属性文档中确定是否可以委托操作。如果 connection 属性指示 support 为 False 或 None，则该操作不使用连接，无法委托。



## 委托任务

如果您想在一个主机上执行任务并引用其他主机，请在任务中使用 delegate_to 关键字。这非常适合管理负载均衡池中的节点或控制中断窗口。您可以将委托与 serial 关键字结合使用，以控制一次执行的主机数量。

```
---
- hosts: webservers
  serial: 5

  tasks:
    - name: Take out of load balancer pool
      ansible.builtin.command: /usr/bin/take_out_of_pool {{ inventory_hostname }}
      delegate_to: 127.0.0.1

    - name: Actual steps would go here
      ansible.builtin.yum:
        name: acme-web-stack
        state: latest

    - name: Add back to load balancer pool
      ansible.builtin.command: /usr/bin/add_back_to_pool {{ inventory_hostname }}
      delegate_to: 127.0.0.1
```



此 play 中的第一个和第三个任务在 127.0.0.1 上运行，这是运行 Ansible 的机器。还有一种速记语法，您可以在每个任务的基础上使用：local_action。这是与上面相同的 Playbook，但使用了速记语法来委托给 127.0.0.1

```
---
# ...

  tasks:
    - name: Take out of load balancer pool
      local_action: ansible.builtin.command /usr/bin/take_out_of_pool {{ inventory_hostname }}

# ...

    - name: Add back to load balancer pool
      local_action: ansible.builtin.command /usr/bin/add_back_to_pool {{ inventory_hostname }}
```



您可以使用本地操作调用“rsync”将文件递归复制到托管服务器

```
---
# ...

  tasks:
    - name: Recursively copy files from management server to target
      local_action: ansible.builtin.command rsync -a /path/to/files {{ inventory_hostname }}:/path/to/target/
```



请注意，您必须为此配置无密码 SSH 密钥或 ssh-agent，否则 rsync 会要求输入密码。

要指定更多参数，请使用以下语法

```
---
# ...

  tasks:
    - name: Send summary mail
      local_action:
        module: community.general.mail
        subject: "Summary Mail"
        to: "{{ mail_recipient }}"
        body: "{{ mail_body }}"
      run_once: True
```





## 委托上下文中的模板

请注意，在委托下，执行解释器（通常是 Python）、connection、become 和 shell 插件选项现在将使用委托到的主机的值进行模板化。除了 inventory_hostname 之外的所有变量现在都将从此主机而不是原始任务主机中消耗。如果那些选项需要来自原始任务主机的变量，则必须使用 hostvars[inventory_hostname]['varname']，即使 inventory_hostname_short 也引用委托的主机。



## 委托和并行执行

默认情况下，Ansible 任务是并行执行的。委托任务不会改变这一点，也不会处理并发问题（多个 fork 写入同一文件）。最常见的情况是，当用户为所有主机更新单个委托主机上的单个文件时（例如，使用 copy、template 或 lineinfile 模块）。它们仍然以并行 fork（默认 5 个）运行并相互覆盖。

这可以通过几种方式处理

```
- name: "handle concurrency with a loop on the hosts with `run_once: true`"
  lineinfile: "<options here>"
  run_once: true
  loop: '{{ ansible_play_hosts_all }}'
```



通过使用带有 serial: 1 的中间 play 或在任务级别使用 throttle: 1，有关更多详细信息，请参阅 控制 Playbook 执行：策略等



## 委托 facts

委托 Ansible 任务就像在现实世界中委托任务一样 - 您的杂货属于您，即使其他人将它们送到您的家中。同样，由委托任务收集的任何 facts 默认情况下都分配给 inventory_hostname（当前主机），而不是分配给生成 facts 的主机（委托到的主机）。要将收集的 facts 分配给委托的主机而不是当前主机，请将 delegate_facts 设置为 true。

```
---
- hosts: app_servers

  tasks:
    - name: Gather facts from db servers
      ansible.builtin.setup:
      delegate_to: "{{ item }}"
      delegate_facts: true
      loop: "{{ groups['dbservers'] }}"
```



此任务收集 dbservers 组中机器的 facts，并将这些 facts 分配给这些机器，即使该 play 的目标是 app_servers 组。这样您就可以查找 hostvars‘dbhost1’[‘address’]，即使 dbservers 不是 play 的一部分，或者通过使用 --limit 排除了它们。



## 本地 Playbook

在远程主机上本地使用 Playbook 可能很有用，而不是通过 SSH 连接。这对于通过将 Playbook 放入 crontab 中来确保系统的配置很有用。这也可以用于在操作系统安装程序（例如 Anaconda kickstart）中运行 Playbook。

要在本地运行整个 Playbook，只需将 hosts: 行设置为 hosts: 127.0.0.1，然后按如下方式运行 Playbook

```
ansible-playbook playbook.yml --connection=local
```



或者，可以在单个 Playbook play 中使用本地连接，即使 Playbook 中的其他 play 使用默认的远程连接类型

```
---
- hosts: 127.0.0.1
  connection: local
```
