---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.5 故障排除
order: 4
---

# 使用剧本进行故障排除

在测试新的剧本或调试剧本时，您可能需要多次运行相同的剧本。为了提高效率，Ansible 提供了两种替代方法来执行剧本：start-at-task 和逐步模式。



## start-at-task

要从特定任务（通常是上次运行失败的任务）开始执行剧本，请使用--start-at-task选项。

```
ansible-playbook playbook.yml --start-at-task="install packages"
```



在这个例子中，Ansible 从名为“安装包”的任务开始执行您的剧本。此功能不适用于动态重用角色或任务(include_*)内的任务，请参见[比较 includes 和 imports：动态和静态重用](https://docs.ansible.org.cn/ansible/latest/playbook_guide/playbooks_reuse.html#dynamic-vs-static)。



## 逐步模式

要交互式地执行剧本，请使用--step。

```
ansible-playbook playbook.yml --step
```



使用此选项，Ansible 会在每个任务上停止，并询问是否应该执行该任务。例如，如果您有一个名为“配置 ssh”的任务，则剧本运行将停止并询问。

```
Perform task: configure ssh (y/n/c):
```



回答“y”以执行任务，“n”以跳过任务，“c”以退出逐步模式，执行所有剩余任务而无需询问。
