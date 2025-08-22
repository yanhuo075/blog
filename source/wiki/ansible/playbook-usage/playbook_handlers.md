---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.11 处理程序
order: 10
---

# 处理程序：运行更改操作

有时您希望任务仅在机器上发生更改时才运行。例如，如果任务更新了该服务的配置，您可能希望重新启动服务，但如果配置未更改，则不希望重新启动。Ansible 使用处理程序来解决此用例。处理程序是仅在收到通知时才运行的任务。

- 处理程序示例
- 通知处理程序
- 通知和循环
- 命名处理程序
- 控制处理程序运行时间
- 定义任务更改时机
- 在处理程序中使用变量
- 角色中的处理程序
- 处理程序中的 include 和 import
- 元任务作为处理程序
- 限制

## 处理程序示例

此 playbook，verify-apache.yml，包含一个带有处理程序的单一 play。

```
---
- name: Verify apache installation
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
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
      notify:
        - Restart apache

    - name: Ensure apache is running
      ansible.builtin.service:
        name: httpd
        state: started

  handlers:
    - name: Restart apache
      ansible.builtin.service:
        name: httpd
        state: restarted
```



在此示例 playbook 中，Apache 服务器在 play 中完成所有任务后由处理程序重新启动。

## 通知处理程序

任务可以使用 notify 关键字指示一个或多个处理程序执行。notify 关键字可以应用于任务，并接受在任务更改时收到通知的处理程序名称列表。或者，也可以提供包含单个处理程序名称的字符串。以下示例演示了如何通过单个任务通知多个处理程序

```
tasks:
- name: Template configuration file
  ansible.builtin.template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
    - Restart apache
    - Restart memcached

handlers:
  - name: Restart memcached
    ansible.builtin.service:
      name: memcached
      state: restarted

  - name: Restart apache
    ansible.builtin.service:
      name: apache
      state: restarted
```



在上面的示例中，处理程序按以下顺序在任务更改时执行：Restart memcached，Restart apache。处理程序按其在 handlers 部分中定义的顺序执行，而不是按 notify 语句中列出的顺序执行。多次通知同一个处理程序只会导致处理程序执行一次，而不管有多少任务通知它。例如，如果多个任务更新配置文件并通知处理程序重新启动 Apache，Ansible 只会重新启动 Apache 一次，以避免不必要的重新启动。

## 通知和循环

任务可以使用循环来通知处理程序。这在与变量结合使用时尤其有用，可以触发多个动态通知。

请注意，如果任务整体发生更改，则会触发处理程序。当使用循环时，如果任何循环项发生更改，则会设置更改状态。也就是说，任何更改都会触发所有处理程序。

```
tasks:
- name: Template services
  ansible.builtin.template:
    src: "{{ item }}.j2"
    dest: /etc/systemd/system/{{ item }}.service
  # Note: if *any* loop iteration triggers a change, *all* handlers are run
  notify: Restart {{ item }}
  loop:
    - memcached
    - apache

handlers:
  - name: Restart memcached
    ansible.builtin.service:
      name: memcached
      state: restarted

  - name: Restart apache
    ansible.builtin.service:
      name: apache
      state: restarted
```



在上面的示例中，如果任一模板文件发生更改，则 memcached 和 apache 都将重新启动；如果没有文件更改，则两者都不会重新启动。

## 命名处理程序

必须命名处理程序，以便任务能够使用 notify 关键字通知它们。

或者，处理程序可以使用 listen 关键字。使用此处理程序关键字，处理程序可以侦听可以对多个处理程序进行分组的主题，如下所示

```
tasks:
  - name: Restart everything
    command: echo "this task will restart the web services"
    notify: "restart web services"

handlers:
  - name: Restart memcached
    service:
      name: memcached
      state: restarted
    listen: "restart web services"

  - name: Restart apache
    service:
      name: apache
      state: restarted
    listen: "restart web services"
```



通知 restart web services 主题会导致执行所有侦听该主题的处理程序，而不管这些处理程序如何命名。

此用法使得触发多个处理程序变得更容易。它还将处理程序与其名称解耦，从而更容易在 playbook 和角色之间共享处理程序（尤其是在使用来自共享源（例如 Ansible Galaxy）的第三方角色时）。

