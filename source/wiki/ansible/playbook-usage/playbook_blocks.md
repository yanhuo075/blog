---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.10 块用法 
order: 9
---

# 块用法

块创建任务的逻辑组。块还提供了处理任务错误的方法，类似于许多编程语言中的异常处理。

- 使用块对任务进行分组
- 使用块处理错误

## 使用块对任务进行分组

块中的所有任务都会继承在块级别应用的指令。您可以应用于单个任务的大部分内容（循环除外）都可以在块级别应用，因此块可以更轻松地设置任务通用的数据或指令。该指令不会影响块本身，它仅由块包含的任务继承。例如，when 语句应用于块内的任务，而不是应用于块本身。

块示例，块内有命名任务

```
 tasks:
   - name: Install, configure, and start Apache
     when: ansible_facts['distribution'] == 'CentOS'
     block:
       - name: Install httpd and memcached
         ansible.builtin.yum:
           name:
           - httpd
           - memcached
           state: present

       - name: Apply the foo config template
         ansible.builtin.template:
           src: templates/src.j2
           dest: /etc/foo.conf

       - name: Start service bar and enable it
         ansible.builtin.service:
           name: bar
           state: started
           enabled: True
     become: true
     become_user: root
     ignore_errors: true
```



在上面的示例中，Ansible 在运行块中的三个任务中的每个任务之前都会评估“when”条件。这三个任务还会继承特权提升指令，以 root 用户身份运行。最后，ignore_errors: true 确保即使某些任务失败，Ansible 也会继续执行 playbook。

从 Ansible 2.3 开始，可以使用块的名称。我们建议在所有任务中使用名称，无论是在块内还是其他地方，以便在运行 playbook 时更好地了解正在执行的任务。



## 使用块处理错误

您可以使用带有 rescue 和 always 部分的块来控制 Ansible 如何响应任务错误。

当块中的早期任务失败时，救援块会指定要运行的任务。这种方法类似于许多编程语言中的异常处理。Ansible 仅在任务返回“failed”状态后才运行救援块。错误的任务定义和无法访问的主机不会触发救援块。



块错误处理示例

```
 tasks:
   - name: Handle the error
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute, due to the above task failing, :-('
     rescue:
       - name: Print when errors
         ansible.builtin.debug:
           msg: 'I caught an error, can do stuff here to fix it, :-)'
```



您还可以向块添加 always 部分。无论前一个块的任务状态如何，always 部分中的任务都会运行。



带有 always 部分的块

```
 tasks:
   - name: Always do X
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes, :-)"
```



这些元素共同提供了复杂的错误处理。

带有所有部分的块

```
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute, due to the above task failing, :-('
     rescue:
       - name: Print when errors
         ansible.builtin.debug:
           msg: 'I caught an error'

       - name: Force a failure in middle of recovery! >:-)
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I also never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes"
```



block 中的任务正常执行。如果块中的任何任务返回 failed，则 rescue 部分会执行任务以从错误中恢复。always 部分会运行，无论 block 和 rescue 部分的结果如何。

如果块中发生错误并且救援任务成功，Ansible 会将运行的原始任务的失败状态还原，并继续运行 playbook，就好像原始任务已成功一样。被救援的任务被认为是成功的，并且不会触发 max_fail_percentage 或 any_errors_fatal 配置。但是，Ansible 仍然会在 playbook 统计信息中报告失败。

您可以在救援任务中使用带有 flush_handlers 的块，以确保即使发生错误，所有处理程序也会运行

错误处理中运行处理程序的块

```
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'
         changed_when: true
         notify: Run me even after an error

       - name: Force a failure
         ansible.builtin.command: /bin/false
     rescue:
       - name: Make sure all handlers run
         meta: flush_handlers
 handlers:
    - name: Run me even after an error
      ansible.builtin.debug:
        msg: 'This handler runs even on error'
```



2.1 版本新增功能。

Ansible 为块的 rescue 部分中的任务提供了一些变量

- ansible_failed_task

  返回“failed”并触发救援的任务。例如，要获取名称，请使用 ansible_failed_task.name。

- ansible_failed_result

  触发救援的失败任务的捕获返回结果。这相当于在 register 关键字中使用了此变量。

可以在 rescue 部分中检查这些变量

在 rescue 部分中使用特殊变量。

```
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Do Something
         ansible.builtin.shell: grep $(whoami) /etc/hosts

       - name: Force a failure, if previous one succeeds
         ansible.builtin.command: /bin/false
     rescue:
       - name: All is good if the first task failed
         when: ansible_failed_task.name == 'Do Something'
         ansible.builtin.debug:
           msg: All is good, ignore error as grep could not find 'me' in hosts

       - name: All is good if the second task failed
         when: "'/bin/false' in ansible_failed_result.cmd | d([])"
         ansible.builtin.fail:
           msg: It is still false!!!
```
