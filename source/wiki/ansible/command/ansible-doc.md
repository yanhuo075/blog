---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.5 ansible-doc
order: 4
---

# ansible-doc

插件文档工具



## 概要

```
usage: ansible-doc [-h] [--version] [-v] [-M MODULE_PATH]
                [--playbook-dir BASEDIR]
                [-t {become,cache,callback,cliconf,connection,httpapi,inventory,lookup,netconf,shell,vars,module,strategy,test,filter,role,keyword}]
                [-j] [-r ROLES_PATH]
                [-e ENTRY_POINT | -s | -F | -l | --metadata-dump]
                [--no-fail-on-errors]
                [plugin ...]
```



## 描述

显示 Ansible 库中已安装模块的信息。它显示插件及其简短描述的简洁列表，提供其 DOCUMENTATION 字符串的打印输出，并且可以创建一个简短的“代码片段”，可以粘贴到 playbook 中。

## 常用选项

- --metadata-dump

  仅供内部使用 转储所有条目的 json 元数据，忽略其他选项。

- --no-fail-on-errors

  仅供内部使用 仅用于 –metadata-dump。 不要因错误而失败。 在 JSON 中报告错误消息。

- --playbook-dir <BASEDIR>

  由于此工具不使用 playbook，请将其用作替代 playbook 目录。 这将为许多功能设置相对路径，包括 roles/ group_vars/ 等。

- --version

  显示程序的版本号、配置文件位置、配置的模块搜索路径、模块位置、可执行文件位置并退出

- -F, --list_files

  显示插件名称及其源文件，不显示摘要（暗示 –list）。 提供的参数将用于过滤，可以是命名空间或完整的集合名称。

- -M, --module-path

  将冒号分隔的路径添加到模块库的前面（默认值={{ ANSIBLE_HOME ~ “/plugins/modules:/usr/share/ansible/plugins/modules” }}）。 此参数可以指定多次。

- -e <ENTRY_POINT>, --entry-point <ENTRY_POINT>

  选择角色（多个）的入口点。

- -h, --help

  显示此帮助消息并退出

- -j, --json

  将输出更改为 json 格式。

- -l, --list

  列出可用的插件。 提供的参数将用于过滤，可以是命名空间或完整的集合名称。

- -r, --roles-path

  包含角色的目录的路径。 此参数可以指定多次。

- -s, --snippet

  显示这些插件类型的 playbook 代码片段：inventory、lookup、module

- -t <TYPE>, --type <TYPE>

  选择插件类型（默认为“module”）。 可用的插件类型为：（‘become’, ‘cache’, ‘callback’, ‘cliconf’, ‘connection’, ‘httpapi’, ‘inventory’, ‘lookup’, ‘netconf’, ‘shell’, ‘vars’, ‘module’, ‘strategy’, ‘test’, ‘filter’, ‘role’, ‘keyword’)

- -v, --verbose

  使 Ansible 打印更多调试消息。 添加多个 -v 将增加详细程度，内置插件当前评估最多 -vvvvvv。 一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。 此参数可以指定多次。

## 环境变量

可以指定以下环境变量

ANSIBLE_LIBRARY – 覆盖默认的 ansible 模块库路径

ANSIBLE_CONFIG – 覆盖默认的 ansible 配置文件

ansible.cfg 中的大多数选项都可以使用更多环境变量

## 文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