每个处理程序都应该具有全局唯一的名称。如果定义了多个具有相同名称的处理程序，则只有加载到 play 中的最后一个处理程序才能收到通知并执行，有效地屏蔽了所有以前具有相同名称的处理程序。

无论处理程序在何处定义，处理程序只有一个全局范围（处理程序名称和侦听主题）。这也包括在角色中定义的处理程序。

## 控制处理程序运行时间

默认情况下，处理程序在特定 play 中的所有任务完成后运行。已通知的处理程序会在以下每个部分后按以下顺序自动执行：pre_tasks、roles/tasks 和 post_tasks。这种方法效率很高，因为处理程序只运行一次，而不管有多少任务通知它。例如，如果多个任务更新配置文件并通知处理程序重新启动 Apache，Ansible 只会重新启动 Apache 一次，以避免不必要的重新启动。

如果您需要处理程序在 play 结束前运行，请添加一个任务以使用元模块刷新它们，该模块执行 Ansible 操作

```
tasks:
  - name: Some tasks go here
    ansible.builtin.shell: ...

  - name: Flush handlers
    meta: flush_handlers

  - name: Some other tasks
    ansible.builtin.shell: ...
```



meta: flush_handlers 任务会触发在 play 中该点已收到通知的任何处理程序。

一旦处理程序执行完毕（要么在每个提到的部分之后自动执行，要么由 flush_handlers 元任务手动执行），就可以在 play 的后面部分再次通知和运行它们。

## 定义任务更改时机

您可以使用 changed_when 关键字控制何时将任务更改通知处理程序。

在以下示例中，每次复制配置文件时，处理程序都会重新启动服务

```
tasks:
  - name: Copy httpd configuration
    ansible.builtin.copy:
      src: ./new_httpd.conf
      dest: /etc/httpd/conf/httpd.conf
    # The task is always reported as changed
    changed_when: True
    notify: Restart apache
```



有关 changed_when 的更多信息，请参阅 定义“已更改”。

## 在处理程序中使用变量

您可能希望您的 Ansible 处理程序使用变量。例如，如果服务的名称因发行版而略有不同，您希望您的输出显示每个目标机器重新启动服务的精确名称。避免将变量放在处理程序的名称中。由于处理程序名称很早就被模板化了，因此 Ansible 可能无法获得此类处理程序名称的值

```
handlers:
# This handler name may cause your play to fail!
- name: Restart "{{ web_service_name }}"
```



如果处理程序名称中使用的变量不可用，则整个 play 都会失败。在 play 中更改该变量 不会 导致创建新的处理程序。

相反，将变量放在处理程序的任务参数中。您可以使用 include_vars 加载值，如下所示

```
tasks:
  - name: Set host variables based on distribution
    include_vars: "{{ ansible_facts.distribution }}.yml"

handlers:
  - name: Restart web service
    ansible.builtin.service:
      name: "{{ web_service_name | default('httpd') }}"
      state: restarted
```



虽然处理程序名称可以包含模板，但 listen 主题不可以。

## 角色中的处理程序

角色中的处理器不仅仅包含在它们的角色中，而是与一个 playbook 中的所有其他处理器一起插入到全局作用域中。因此，它们可以在定义它们的范围之外使用。这也意味着它们的名字可能与角色外部的处理器冲突。为了确保角色中的处理器得到通知，而不是具有相同名称的角色外部的处理器，请使用以下格式通知处理器：role_name : handler_name。

在roles部分中通知的处理器会在tasks部分结束但任何tasks处理器之前自动刷新。

## 处理器中的包含和导入

将动态包含（例如include_task）作为处理器进行通知会导致执行包含中的所有任务。无法通知在动态包含中定义的处理器。

将静态包含（例如import_task）作为处理器，会导致在 playbook 执行之前，该处理器被来自该导入中的处理器有效地重写。静态包含本身无法被通知；另一方面，该包含中的任务可以单独被通知。

## 元任务作为处理器

从 Ansible 2.14 开始，允许使用元任务并将其作为处理器进行通知。但是请注意，flush_handlers不能用作处理器，以防止出现意外行为。

## 限制

处理器不能运行import_role或include_role。处理器忽略标签。
