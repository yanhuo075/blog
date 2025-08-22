---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.6 调试任务
order: 5
---

# 调试任务

Ansible 提供了一个任务调试器，以便您可以在执行期间修复错误，而无需编辑 playbook 并再次运行它来查看更改是否有效。您可以在任务的上下文中访问调试器的所有功能。您可以检查或设置变量的值，更新模块参数，并使用新的变量和参数重新运行任务。调试器允许您解决故障原因并继续 playbook 的执行。

- 启用调试器
  - 使用 debugger 关键字启用调试器
    - 使用 debugger 关键字的示例
  - 在配置或环境变量中启用调试器
  - 将调试器作为策略启用
- 在调试器中解决错误
- 可用的调试命令
  - print 命令
  - update args 命令
  - update vars 命令
  - update task 命令
  - redo 命令
  - continue 命令
  - quit 命令
- 调试器如何与 free 策略交互

## 启用调试器

默认情况下，调试器未启用。如果您想在 playbook 执行期间调用调试器，则必须先启用它。

使用以下三种方法之一启用调试器

> - 使用 debugger 关键字
> - 在配置或环境变量中，或者
> - 作为策略

### 使用 debugger 关键字启用调试器

2.5 版本新增。

您可以使用 debugger 关键字为特定的 play、角色、块或任务启用（或禁用）调试器。此选项在开发或扩展 playbook、play 和角色时尤其有用。您可以在新任务或更新的任务上启用调试器。如果它们失败，您可以高效地修复错误。debugger 关键字接受五个值

| 值             | 结果                         |
| -------------- | ---------------------------- |
| always         | 始终调用调试器，无论结果如何 |
| never          | 从不调用调试器，无论结果如何 |
| on_failed      | 仅在任务失败时调用调试器     |
| on_unreachable | 仅当主机不可达时调用调试器   |
| on_skipped     | 仅当任务被跳过时调用调试器   |

当您使用 debugger 关键字时，您指定的值会覆盖任何全局配置以启用或禁用调试器。如果您在多个级别（例如在角色和任务中）定义 debugger，Ansible 将遵循最细粒度的定义。play 或角色级别的定义适用于该 play 或角色中的所有块和任务，除非它们指定不同的值。块级别的定义会覆盖 play 或角色级别的定义，并适用于该块中的所有任务，除非它们指定不同的值。任务级别的定义始终适用于该任务；它会覆盖块、play 或角色级别的定义。

#### 使用 debugger 关键字的示例

在任务上设置 debugger 关键字的示例

```
- name: Execute a command
  ansible.builtin.command: "false"
  debugger: on_failed
```



在 play 上设置 debugger 关键字的示例

```
- name: My play
  hosts: all
  debugger: on_skipped
  tasks:
    - name: Execute a command
      ansible.builtin.command: "true"
      when: False
```



在多个级别设置 debugger 关键字的示例

```
- name: Play
  hosts: all
  debugger: never
  tasks:
    - name: Execute a command
      ansible.builtin.command: "false"
      debugger: on_failed
```



在这个示例中，调试器在 play 级别设置为 never，在任务级别设置为 on_failed。如果任务失败，Ansible 将调用调试器，因为任务的定义会覆盖其父 play 的定义。

### 在配置或环境变量中启用调试器

2.5 版本新增。

您可以使用 ansible.cfg 中的设置或环境变量全局启用任务调试器。唯一的选择是 True 或 False。如果您将配置选项或环境变量设置为 True，Ansible 默认情况下会在失败的任务上运行调试器。

要从 ansible.cfg 启用任务调试器，请将此设置添加到 [defaults] 部分

```
[defaults]
enable_task_debugger = True
```



要使用环境变量启用任务调试器，请在运行 playbook 时传递变量

```
ANSIBLE_ENABLE_TASK_DEBUGGER=True ansible-playbook -i hosts site.yml
```



当您全局启用调试器时，每个失败的任务都会调用调试器，除非角色、play、块或任务显式禁用调试器。如果您需要更精细地控制触发调试器的条件，请使用 debugger 关键字。

### 将调试器作为策略启用

如果您正在运行旧版 playbook 或角色，您可能会看到调试器作为 策略启用。您可以在 play 级别、在 ansible.cfg 中或使用环境变量 ANSIBLE_STRATEGY=debug 来执行此操作。例如

```
- hosts: test
  strategy: debug
  tasks:
  ...
```



或者在 ansible.cfg 中

```
[defaults]
strategy = debug
```



## 在调试器中解决错误

Ansible 调用调试器后，您可以使用七个 调试器命令 来解决 Ansible 遇到的错误。考虑此示例 playbook，它定义了 var1 变量，但在任务中错误地使用了未定义的 wrong_var 变量。

```
- hosts: test
  debugger: on_failed
  gather_facts: false
  vars:
    var1: value1
  tasks:
    - name: Use a wrong variable
      ansible.builtin.ping: data={{ wrong_var }}
```



如果您运行此 playbook，当任务失败时，Ansible 将调用调试器。从调试提示符中，您可以更改模块参数或变量并再次运行任务。

