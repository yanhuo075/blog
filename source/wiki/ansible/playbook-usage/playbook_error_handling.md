---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.12 错误处理
order: 11
---

# Playbook 中的错误处理

当 Ansible 从命令收到非零返回代码或从模块收到失败时，默认情况下，它会在该主机上停止执行，并在其他主机上继续执行。但是，在某些情况下，您可能需要不同的行为。有时，非零返回代码表示成功。有时，您希望一个主机上的失败停止所有主机上的执行。Ansible 提供了工具和设置来处理这些情况，并帮助您获得所需的行为、输出和报告。

- 忽略失败的命令
- 忽略无法访问的主机错误
- 重置无法访问的主机
- 处理程序和失败
- 定义失败
- 定义“已更改”
- 确保命令和 shell 的成功
- 中止所有主机上的剧本
  - 在第一个错误时中止：any_errors_fatal
  - 设置最大失败百分比
- 控制块中的错误



## 忽略失败的命令

默认情况下，当主机上的任务失败时，Ansible 会停止在该主机上执行任务。您可以使用 ignore_errors 继续执行，尽管发生了故障。

```
- name: Do not count this as a failure
  ansible.builtin.command: /bin/false
  ignore_errors: true
```



ignore_errors 指令仅在任务可以运行并返回“failed”值时才有效。它不会使 Ansible 忽略未定义的变量错误、连接失败、执行问题（例如，缺少软件包）或语法错误。



## 忽略无法访问的主机错误

2.7 版本新增。

您可以使用 ignore_unreachable 关键字忽略由于主机实例“UNREACHABLE”而导致的任务失败。Ansible 会忽略任务错误，但会继续对无法访问的主机执行将来的任务。例如，在任务级别

```
- name: This executes, fails, and the failure is ignored
  ansible.builtin.command: /bin/true
  ignore_unreachable: true

- name: This executes, fails, and ends the play for this host
  ansible.builtin.command: /bin/true
```



在 Playbook 级别

```
- hosts: all
  ignore_unreachable: true
  tasks:
  - name: This executes, fails, and the failure is ignored
    ansible.builtin.command: /bin/true

  - name: This executes, fails, and ends the play for this host
    ansible.builtin.command: /bin/true
    ignore_unreachable: false
```





## 重置无法访问的主机

如果 Ansible 无法连接到主机，它会将该主机标记为“UNREACHABLE”，并将其从运行的活动主机列表中删除。您可以使用 meta: clear_host_errors 重新激活所有主机，以便后续任务可以再次尝试访问它们。



## 处理程序和失败

Ansible 在每个剧本结束时运行处理程序。如果一个任务通知了一个处理程序，但同一剧本中的另一个任务稍后失败，则默认情况下，该处理程序不会在该主机上运行，这可能会使主机处于意外状态。例如，一个任务可以更新配置文件并通知处理程序重新启动某些服务。如果同一剧本中的后续任务失败，则可能会更改配置文件，但不会重新启动服务。

您可以使用 --force-handlers 命令行选项、在剧本中包含 force_handlers: True 或将 force_handlers = True 添加到 ansible.cfg 来更改此行为。强制处理程序时，Ansible 将在所有主机上运行所有已通知的处理程序，即使是任务失败的主机。（请注意，某些错误仍可能阻止处理程序运行，例如主机变得无法访问。）



## 定义失败

Ansible 允许您使用 failed_when 条件在每个任务中定义“失败”的含义。与 Ansible 中的所有条件一样，多个 failed_when 条件的列表使用隐式 and 连接，这意味着仅当满足所有条件时，任务才会失败。如果要在满足任何条件时触发失败，则必须在带有显式 or 运算符的字符串中定义条件。

您可以通过在命令的输出中搜索单词或短语来检查失败

```
- name: Fail task when the command error output prints FAILED
  ansible.builtin.command: /usr/bin/example-command -x -y -z
  register: command_result
  failed_when: "'FAILED' in command_result.stderr"
```



或基于返回代码

```
- name: Fail task when both files are identical
  ansible.builtin.raw: diff foo/file1 bar/file2
  register: diff_cmd
  failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2
```



您还可以组合多个失败条件。如果两个条件都为真，则此任务将失败

```
- name: Check if a file exists in temp and fail task if it does
  ansible.builtin.command: ls /tmp/this_should_not_be_here
  register: result
  failed_when:
    - result.rc == 0
    - '"No such" not in result.stderr'
```



如果希望仅在满足一个条件时任务失败，请将 failed_when 定义更改为

```
failed_when: result.rc == 0 or "No such" not in result.stderr
```



