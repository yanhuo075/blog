---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.2 验证任务
order: 1
---

# 验证任务：检查模式和差异模式

Ansible 提供了两种执行模式来验证任务：检查模式和差异模式。这些模式可以单独使用或组合使用。当您创建或编辑 playbook 或角色，并且想知道它将做什么时，它们很有用。在检查模式下，Ansible 运行但不更改远程系统。支持检查模式的模块会报告它们将进行的更改。不支持检查模式的模块不报告任何内容，也不执行任何操作。在差异模式下，Ansible 提供前后比较。支持差异模式的模块会显示详细信息。您可以结合使用检查模式和差异模式来详细验证您的 playbook 或角色。

- 使用检查模式
  - 强制或阻止任务使用检查模式
  - 在检查模式下跳过任务或忽略错误
- 使用差异模式
  - 强制或阻止任务使用差异模式

## 使用检查模式

检查模式只是一种模拟。它不会为使用基于注册变量的条件（先前任务的结果）的任务生成输出。但是，它非常适合验证一次在一个节点上运行的配置管理 playbook。要在检查模式下运行 playbook

```
ansible-playbook foo.yml --check
```





### 强制或阻止任务使用检查模式

2.2 版本新增。

如果您希望某些任务始终在检查模式下运行，或者永远不运行，无论您是否使用 --check 运行 playbook，您都可以将 check_mode 选项添加到这些任务中

> - 要强制任务在检查模式下运行，即使在不使用 --check 调用 playbook 时也是如此，请设置 check_mode: true。
> - 要强制任务在正常模式下运行并更改系统，即使在使用 --check 调用 playbook 时也是如此，请设置 check_mode: false。

例如

```
tasks:
  - name: This task will always make changes to the system
    ansible.builtin.command: /something/to/run --even-in-check-mode
    check_mode: false

  - name: This task will never make changes to the system
    ansible.builtin.lineinfile:
      line: "important config"
      dest: /path/to/myconfig.conf
      state: present
    check_mode: true
    register: changes_to_important_config
```



使用 check_mode: true 运行单个任务对于测试 Ansible 模块非常有用，无论是测试模块本身还是测试模块将进行更改的条件。您可以在这些任务上注册变量（请参阅条件），以获取有关潜在更改的更多详细信息。

### 在检查模式下跳过任务或忽略错误

2.1 版本新增。

如果您想在检查模式下运行 Ansible 时跳过任务或忽略任务错误，可以使用布尔魔术变量 ansible_check_mode，当 Ansible 在检查模式下运行时，该变量设置为 True。例如

```
tasks:

  - name: This task will be skipped in check mode
    ansible.builtin.git:
      repo: ssh://[email protected]/mylogin/hello.git
      dest: /home/mylogin/hello
    when: not ansible_check_mode

  - name: This task will ignore errors in check mode
    ansible.builtin.git:
      repo: ssh://[email protected]/mylogin/hello.git
      dest: /home/mylogin/hello
    ignore_errors: "{{ ansible_check_mode }}"
```





## 使用差异模式

ansible-playbook 的 --diff 选项可以单独使用或与 --check 一起使用。在差异模式下运行时，任何支持差异模式的模块都会报告所做的更改，或者，如果与 --check 一起使用，则报告将要进行的更改。差异模式在操作文件的模块（例如，模板模块）中最常见，但其他模块也可能显示“前后”信息（例如，用户模块）。

差异模式会产生大量输出，因此最好在一次检查单个主机时使用。例如

```
ansible-playbook foo.yml --check --diff --limit foo.example.com
```



2.4 版本新增。

### 强制或阻止任务使用差异模式

因为 --diff 选项可能会泄露敏感信息，您可以通过指定 diff: false 来禁用任务的此选项。例如

```
tasks:
  - name: This task will not report a diff when the file changes
    ansible.builtin.template:
      src: secret.conf.j2
      dest: /etc/secret.conf
      owner: root
      group: root
      mode: '0600'
    diff: false
```

