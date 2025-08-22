---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.6 查找语句
order: 5
---

# 查找语句

查找插件从外部来源检索数据，例如文件、数据库、键值存储、API 和其他服务。与所有模板一样，查找在 Ansible 控制机器上执行和评估。Ansible 使用标准模板系统提供查找插件返回的数据。在 Ansible 2.5 之前，查找主要间接用于with_<lookup>结构中进行循环。从 Ansible 2.5 开始，查找更明确地用作提供给loop关键字的 Jinja2 表达式的一部分。



## lookup 函数

可以使用lookup函数动态填充变量。Ansible 在每次在任务（或模板）中执行时都会评估该值。

```
vars:
  motd_value: "{{ lookup('file', '/etc/motd') }}"
tasks:
  - debug:
      msg: "motd value is {{ motd_value }}"
```



lookup函数的第一个参数是必需的，它指定查找插件的名称。如果查找插件位于集合中，则必须提供完全限定名称，因为collections 关键字不适用于查找插件。

lookup函数还接受一个可选的布尔关键字wantlist，其默认值为False。当为True时，确保查找的结果为列表。

请参阅查找插件的文档以查看特定于插件的参数和关键字。



## query/q 函数

此函数是lookup(..., wantlist=True)的简写。它们是等效的。

```
block:
  - debug:
      msg: "{{ item }}"
    loop: "{{ lookup('ns.col.lookup_items', wantlist=True) }}"

  - debug:
      msg: "{{ item }}"
    loop: "{{ q('ns.col.lookup_items') }}"
```



有关更多详细信息以及 ansible-core 中查找插件的列表，请参阅使用插件。您还可以在集合中找到查找插件。可以使用命令ansible-doc -l -t lookup查看在您的控制机器上安装的查找插件列表。