如果您有太多条件无法整齐地放入一行中，则可以使用 > 将其拆分为多行 YAML 值。

```
- name: example of many failed_when conditions with OR
  ansible.builtin.shell: "./myBinary"
  register: ret
  failed_when: >
    ("No such file or directory" in ret.stdout) or
    (ret.stderr != '') or
    (ret.rc == 10)
```





## 定义“已更改”

Ansible 允许您使用 changed_when 条件定义特定任务何时“更改”了远程节点。这使您可以根据返回代码或输出确定是否应在 Ansible 统计信息中报告更改以及是否应触发处理程序。与 Ansible 中的所有条件一样，多个 changed_when 条件的列表使用隐式 and 连接，这意味着仅当满足所有条件时，任务才会报告更改。如果要在满足任何条件时报告更改，则必须在带有显式 or 运算符的字符串中定义条件。例如

```
tasks:

  - name: Report 'changed' when the return code is not equal to 2
    ansible.builtin.shell: /usr/bin/billybass --mode="take me to the river"
    register: bass_result
    changed_when: "bass_result.rc != 2"

  - name: This will never report 'changed' status
    ansible.builtin.shell: wall 'beep'
    changed_when: False

  - name: This task will always report 'changed' status
    ansible.builtin.command: /path/to/command
    changed_when: True
```



您还可以组合多个条件以覆盖“changed”结果。

```
- name: Combine multiple conditions to override 'changed' result
  ansible.builtin.command: /bin/fake_command
  register: result
  ignore_errors: True
  changed_when:
    - '"ERROR" in result.stderr'
    - result.rc == 2
```



有关更多条件语法示例，请参阅定义失败。

## 确保命令和 shell 的成功

command 和 shell 模块关心返回代码，因此如果您的命令的成功退出代码不为零，则可以这样做

```
tasks:
  - name: Run this command and ignore the result
    ansible.builtin.shell: /usr/bin/somecommand || /bin/true
```



## 中止所有主机上的剧本

有时，您希望单个主机上的失败或一定百分比的主机上的失败中止所有主机上的整个剧本。您可以使用 any_errors_fatal 在发生第一个失败后停止剧本执行。为了更精细的控制，您可以使用 max_fail_percentage 在给定百分比的主机失败后中止运行。

### 在第一个错误时中止：any_errors_fatal

如果您设置 any_errors_fatal 并且任务返回错误，则 Ansible 将完成当前批次中所有主机上的致命任务，然后停止在所有主机上执行剧本。后续任务和剧本不会执行。您可以通过向块添加rescue 部分从致命错误中恢复。您可以在剧本或块级别设置 any_errors_fatal。

```
- hosts: somehosts
  any_errors_fatal: true
  roles:
    - myrole

- hosts: somehosts
  tasks:
    - block:
        - include_tasks: mytasks.yml
      any_errors_fatal: true
```



当所有任务都必须 100% 成功才能继续执行剧本时，您可以使用此功能。例如，如果您在多个数据中心的机器上运行服务，并使用负载均衡器将流量从用户传递到该服务，您希望在停止服务进行维护之前禁用所有负载均衡器。为了确保禁用负载均衡器的任务中的任何失败都会停止所有其他任务

```
---
- hosts: load_balancers_dc_a
  any_errors_fatal: true

  tasks:
    - name: Shut down datacenter 'A'
      ansible.builtin.command: /usr/bin/disable-dc

- hosts: frontends_dc_a

  tasks:
    - name: Stop service
      ansible.builtin.command: /usr/bin/stop-software

    - name: Update software
      ansible.builtin.command: /usr/bin/upgrade-software

- hosts: load_balancers_dc_a

  tasks:
    - name: Start datacenter 'A'
      ansible.builtin.command: /usr/bin/enable-dc
```



在此示例中，只有在成功禁用所有负载均衡器后，Ansible 才会开始前端的软件升级。



### 设置最大失败百分比

默认情况下，只要有尚未失败的主机，Ansible 就会继续执行任务。在某些情况下，例如执行滚动更新时，您可能希望在达到一定的失败阈值时中止剧本。为了实现这一点，您可以在剧本上设置最大失败百分比

```
---
- hosts: webservers
  max_fail_percentage: 30
  serial: 10
```



当您将 max_fail_percentage 设置与 serial 一起使用时，该设置将应用于每个批次。在上面的示例中，如果第一批（或任何批次）的 10 台服务器中有超过 3 台失败，则剧本的其余部分将被中止。

## 控制块中的错误

您还可以使用块来定义对任务错误的响应。这种方法类似于许多编程语言中的异常处理。有关详细信息和示例，请参阅 使用块处理错误。
