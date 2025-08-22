---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.8 控制剧本执行策略 
order: 7
---

# 控制剧本执行：策略及更多

默认情况下，Ansible 使用 5 个并发进程，在开始任何主机上的下一个任务之前，先在所有受剧场影响的主机上运行每个任务。如果要更改此默认行为，可以使用不同的策略插件，更改并发进程数，或应用 serial 等多个关键字。

- 选择策略
- 设置并发进程数
- 使用关键字控制执行
  - 使用 serial 设置批处理大小
  - 使用 throttle 限制执行
  - 基于清单排序执行
  - 使用 run_once 在单台机器上运行

## 选择策略

上面描述的默认行为是 线性策略。Ansible 提供其他策略，包括 调试策略（另见 调试任务）和 自由策略，它允许每个主机尽可能快地运行到剧场的结尾。

```
- hosts: all
  strategy: free
  tasks:
  # ...
```



您可以为每个剧场选择不同的策略（如上所示），或者在 ansible.cfg 文件的 defaults 部分全局设置首选策略。

```
[defaults]
strategy = free
```



所有策略都实现为 策略插件。请查看每个策略插件的文档，了解其工作原理的详细信息。

## 设置并发进程数

如果具备足够的处理能力并想要使用更多并发进程，可以在 ansible.cfg 中设置数量。

```
[defaults]
forks = 30
```



或者在命令行中传递：ansible-playbook -f 30 my_playbook.yml。

## 使用关键字控制执行

除了策略之外，还有几个 关键字 也影响剧场的执行。您可以使用 serial 设置要同时管理的主机数量、百分比或数字列表。Ansible 在完成指定数量或百分比的主机上的剧场后，才会开始下一批主机。您可以使用 throttle 限制分配给块或任务的工作程序数量。您可以使用 order 控制 Ansible 选择要对其执行的组中的下一个主机的方式。您可以使用 run_once 在单个主机上运行任务。这些关键字不是策略。它们是应用于剧场、块或任务的指令或选项。

其他影响剧场执行的关键字包括 ignore_errors、ignore_unreachable 和 any_errors_fatal。这些选项在 剧本中的错误处理 中有说明。



### 使用 serial 设置批处理大小

默认情况下，Ansible 会并行地对您在每个剧场的 hosts: 字段中设置的 模式 中的所有主机进行操作。如果您只想一次管理少量机器（例如在滚动更新期间），可以使用 serial 关键字定义 Ansible 应该一次管理多少主机。

```
---
- name: test play
  hosts: webservers
  serial: 3
  gather_facts: False

  tasks:
    - name: first task
      command: hostname
    - name: second task
      command: hostname
```



在上面的示例中，如果我们在 ‘webservers’ 组中有 6 台主机，Ansible 将在 3 台主机上完全执行剧场（两个任务），然后才能转移到接下来的 3 台主机。

```
PLAY [webservers] ***********************************************************************

TASK [first task] ***********************************************************************
changed: [web1]
changed: [web3]
changed: [web2]

TASK [second task] **********************************************************************
changed: [web1]
changed: [web2]
changed: [web3]

PLAY [webservers] ***********************************************************************

TASK [first task] ***********************************************************************
changed: [web4]
changed: [web5]
changed: [web6]

TASK [second task] **********************************************************************
changed: [web4]
changed: [web5]
changed: [web6]

PLAY RECAP ******************************************************************************
web1                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web2                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web3                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web4                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web5                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web6                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



您还可以使用 serial 关键字指定百分比。Ansible 将百分比应用于剧场中的主机总数以确定每次通过的主机数量。

```
---
- name: test play
  hosts: webservers
  serial: "30%"
```



如果主机数量不能平均分配到通过次数，则最后一次通过包含余数。在这个例子中，如果你的 webservers 组中有 20 台主机，第一批将包含 6 台主机，第二批将包含 6 台主机，第三批将包含 6 台主机，最后一批将包含 2 台主机。

您还可以将批处理大小指定为列表。例如：

```
---
- name: test play
  hosts: webservers
  serial:
    - 1
    - 5
    - 10
```



在上面的示例中，第一批将包含一台主机，下一批将包含 5 台主机，（如果还有主机剩余），每批接下来的批次将包含 10 台主机或所有剩余的主机（如果剩余少于 10 台主机）。

您可以列出多个批处理大小作为百分比。

```
---
- name: test play
  hosts: webservers
  serial:
    - "10%"
    - "20%"
    - "100%"
```



您也可以混合和匹配值。

```
---
- name: test play
  hosts: webservers
  serial:
    - 1
    - 5
    - "20%"
```



### 使用 throttle 限制执行

throttle 关键字限制特定任务的工作程序数量。它可以在块和任务级别设置。使用 throttle 限制可能是 CPU 密集型或与速率限制 API 交互的任务。

```
tasks:
- command: /path/to/cpu_intensive_command
  throttle: 1
```



如果您已经限制了并发进程数或要并行执行的机器数量，可以使用 throttle 减少工作程序数量，但不能增加它。换句话说，为了生效，如果同时使用 forks 或 serial，则 throttle 设置必须低于 forks 或 serial 设置。

### 基于清单排序执行

order 关键字控制运行主机的顺序。order 的可能值是：

- inventory

  (默认) 清单为请求的选择提供的顺序（参见下面的说明）

- reverse_inventory

  与上面相同，但反转返回的列表

- sorted

  按名称字母顺序排序

- reverse_sorted

  按名称反向字母顺序排序

- shuffle

  每次运行随机排序



### 使用run_once在单台机器上运行

如果希望任务仅在主机批次中的第一台主机上运行，请将该任务的run_once设置为true。

```
---
# ...

  tasks:

    # ...

    - command: /opt/application/upgrade_db.py
      run_once: true

    # ...
```



Ansible 会在当前批次中的第一台主机上执行此任务，并将所有结果和事实应用于同一批次中的所有主机。这种方法类似于将条件应用于任务，例如：

```
- command: /opt/application/upgrade_db.py
  when: inventory_hostname == webservers[0]
```



但是，使用run_once，结果会应用于所有主机。要在一个特定主机（而不是批次中的第一台主机）上运行任务，请委托该任务。

```
- command: /opt/application/upgrade_db.py
  run_once: true
  delegate_to: web01.example.org
```



与委托一样，操作将在委托的主机上执行，但信息仍然是任务中原始主机的。
