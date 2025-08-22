---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.4 ansible-console
order: 3
---

# ansible-console

用于执行 Ansible 任务的 REPL 控制台。

## 概要

```
usage: ansible-console [-h] [--version] [-v] [-b]
                    [--become-method BECOME_METHOD]
                    [--become-user BECOME_USER]
                    [-K | --become-password-file BECOME_PASSWORD_FILE]
                    [-i INVENTORY] [--list-hosts] [-l SUBSET]
                    [--private-key PRIVATE_KEY_FILE] [-u REMOTE_USER]
                    [-c CONNECTION] [-T TIMEOUT]
                    [--ssh-common-args SSH_COMMON_ARGS]
                    [--sftp-extra-args SFTP_EXTRA_ARGS]
                    [--scp-extra-args SCP_EXTRA_ARGS]
                    [--ssh-extra-args SSH_EXTRA_ARGS]
                    [-k | --connection-password-file CONNECTION_PASSWORD_FILE]
                    [-C] [-D] [--vault-id VAULT_IDS]
                    [-J | --vault-password-file VAULT_PASSWORD_FILES]
                    [-f FORKS] [-M MODULE_PATH] [--playbook-dir BASEDIR]
                    [-e EXTRA_VARS] [--task-timeout TASK_TIMEOUT] [--step]
                    [pattern]
```



## 描述

一个 REPL，允许从一个带有内置制表符补全功能的友好 shell 中针对所选清单运行即席任务（基于 dominis 的 ansible-shell）。

它支持几个命令，您可以在运行时修改其配置

- cd [pattern]：更改主机/组（您可以使用主机模式，例如：app*.dc*:!app01*）
- list：列出当前路径中可用的主机
- list groups：列出当前路径中包含的组
- become：切换 become 标志
- !：强制使用 shell 模块而不是 ansible 模块（!yum update -y）
- verbosity [num]：设置详细级别
- forks [num]：设置 fork 的数量
- become_user [user]：设置 become_user
- remote_user [user]：设置 remote_user
- become_method [method]：设置权限提升方法
- check [bool]：切换检查模式
- diff [bool]：切换 diff 模式
- timeout [integer]：设置任务超时时间（以秒为单位）（0 表示禁用）
- help [command/module]：显示命令或模块的文档
- exit：退出 ansible-console

## 常用选项

- --become-method <BECOME_METHOD>

  要使用的权限提升方法（默认值=sudo），使用 ansible-doc -t become -l 列出有效选择。

- --become-password-file <BECOME_PASSWORD_FILE>, --become-pass-file <BECOME_PASSWORD_FILE>

  Become 密码文件

- --become-user <BECOME_USER>

  以该用户身份运行操作（默认值=root）

- --connection-password-file <CONNECTION_PASSWORD_FILE>, --conn-pass-file <CONNECTION_PASSWORD_FILE>

  连接密码文件

- --list-hosts

  输出匹配主机的列表；不执行任何其他操作

- --playbook-dir <BASEDIR>

  由于此工具不使用 playbook，请使用此目录作为替代的 playbook 目录。这设置了许多功能的相对路径，包括 roles/ group_vars/ 等。

- --private-key <PRIVATE_KEY_FILE>, --key-file <PRIVATE_KEY_FILE>

  使用此文件来验证连接

- --scp-extra-args <SCP_EXTRA_ARGS>

  指定仅传递给 scp 的额外参数（例如 -l）

- --sftp-extra-args <SFTP_EXTRA_ARGS>

  指定仅传递给 sftp 的额外参数（例如 -f、-l）

- --ssh-common-args <SSH_COMMON_ARGS>

  指定传递给 sftp/scp/ssh 的通用参数（例如 ProxyCommand）

- --ssh-extra-args <SSH_EXTRA_ARGS>

  指定仅传递给 ssh 的额外参数（例如 -R）

- --step

  一次一步：在运行每个任务之前确认

- --task-timeout <TASK_TIMEOUT>

  设置任务超时限制（以秒为单位），必须为正整数。

- --vault-id

  要使用的 Vault 身份。可以多次指定此参数。

- --vault-password-file, --vault-pass-file

  Vault 密码文件

- --version

  显示程序的版本号、配置文件位置、配置的模块搜索路径、模块位置、可执行文件位置并退出

- -C, --check

  不进行任何更改；而是尝试预测可能发生的一些更改

- -D, --diff

  在更改（小）文件和模板时，显示这些文件中的差异；与 –check 一起使用效果很好

- -J, --ask-vault-password, --ask-vault-pass

  请求 Vault 密码

- -K, --ask-become-pass

  请求权限提升密码

- -M, --module-path

  将以冒号分隔的路径添加到模块库路径的前面 (默认值={{ ANSIBLE_HOME ~ “/plugins/modules:/usr/share/ansible/plugins/modules” }}). 此参数可以指定多次。

- -T <TIMEOUT>, --timeout <TIMEOUT>

  覆盖连接超时时间，单位为秒 (默认值取决于连接方式)

- -b, --become

  使用 become 运行操作 (不表示需要密码提示)

- -c <CONNECTION>, --connection <CONNECTION>

  要使用的连接类型 (默认值=ssh)

- -e, --extra-vars

  设置额外的变量为 key=value 或 YAML/JSON 格式，如果为文件名，则在前面加上 @。 此参数可以指定多次。

- -f <FORKS>, --forks <FORKS>

  指定要使用的并行进程数 (默认值=5)

- -h, --help

  显示此帮助信息并退出

- -i, --inventory, --inventory-file

  指定清单主机路径或逗号分隔的主机列表。 –inventory-file 已弃用。此参数可以指定多次。

- -k, --ask-pass

  询问连接密码

- -l <SUBSET>, --limit <SUBSET>

  进一步限制所选主机为附加的模式

- -u <REMOTE_USER>, --user <REMOTE_USER>

  以该用户身份连接 (默认值=None)

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件当前评估最多 -vvvvvv。合理的起始级别为 -vvv，连接调试可能需要 -vvvv。此参数可以指定多次。

## 参数

- host-pattern

  清单中一个组的名称，一个类似 shell 的 glob 选择清单中的主机，或者两者用逗号分隔的任意组合。

## 环境

可以指定以下环境变量。

ANSIBLE_INVENTORY – 覆盖默认的 Ansible 清单文件

ANSIBLE_LIBRARY – 覆盖默认的 Ansible 模块库路径

ANSIBLE_CONFIG – 覆盖默认的 Ansible 配置文件

ansible.cfg 中的大多数选项还有许多其他可用选项

## 文件

/etc/ansible/hosts – 默认的清单文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