```
PLAY ***************************************************************************

TASK [wrong variable] **********************************************************
fatal: [192.0.2.10]: FAILED! => {"failed": true, "msg": "ERROR! 'wrong_var' is undefined"}
Debugger invoked
[192.0.2.10] TASK: wrong variable (debug)> p result._result
{'failed': True,
 'msg': 'The task includes an option with an undefined variable. The error '
        "was: 'wrong_var' is undefined\n"
        '\n'
        'The error appears to have been in '
        "'playbooks/debugger.yml': line 7, "
        'column 7, but may\n'
        'be elsewhere in the file depending on the exact syntax problem.\n'
        '\n'
        'The offending line appears to be:\n'
        '\n'
        '  tasks:\n'
        '    - name: wrong variable\n'
        '      ^ here\n'}
[192.0.2.10] TASK: wrong variable (debug)> p task.args
{u'data': u'{{ wrong_var }}'}
[192.0.2.10] TASK: wrong variable (debug)> task.args['data'] = '{{ var1 }}'
[192.0.2.10] TASK: wrong variable (debug)> p task.args
{u'data': '{{ var1 }}'}
[192.0.2.10] TASK: wrong variable (debug)> redo
ok: [192.0.2.10]

PLAY RECAP *********************************************************************
192.0.2.10               : ok=1    changed=0    unreachable=0    failed=0
```



在调试器中将任务参数更改为使用 var1 而不是 wrong_var 将使任务成功运行。



## 可用的调试命令

您可以在调试提示符下使用以下七个命令

| 命令                   | 快捷键   | 操作                                    |
| ---------------------- | -------- | --------------------------------------- |
| print                  | p        | 打印有关任务的信息                      |
| task.args[key] = value | 无快捷键 | 更新模块参数                            |
| task_vars[key] = value | 无快捷键 | 更新任务变量（您必须接下来update_task） |
| update_task            | u        | 使用更新后的任务变量重新创建任务        |
| redo                   | r        | 再次运行任务                            |
| continue               | c        | 继续执行，从下一个任务开始              |
| quit                   | q        | 退出调试器                              |

更多详情，请参见下面的详细描述和示例。



### 打印命令

print *task/task.args/task_vars/host/result* 打印有关任务的信息。

```
[192.0.2.10] TASK: install package (debug)> p task
TASK: install package
[192.0.2.10] TASK: install package (debug)> p task.args
{u'name': u'{{ pkg_name }}'}
[192.0.2.10] TASK: install package (debug)> p task_vars
{u'ansible_all_ipv4_addresses': [u'192.0.2.10'],
 u'ansible_architecture': u'x86_64',
 ...
}
[192.0.2.10] TASK: install package (debug)> p task_vars['pkg_name']
u'bash'
[192.0.2.10] TASK: install package (debug)> p host
192.0.2.10
[192.0.2.10] TASK: install package (debug)> p result._result
{'_ansible_no_log': False,
 'changed': False,
 u'failed': True,
 ...
 u'msg': u"No package matching 'not_exist' is available"}
```





### 更新参数命令

task.args[*key*] = *value* 更新模块参数。此示例剧本包含无效的包名称。

```
- hosts: test
  strategy: debug
  gather_facts: true
  vars:
    pkg_name: not_exist
  tasks:
    - name: Install a package
      ansible.builtin.apt: name={{ pkg_name }}
```



运行剧本时，无效的包名称会触发错误，Ansible 会调用调试器。您可以通过查看并更新模块参数来修复包名称。

```
[192.0.2.10] TASK: install package (debug)> p task.args
{u'name': u'{{ pkg_name }}'}
[192.0.2.10] TASK: install package (debug)> task.args['name'] = 'bash'
[192.0.2.10] TASK: install package (debug)> p task.args
{u'name': 'bash'}
[192.0.2.10] TASK: install package (debug)> redo
```



更新模块参数后，使用redo命令使用新的参数再次运行任务。



### 更新变量命令

task_vars[*key*] = *value* 更新task_vars。您可以通过查看并更新任务变量（而不是模块参数）来修复上述剧本。

```
[192.0.2.10] TASK: install package (debug)> p task_vars['pkg_name']
u'not_exist'
[192.0.2.10] TASK: install package (debug)> task_vars['pkg_name'] = 'bash'
[192.0.2.10] TASK: install package (debug)> p task_vars['pkg_name']
'bash'
[192.0.2.10] TASK: install package (debug)> update_task
[192.0.2.10] TASK: install package (debug)> redo
```



更新任务变量后，您必须使用update_task加载新变量，然后才能使用redo再次运行任务。



### 更新任务命令

2.8 版本新增。

u 或 update_task 使用更新后的任务变量从原始任务数据结构和模板重新创建任务。有关使用方法示例，请参见条目更新变量命令。



### Redo 命令

r 或 redo 再次运行任务。



### Continue 命令

c 或 continue 继续执行，从下一个任务开始。



### Quit 命令

q 或 quit 退出调试器。剧本执行将中止。

## 调试器如何与 free 策略交互

启用默认的`linear`策略时，Ansible 会在调试器处于活动状态时暂停执行，并在您输入`redo`命令后立即运行调试的任务。但是，启用`free`策略时，Ansible不会等待所有主机，并且可能在一个主机上为后续任务排队，然后再在另一个主机上发生任务失败。使用`free`策略时，Ansible 在调试器处于活动状态时不会排队或执行任何任务。但是，所有排队的任务都将保留在队列中，并在您退出调试器后立即运行。如果您使用`redo`从调试器重新调度任务，则其他排队的任务可能会在您重新调度的任务之前执行。有关策略的更多信息，请参见[控制剧本执行：策略及更多](https://docs.ansible.org.cn/ansible/latest/playbook_guide/playbooks_strategies.html#playbooks-strategies)。
