---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.7 异步操作和轮询
order: 6
---

# 异步操作和轮询

默认情况下，Ansible 同步运行任务，保持与远程节点的连接打开，直到操作完成。这意味着，在一个 playbook 中，每个任务默认都会阻塞下一个任务，后续任务只有在当前任务完成后才会运行。这种行为可能会带来挑战。例如，一个任务的完成时间可能比 SSH 会话允许的时间长，从而导致超时。或者您可能希望在并发执行其他任务的同时，在后台执行一个长时间运行的进程。异步模式允许您控制长时间运行的任务的执行方式。

- 异步 ad hoc 任务
- 异步 playbook 任务
  - 避免连接超时：poll > 0
  - 并发运行任务：poll = 0

## 异步 ad hoc 任务

您可以使用 ad hoc 任务 在后台执行长时间运行的操作。例如，要异步地在后台执行 long_running_operation，超时时间 (-B) 为 3600 秒，且不进行轮询 (-P)

```
$ ansible all -B 3600 -P 0 -a "/usr/bin/long_running_operation --do-stuff"
```



要稍后检查作业状态，请使用 async_status 模块，并将您在后台运行原始作业时返回的作业 ID 传递给它。

```
$ ansible web1.example.com -m async_status -a "jid=488359678239.2844"
```



Ansible 还可以通过轮询自动检查长时间运行作业的状态。在大多数情况下，Ansible 会在轮询之间保持与远程节点的连接。要运行 30 分钟并每 60 秒轮询一次状态

```
$ ansible all -B 1800 -P 60 -a "/usr/bin/long_running_operation --do-stuff"
```



轮询模式很智能，因此所有作业都会在任何机器上开始轮询之前启动。如果您希望所有作业都非常快速地启动，请务必使用足够高的 --forks 值。在时间限制 (以秒为单位) 到期 (-B) 后，远程节点上的进程将被终止。

异步模式最适合长时间运行的 shell 命令或软件升级。例如，异步运行 copy 模块不会进行后台文件传输。

## 异步 playbook 任务

Playbook 也支持异步模式和轮询，并具有简化的语法。您可以在 playbook 中使用异步模式来避免连接超时或避免阻塞后续任务。playbook 中异步模式的行为取决于 poll 的值。

### 避免连接超时：poll > 0

如果您想为 playbook 中的某个特定任务设置更长的超时限制，请使用 async 并将 poll 设置为正值。Ansible 仍然会阻塞 playbook 中的下一个任务，等待异步任务完成、失败或超时。但是，只有当任务超过您使用 async 参数设置的超时限制时，才会超时。

要避免任务超时，请指定其最大运行时间以及您希望多久轮询一次状态

```
---

- hosts: all
  remote_user: root

  tasks:

  - name: Simulate long running op (15 sec), wait for up to 45 sec, poll every 5 sec
    ansible.builtin.command: /bin/sleep 15
    async: 45
    poll: 5
```



### 并发运行任务：poll = 0

如果您想在 playbook 中并发运行多个任务，请使用 async 并将 poll 设置为 0。当您设置 poll: 0 时，Ansible 将启动任务并立即转到下一个任务，而无需等待结果。每个异步任务都运行到完成、失败或超时 (运行时间超过其 async 值) 为止。playbook 运行结束时不会检查异步任务。

要异步运行 playbook 任务

```
---

- hosts: all
  remote_user: root

  tasks:

  - name: Simulate long running op, allow to run for 45 sec, fire and forget
    ansible.builtin.command: /bin/sleep 15
    async: 45
    poll: 0
```



如果您需要与异步任务进行同步点，可以将其注册以获取其作业 ID，并使用 async_status 模块在以后的任务中观察它。例如

```
- name: Run an async task
  ansible.builtin.yum:
    name: docker-io
    state: present
  async: 1000
  poll: 0
  register: yum_sleeper

- name: Check on an async task
  async_status:
    jid: "{{ yum_sleeper.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 100
  delay: 10
```



要同时运行多个异步任务，同时限制并发运行的任务数量

```
#####################
# main.yml
#####################
- name: Run items asynchronously in batch of two items
  vars:
    sleep_durations:
      - 1
      - 2
      - 3
      - 4
      - 5
    durations: "{{ item }}"
  include_tasks: execute_batch.yml
  loop: "{{ sleep_durations | batch(2) | list }}"

#####################
# execute_batch.yml
#####################
- name: Async sleeping for batched_items
  ansible.builtin.command: sleep {{ async_item }}
  async: 45
  poll: 0
  loop: "{{ durations }}"
  loop_control:
    loop_var: "async_item"
  register: async_results

- name: Check sync status
  async_status:
    jid: "{{ async_result_item.ansible_job_id }}"
  loop: "{{ async_results.results }}"
  loop_control:
    loop_var: "async_result_item"
  register: async_poll_results
  until: async_poll_results.finished
  retries: 30
```

