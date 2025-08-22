---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.14 剧本重用
order: 13
---

# 重用 Ansible 剧本

你可以将一个简单的 Playbook 编写在一个非常大的文件中，大多数用户首先学习单文件方法。但是，将自动化工作分解成更小的文件是组织复杂任务集和重用它们的好方法。更小、更分散的工件允许你在多个 Playbook 中重用相同的变量、任务和 Play，以解决不同的用例。你可以在多个父 Playbook 中，甚至在一个 Playbook 内多次使用分散的工件。例如，你可能希望在几个不同的 Playbook 中更新你的客户数据库。如果你将所有与更新数据库相关的任务放在一个任务文件或角色中，你可以在许多 Playbook 中重用它们，同时只在一个地方维护它们。

- 创建可重用的文件和角色
- 重用 Playbook
- 何时将 Playbook 转换为角色
- 重用文件和角色
  - 包含：动态重用
  - 导入：静态重用
  - 比较包含和导入：动态和静态重用
- 将任务重用为处理程序
  - 触发包含（动态）处理程序
  - 触发导入（静态）处理程序

## 创建可重用的文件和角色

Ansible 提供四种分散的、可重用的工件：变量文件、任务文件、Playbook 和角色。

> - 变量文件仅包含变量。
> - 任务文件仅包含任务。
> - Playbook 至少包含一个 Play，并且可能包含变量、任务和其他内容。你可以重用紧密聚焦的 Playbook，但你只能静态地重用它们，而不是动态地重用它们。
> - 角色包含一组相关的任务、变量、默认值、处理程序，甚至模块或其他插件在一个定义的文件树中。与变量文件、任务文件或 Playbook 不同，角色可以很容易地通过 Ansible Galaxy 上传和共享。有关创建和使用角色的详细信息，请参阅角色。

2.4 版本中的新功能。

## 重用 Playbook

你可以将多个 Playbook 合并到一个主 Playbook 中。但是，你只能使用导入来重用 Playbook。例如

```
- import_playbook: webservers.yml
- import_playbook: databases.yml
```



导入将其他 Playbook 中的 Playbook 静态地合并到其他 Playbook 中。 Ansible 按列出的顺序运行每个导入的 Playbook 中的 Play 和任务，就好像它们是直接在主 Playbook 中定义的一样。

你可以通过使用变量定义导入的 Playbook 文件名，然后在运行时使用 --extra-vars 或 vars 关键字传递该变量来选择要导入的 Playbook。例如

```
- import_playbook: "/path/to/{{ import_from_extra_var }}"
- import_playbook: "{{ import_from_vars }}"
  vars:
    import_from_vars: /path/to/one_playbook.yml
```



如果你使用 ansible-playbook my_playbook -e import_from_extra_var=other_playbook.yml 运行此 Playbook，则 Ansible 会导入 one_playbook.yml 和 other_playbook.yml。

## 何时将 Playbook 转换为角色

对于某些用例，简单的 Playbook 效果很好。但是，从某种复杂程度开始，角色比 Playbook 更好。角色允许你将默认值、处理程序、变量和任务存储在单独的目录中，而不是存储在一个长的文档中。角色很容易在 Ansible Galaxy 上共享。对于复杂的用例，大多数用户发现角色比一体式 Playbook 更容易阅读、理解和维护。

## 重用文件和角色

Ansible 提供了两种在 Playbook 中重用文件和角色的方法：动态和静态。

> - 对于动态重用，在 Play 的任务部分添加一个 include_* 任务
>   - include_role
>   - include_tasks
>   - include_vars
> - 对于静态重用，在 Play 的任务部分添加一个 import_* 任务
>   - import_role
>   - import_tasks

任务包含和导入语句可以在任意深度使用。

你仍然可以在 Play 级别使用裸的 roles 关键字来静态地将角色合并到 Playbook 中。但是，曾经用于任务文件和 Playbook 级别包含的裸 include 关键字现在已被弃用。

### 包含：动态重用

包含角色、任务或变量会动态地将它们添加到 Playbook 中。 Ansible 在 Playbook 中出现时处理包含的文件和角色，因此包含的任务可能会受到顶级 Playbook 中较早任务的结果的影响。包含的角色和任务类似于处理程序——它们可能会或可能不会运行，具体取决于顶级 Playbook 中其他任务的结果。

使用 include_* 语句的主要优点是循环。当循环与包含一起使用时，包含的任务或角色将为循环中的每个项执行一次。

在包含之前，会模板化包含的角色、任务和变量的文件名。

你可以将变量传递到包含中。有关变量继承和优先级的更多详细信息，请参阅变量优先级：我应该将变量放在哪里？。

