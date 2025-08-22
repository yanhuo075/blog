---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.6 ansible-inventory
order: 5
---

# ansible-inventory

显示 Ansible 清单信息，默认情况下它使用清单脚本 JSON 格式



## 概要

```
usage: ansible-inventory [-h] [--version] [-v] [-i INVENTORY] [-l SUBSET]
                      [--vault-id VAULT_IDS]
                      [-J | --vault-password-file VAULT_PASSWORD_FILES]
                      [--playbook-dir BASEDIR] [-e EXTRA_VARS] [--list]
                      [--host HOST] [--graph] [-y] [--toml] [--vars]
                      [--export] [--output OUTPUT_FILE]
                      [group]
```



## 描述

用于显示或转储 Ansible 所看到的已配置清单

## 常用选项

- --export

  在执行 –list 时，以适合导出优化的方式表示，而不是 Ansible 处理它的准确表示

- --graph

  创建清单图，如果提供模式，它必须是有效的组名。它将忽略 limit

- --host <HOST>

  输出特定主机信息，作为清单脚本。它将忽略 limit

- --list

  输出所有主机信息，作为清单脚本

- --output <OUTPUT_FILE>

  在执行 –list 时，将清单发送到文件而不是屏幕

- --playbook-dir <BASEDIR>

  由于此工具不使用 playbook，因此将其用作替代 playbook 目录。这将设置许多功能的相对路径，包括 roles/ group_vars/ 等。

- --toml

  使用 TOML 格式而不是默认的 JSON，对于 –graph 将被忽略

- --vars

  向图形显示添加变量，除非与 –graph 一起使用，否则将被忽略

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- --version

  显示程序的版本号、配置文件位置、已配置的模块搜索路径、模块位置、可执行文件位置并退出

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码

- -e, --extra-vars

  设置附加变量为 key=value 或 YAML/JSON，如果文件名以 @ 开头。此参数可以多次指定。

- -h, --help

  显示此帮助消息并退出

- -i, --inventory, --inventory-file

  指定清单主机路径或以逗号分隔的主机列表。–inventory-file 已弃用。此参数可以多次指定。

- -l <SUBSET>, --limit <SUBSET>

  将选定的主机进一步限制到其他模式

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件当前最多评估到 -vvvvvv。合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。

- -y, --yaml

  使用 YAML 格式而不是默认的 JSON，对于 –graph 将被忽略

## 参数

- group

  清单中组的名称，在使用 –graph 时相关

## 环境变量

可以指定以下环境变量。

ANSIBLE_INVENTORY – 覆盖默认的 ansible 清单文件

ANSIBLE_CONFIG – 覆盖默认的 ansible 配置文件

ansible.cfg 中的大多数选项都提供更多环境变量。

## 文件

/etc/ansible/hosts – 默认清单文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置

