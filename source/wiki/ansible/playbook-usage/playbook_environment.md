---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.13 环境设置
order: 12
---

# 设置远程环境

1.1 版本新增。

您可以在 play、block 或 task 级别使用 environment 关键字，为远程主机上的操作设置环境变量。使用此关键字，您可以为执行 http 请求的任务启用代理，为特定于语言的版本管理器设置所需的环境变量，等等。

当您在 play 或 block 级别使用 environment: 设置值时，该值仅对 play 或 block 内由同一用户执行的任务可用。 environment: 关键字不影响 Ansible 本身、Ansible 配置设置、其他用户的环境，或查找和过滤器等其他插件的执行。使用 environment: 设置的变量不会自动成为 Ansible 事实，即使您在 play 级别设置它们。您必须在 playbook 中包含一个明确的 gather_facts 任务，并在该任务上设置 environment 关键字，才能将这些值转换为 Ansible 事实。

- 在任务中设置远程环境

## 在任务中设置远程环境

您可以直接在任务级别设置环境。

```
- hosts: all
  remote_user: root

  tasks:

    - name: Install cobbler
      ansible.builtin.package:
        name: cobbler
        state: present
      environment:
        http_proxy: http://proxy.example.com:8080
```



您可以通过在 play 中将环境设置定义为变量，并在任务中访问它们，就像访问任何存储的 Ansible 变量一样，来重用环境设置。

```
- hosts: all
  remote_user: root

  # create a variable named "proxy_env" that is a dictionary
  vars:
    proxy_env:
      http_proxy: http://proxy.example.com:8080

  tasks:

    - name: Install cobbler
      ansible.builtin.package:
        name: cobbler
        state: present
      environment: "{{ proxy_env }}"
```



您可以通过在 group_vars 文件中定义环境设置来存储它们，以便在多个 playbook 中重用。

```
---
# file: group_vars/boston

ntp_server: ntp.bos.example.com
backup: bak.bos.example.com
proxy_env:
  http_proxy: http://proxy.bos.example.com:8080
  https_proxy: http://proxy.bos.example.com:8080
```



您可以在 play 级别设置远程环境。

```
- hosts: testing

  roles:
     - php
     - nginx

  environment:
    http_proxy: http://proxy.example.com:8080
```



这些示例显示了代理设置，但您可以以这种方式提供任意数量的设置。

# 使用特定于语言的版本管理器

一些特定于语言的版本管理器（例如 rbenv 和 nvm）要求您在使用这些工具时设置环境变量。手动使用这些工具时，您通常会从脚本或添加到 shell 配置文件中的行中获取一些环境变量。在 Ansible 中，您可以在 play 级别使用 environment 关键字执行此操作。

```
---
### A playbook demonstrating a common npm workflow:
# - Check for package.json in the application directory
# - If package.json exists:
#   * Run npm prune
#   * Run npm install

- hosts: application
  become: false

  vars:
    node_app_dir: /var/local/my_node_app

  environment:
    NVM_DIR: /var/local/nvm
    PATH: /var/local/nvm/versions/node/v4.2.1/bin:{{ ansible_env.PATH }}

  tasks:
  - name: Check for package.json
    ansible.builtin.stat:
      path: '{{ node_app_dir }}/package.json'
    register: packagejson

  - name: Run npm prune
    ansible.builtin.command: npm prune
    args:
      chdir: '{{ node_app_dir }}'
    when: packagejson.stat.exists

  - name: Run npm install
    community.general.npm:
      path: '{{ node_app_dir }}'
    when: packagejson.stat.exists
```



您还可以在任务级别指定环境。

```
---
- name: Install ruby 2.3.1
  ansible.builtin.command: rbenv install {{ rbenv_ruby_version }}
  args:
    creates: '{{ rbenv_root }}/versions/{{ rbenv_ruby_version }}/bin/ruby'
  vars:
    rbenv_root: /usr/local/rbenv
    rbenv_ruby_version: 2.3.1
  environment:
    CONFIGURE_OPTS: '--disable-install-doc'
    RBENV_ROOT: '{{ rbenv_root }}'
    PATH: '{{ rbenv_root }}/bin:{{ rbenv_root }}/shims:{{ rbenv_plugins }}/ruby-build/bin:{{ ansible_env.PATH }}'
```