### 导入：静态重用

导入角色、任务或 Playbook 会静态地将它们添加到 Playbook 中。 Ansible 在运行 Playbook 中的任何任务之前预处理导入的文件和角色，因此导入的内容永远不会受到顶级 Playbook 中其他任务的影响。

导入的角色和任务的文件名支持模板化，但是当 Ansible 预处理导入时，变量必须可用。这可以使用 vars 关键字或使用 --extra-vars 来完成。

你可以将变量传递给导入。如果要在一个 Playbook 中多次运行导入的文件，则必须传递变量。例如

```
tasks:
- import_tasks: wordpress.yml
  vars:
    wp_user: timmy

- import_tasks: wordpress.yml
  vars:
    wp_user: alice

- import_tasks: wordpress.yml
  vars:
    wp_user: bob
```



有关变量继承和优先级的更多详细信息，请参阅变量优先级：我应该将变量放在哪里？。



### 比较包含和导入：动态和静态重用

每种重用分散的 Ansible 工件的方法都有其优点和局限性。你可能会为某些 Playbook 选择动态重用，而为其他 Playbook 选择静态重用。尽管你可以在单个 Playbook 中同时使用动态和静态重用，但最好为每个 Playbook 选择一种方法。混合静态和动态重用可能会在你的 Playbook 中引入难以诊断的错误。下表总结了主要差异，以便你可以为创建的每个 Playbook 选择最佳方法。

|                      | Include_*                           | Import_*                           |
| -------------------- | ----------------------------------- | ---------------------------------- |
| 重用类型             | 动态                                | 静态                               |
| 何时处理             | 运行时，当遇到时                    | 在 Playbook 解析期间预处理         |
| 任务或 Play          | 所有包含都是任务                    | import_playbook 不能是任务         |
| 任务选项             | 仅应用于包含任务本身                | 应用于导入中的所有子任务           |
| 从循环调用           | 为每个循环项执行一次                | 不能在循环中使用                   |
| 使用 --list-tags     | 未列出包含中的标签                  | 所有标签都与 --list-tags 一起显示  |
| 使用 --list-tasks    | 未列出包含中的任务                  | 所有任务都与 --list-tasks 一起显示 |
| 通知处理程序         | 无法触发包含中的处理程序            | 可以触发单独导入的处理程序         |
| 使用 --start-at-task | 无法从包含中的任务开始              | 可以从导入的任务开始               |
| 使用清单变量         | 可以 include_*: {{ inventory_var }} | 无法 import_*: {{ inventory_var }} |
| 对于剧本 (playbooks) | 没有 include_playbook               | 可以导入完整的剧本                 |
| 对于变量文件         | 可以包含变量文件                    | 使用 vars_files: 导入变量          |

## 将任务重用为处理程序

您还可以在剧本的 处理程序：在更改时运行操作 部分中使用包含和导入。例如，如果您想定义如何重启 Apache，您只需为所有剧本执行一次即可。您可以创建一个名为 restarts.yml 的文件，如下所示：

```
# restarts.yml
- name: Restart apache
  ansible.builtin.service:
    name: apache
    state: restarted

- name: Restart mysql
  ansible.builtin.service:
    name: mysql
    state: restarted
```



您可以从导入或包含触发处理程序，但每种重用方法的过程都不同。如果您包含该文件，则必须通知包含本身，这将触发 restarts.yml 中的所有任务。如果您导入该文件，则必须通知 restarts.yml 中的单个任务。您可以将直接任务和处理程序与包含或导入的任务和处理程序混合使用。

### 触发包含的（动态）处理程序

包含在运行时执行，因此包含的名称在剧本执行期间存在，但包含的任务在触发包含本身之前不存在。要使用动态重用的 Restart apache 任务，请引用包含本身的名称。此方法将触发包含文件中作为处理程序的所有任务。例如，使用上面显示的任务文件：

```
- name: Trigger an included (dynamic) handler
  hosts: localhost
  handlers:
    - name: Restart services
      include_tasks: restarts.yml
  tasks:
    - command: "true"
      notify: Restart services
```



### 触发导入的（静态）处理程序

导入在剧本开始之前处理，因此导入的名称在剧本执行期间不再存在，但各个导入的任务的名称确实存在。要使用静态重用的 Restart apache 任务，请引用导入文件中每个任务的名称。例如，使用上面显示的任务文件：

```
- name: Trigger an imported (static) handler
  hosts: localhost
  handlers:
    - name: Restart services
      import_tasks: restarts.yml
  tasks:
    - command: "true"
      notify: Restart apache
    - command: "true"
      notify: Restart mysql
```
