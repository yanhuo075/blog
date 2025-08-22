---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.7 ansible-playbook
order: 6
---

# ansible-playbook

运行 Ansible playbook，在目标主机上执行定义的任务。



## 概要

```
usage: ansible-playbook [-h] [--version] [-v] [--private-key PRIVATE_KEY_FILE]
                     [-u REMOTE_USER] [-c CONNECTION] [-T TIMEOUT]
                     [--ssh-common-args SSH_COMMON_ARGS]
                     [--sftp-extra-args SFTP_EXTRA_ARGS]
                     [--scp-extra-args SCP_EXTRA_ARGS]
                     [--ssh-extra-args SSH_EXTRA_ARGS]
                     [-k | --connection-password-file CONNECTION_PASSWORD_FILE]
                     [--force-handlers] [--flush-cache] [-b]
                     [--become-method BECOME_METHOD]
                     [--become-user BECOME_USER]
                     [-K | --become-password-file BECOME_PASSWORD_FILE]
                     [-t TAGS] [--skip-tags SKIP_TAGS] [-C] [-D]
                     [-i INVENTORY] [--list-hosts] [-l SUBSET]
                     [-e EXTRA_VARS] [--vault-id VAULT_IDS]
                     [-J | --vault-password-file VAULT_PASSWORD_FILES]
                     [-f FORKS] [-M MODULE_PATH] [--syntax-check]
                     [--list-tasks] [--list-tags] [--step]
                     [--start-at-task START_AT_TASK]
                     playbook [playbook ...]
```



## 描述

这是一个运行 Ansible playbook 的工具，Ansible playbook 是一种配置和多节点部署系统。更多信息请访问项目主页 (https://docs.ansible.org.cn)。

## 常用选项

- --become-method <提权方法>

  要使用的提权方法 (默认值=sudo)，使用 ansible-doc -t become -l 列出有效的选项。

- --become-password-file <提权密码文件>, --become-pass-file <提权密码文件>

  提权密码文件

- --become-user <提权用户>

  以该用户身份运行操作 (默认值=root)

- --connection-password-file <连接密码文件>, --conn-pass-file <连接密码文件>

  连接密码文件

- --flush-cache

  清除清单中每个主机的 fact 缓存

- --force-handlers

  即使任务失败也运行处理器

- --list-hosts

  输出匹配主机的列表；不执行其他任何操作

- --list-tags

  列出所有可用标签

- --list-tasks

  列出将要执行的所有任务

- --private-key <私钥文件>, --key-file <私钥文件>

  使用此文件来验证连接

- --scp-extra-args <SCP额外参数>

  指定要仅传递给 scp 的额外参数 (例如 -l)

- --sftp-extra-args <SFTP额外参数>

  指定要仅传递给 sftp 的额外参数 (例如 -f, -l)

- --skip-tags

  仅运行其标签与这些值不匹配的 play 和任务。此参数可以多次指定。

- --ssh-common-args <SSH常用参数>

  指定要传递给 sftp/scp/ssh 的常用参数 (例如 ProxyCommand)

- --ssh-extra-args <SSH额外参数>

  指定要仅传递给 ssh 的额外参数 (例如 -R)

- --start-at-task <起始任务>

  从与此名称匹配的任务开始 playbook

- --step

  一步一步执行：在运行每个任务之前确认

- --syntax-check

  对 playbook 执行语法检查，但不执行它

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- --version

  显示程序的版本号、配置文件位置、已配置的模块搜索路径、模块位置、可执行文件位置并退出

- -C, --check

  不进行任何更改；而是尝试预测可能发生的一些更改

- -D, --diff

  更改（小）文件和模板时，显示这些文件的差异；与 –check 配合使用效果很好

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码

- -K, --ask-become-pass

  询问提权密码

- -M, --module-path

  将冒号分隔的路径添加到模块库的前面 (默认值={{ ANSIBLE_HOME ~ “/plugins/modules:/usr/share/ansible/plugins/modules” }})。此参数可以多次指定。

- -T <超时时间>, --timeout <超时时间>

  以秒为单位覆盖连接超时时间 (默认值取决于连接)

- -b, --become

  使用 become 运行操作（不暗示密码提示）

- -c <连接类型>, --connection <连接类型>

  要使用的连接类型 (默认值=ssh)

- -e, --extra-vars

  设置附加变量，格式为键值对 (key=value) 或 YAML/JSON 格式。如果文件名以 @ 开头，则表示读取文件内容。此参数可以多次指定。

- -f <FORKS>, --forks <FORKS>

  指定并行进程数 (默认为 5)

- -h, --help

  显示此帮助信息并退出

- -i, --inventory, --inventory-file

  指定清单主机路径或以逗号分隔的主机列表。--inventory-file 已弃用。此参数可以多次指定。

- -k, --ask-pass

  询问连接密码

- -l <SUBSET>, --limit <SUBSET>

  进一步将选定的主机限制到附加模式

- -t, --tags

  仅运行带有这些标签的 playbook 和任务。此参数可以多次指定。

- -u <REMOTE_USER>, --user <REMOTE_USER>

  以该用户身份连接 (默认为 None)

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件当前最多评估到 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。

## 环境变量

可以指定以下环境变量。

ANSIBLE_INVENTORY – 覆盖默认的 Ansible 清单文件

ANSIBLE_LIBRARY – 覆盖默认的 Ansible 模块库路径

ANSIBLE_CONFIG – 覆盖默认的 Ansible 配置文件

Ansible.cfg 中还有许多其他选项可供使用。

## 文件

/etc/ansible/hosts – 默认清单文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
