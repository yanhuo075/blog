---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.3 ansible-config
order: 2
---

# ansible-config

查看 ansible 配置。

## 概要

```
usage: ansible-config [-h] [--version] [-v] {list,dump,view,init} ...
```



## 描述

配置命令行类



## 常用选项

- --version

  显示程序的版本号、配置文件位置、配置的模块搜索路径、模块位置、可执行文件位置并退出

- -h, --help

  显示此帮助消息并退出

- -v, --verbose

  导致 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件当前评估最多 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。



## 操作

### list

列出并输出可用的配置

- --format <FORMAT>, -f <FORMAT>

  列表的输出格式

- -c <CONFIG_FILE>, --config <CONFIG_FILE>

  配置文件的路径，默认为优先级中找到的第一个文件。

- -t <TYPE>, --type <TYPE>

  筛选到特定的插件类型。



### dump

显示当前设置，如果指定则合并 ansible.cfg

- --format <FORMAT>, -f <FORMAT>

  dump 的输出格式

- --only-changed, --changed-only

  仅显示与默认值不同的配置

- -c <CONFIG_FILE>, --config <CONFIG_FILE>

  配置文件的路径，默认为优先级中找到的第一个文件。

- -t <TYPE>, --type <TYPE>

  筛选到特定的插件类型。



### view

显示当前配置文件

- -c <CONFIG_FILE>, --config <CONFIG_FILE>

  配置文件的路径，默认为优先级中找到的第一个文件。

- -t <TYPE>, --type <TYPE>

  筛选到特定的插件类型。



### init

创建初始配置

- --disabled

  在所有条目前面添加注释字符以禁用它们

- --format <FORMAT>, -f <FORMAT>

  init 的输出格式

- -c <CONFIG_FILE>, --config <CONFIG_FILE>

  配置文件的路径，默认为优先级中找到的第一个文件。

- -t <TYPE>, --type <TYPE>

  筛选到特定的插件类型。

## 环境变量

可以指定以下环境变量。

ANSIBLE_CONFIG – 覆盖默认的 ansible 配置文件

ansible.cfg 中大多数选项都有更多可用环境变量

## 文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
