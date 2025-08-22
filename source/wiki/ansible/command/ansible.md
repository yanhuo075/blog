---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.2 ansible
order: 1
---

# ansible

定义并针对一组主机运行单个任务“playbook”

## 概要

```
usage: ansible [-h] [--version] [-v] [-b] [--become-method BECOME_METHOD]
            [--become-user BECOME_USER]
            [-K | --become-password-file BECOME_PASSWORD_FILE]
            [-i INVENTORY] [--list-hosts] [-l SUBSET] [-P POLL_INTERVAL]
            [-B SECONDS] [-o] [-t TREE] [--private-key PRIVATE_KEY_FILE]
            [-u REMOTE_USER] [-c CONNECTION] [-T TIMEOUT]
            [--ssh-common-args SSH_COMMON_ARGS]
            [--sftp-extra-args SFTP_EXTRA_ARGS]
            [--scp-extra-args SCP_EXTRA_ARGS]
            [--ssh-extra-args SSH_EXTRA_ARGS]
            [-k | --connection-password-file CONNECTION_PASSWORD_FILE] [-C]
            [-D] [-e EXTRA_VARS] [--vault-id VAULT_IDS]
            [-J | --vault-password-file VAULT_PASSWORD_FILES] [-f FORKS]
            [-M MODULE_PATH] [--playbook-dir BASEDIR]
            [--task-timeout TASK_TIMEOUT] [-a MODULE_ARGS] [-m MODULE_NAME]
            pattern
```



## 描述

是一个用于执行“远程操作”的极其简单的工具/框架/API。此命令允许您定义并针对一组主机运行单个任务“playbook”

## 常用选项

- --become-method <BECOME_METHOD>

  要使用的权限提升方法 (默认=sudo)，使用 ansible-doc -t become -l 列出有效的选择。

- --become-password-file <BECOME_PASSWORD_FILE>, --become-pass-file <BECOME_PASSWORD_FILE>

  Become 密码文件

- --become-user <BECOME_USER>

  以该用户身份运行操作 (默认=root)

- --connection-password-file <CONNECTION_PASSWORD_FILE>, --conn-pass-file <CONNECTION_PASSWORD_FILE>

  连接密码文件

- --list-hosts

  输出匹配主机的列表；不执行任何其他操作

- --playbook-dir <BASEDIR>

  由于此工具不使用 Playbook，因此使用此目录作为替代的 Playbook 目录。 这设置了许多功能（包括 roles/ group_vars/ 等）的相对路径。

- --private-key <PRIVATE_KEY_FILE>, --key-file <PRIVATE_KEY_FILE>

  使用此文件来验证连接

- --scp-extra-args <SCP_EXTRA_ARGS>

  指定仅传递给 scp 的额外参数（例如 -l）

- --sftp-extra-args <SFTP_EXTRA_ARGS>

  指定仅传递给 sftp 的额外参数（例如 -f，-l）

- --ssh-common-args <SSH_COMMON_ARGS>

  指定传递给 sftp/scp/ssh 的通用参数（例如 ProxyCommand）

- --ssh-extra-args <SSH_EXTRA_ARGS>

  指定仅传递给 ssh 的额外参数（例如 -R）

- --task-timeout <TASK_TIMEOUT>

  设置任务超时限制（以秒为单位），必须为正整数。

- --vault-id

  要使用的 Vault 标识。可以多次指定此参数。

- --vault-password-file, --vault-pass-file

  Vault 密码文件

- --version

  显示程序版本号、配置文件位置、配置的模块搜索路径、模块位置、可执行文件位置并退出

- -B <SECONDS>, --background <SECONDS>

  异步运行，在 X 秒后失败 (默认=N/A)

- -C, --check

  不进行任何更改；而是尝试预测可能会发生的一些更改

- -D, --diff

  当更改（小）文件和模板时，显示这些文件中的差异；与 --check 一起使用效果很好

- -J, --ask-vault-password, --ask-vault-pass

  要求输入 Vault 密码

- -K, --ask-become-pass

  要求输入权限提升密码

- -M, --module-path

  将冒号分隔的路径预先添加到模块库 (默认={{ ANSIBLE_HOME ~ “/plugins/modules:/usr/share/ansible/plugins/modules” }}). 可以多次指定此参数。

- -P <POLL_INTERVAL>, --poll <POLL_INTERVAL>

  如果使用 -B，则设置轮询间隔（默认=15）

- -T <TIMEOUT>, --timeout <TIMEOUT>

  覆盖连接超时时间（以秒为单位），默认值取决于连接

- -a <MODULE_ARGS>, --args <MODULE_ARGS>

  以空格分隔的 k=v 格式指定操作的选项：-a 'opt1=val1 opt2=val2' 或 JSON 字符串：-a '{"opt1": "val1", "opt2": "val2"}'

- -b, --become

  使用 become 运行操作（不暗示密码提示）

- -c <CONNECTION>, --connection <CONNECTION>

  要使用的连接类型 (默认=ssh)

- -e, --extra-vars

  设置额外的变量，格式为 key=value 或 YAML/JSON，如果是文件名，则在前面加上 @。 此参数可以指定多次。

- -f <FORKS>, --forks <FORKS>

  指定要使用的并行进程数（默认=5）

- -h, --help

  显示此帮助消息并退出

- -i, --inventory, --inventory-file

  指定清单主机路径或逗号分隔的主机列表。--inventory-file 已弃用。此参数可以指定多次。

- -k, --ask-pass

  请求连接密码

- -l <SUBSET>, --limit <SUBSET>

  进一步将选定的主机限制为其他模式

- -m <MODULE_NAME>, --module-name <MODULE_NAME>

  要执行的操作的名称 (默认=command)

- -o, --one-line

  压缩输出

- -t <TREE>, --tree <TREE>

  将输出记录到此目录

- -u <REMOTE_USER>, --user <REMOTE_USER>

  以此用户身份连接 (默认=None)

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件目前评估到 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以指定多次。

## 环境变量

可以指定以下环境变量。

ANSIBLE_INVENTORY – 覆盖默认的 ansible 清单文件

ANSIBLE_LIBRARY – 覆盖默认的 ansible 模块库路径

ANSIBLE_CONFIG – 覆盖默认的 ansible 配置文件

ansible.cfg 中大多数选项都有更多可用选项

## 文件

/etc/ansible/hosts – 默认的清单文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
