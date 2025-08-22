---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.8 ansible-pull
order: 7
---

# ansible-pull

从 VCS 仓库拉取 playbook 并将其在目标主机上执行



## 概要

```
usage: ansible-pull [-h] [--version] [-v] [--private-key PRIVATE_KEY_FILE]
                 [-u REMOTE_USER] [-c CONNECTION] [-T TIMEOUT]
                 [--ssh-common-args SSH_COMMON_ARGS]
                 [--sftp-extra-args SFTP_EXTRA_ARGS]
                 [--scp-extra-args SCP_EXTRA_ARGS]
                 [--ssh-extra-args SSH_EXTRA_ARGS]
                 [-k | --connection-password-file CONNECTION_PASSWORD_FILE]
                 [--vault-id VAULT_IDS]
                 [-J | --vault-password-file VAULT_PASSWORD_FILES]
                 [-e EXTRA_VARS] [-t TAGS] [--skip-tags SKIP_TAGS]
                 [-i INVENTORY] [--list-hosts] [-l SUBSET] [-M MODULE_PATH]
                 [-K | --become-password-file BECOME_PASSWORD_FILE]
                 [--purge] [-o] [-s SLEEP] [-f] [-d DEST] [-U URL] [--full]
                 [-C CHECKOUT] [--accept-host-key] [-m MODULE_NAME]
                 [--verify-commit] [--clean] [--track-subs] [--check]
                 [--diff]
                 [playbook.yml ...]
```



## 描述

用于在每个受管节点上拉取 Ansible 的远程副本，每个副本都设置为通过 cron 运行并通过源代码库更新 playbook 源代码。这将 Ansible 的默认 推送 架构反转为 拉取 架构，后者具有几乎无限的扩展潜力。

没有一个 CLI 工具设计为与自身同时运行，您应该使用外部调度程序和/或锁定来确保没有冲突的操作。

可以调整设置 playbook 以更改 cron 频率、日志位置和 ansible-pull 的参数。这对于极端水平扩展和定期修复都非常有用。使用 ‘fetch’ 模块来检索 ansible-pull 运行的日志将是收集和分析 ansible-pull 的远程日志的极好方法。

## 常用选项

- --accept-host-key

  如果尚未添加，则添加仓库 URL 的主机密钥

- --become-password-file <BECOME_PASSWORD_FILE>, --become-pass-file <BECOME_PASSWORD_FILE>

  授权密码文件

- --check

  不进行任何更改；而是尝试预测可能发生的某些更改

- --clean

  工作存储库中的已修改文件将被丢弃

- --connection-password-file <CONNECTION_PASSWORD_FILE>, --conn-pass-file <CONNECTION_PASSWORD_FILE>

  连接密码文件

- --diff

  更改（小）文件和模板时，显示这些文件的差异；与 –check 配合使用效果很好

- --full

  执行完整克隆，而不是浅克隆。

- --list-hosts

  输出匹配主机的列表；不执行任何其他操作

- --private-key <PRIVATE_KEY_FILE>, --key-file <PRIVATE_KEY_FILE>

  使用此文件来验证连接

- --purge

  在 playbook 运行后清除签出

- --scp-extra-args <SCP_EXTRA_ARGS>

  指定要传递给 scp 的额外参数（例如 -l）

- --sftp-extra-args <SFTP_EXTRA_ARGS>

  指定要传递给 sftp 的额外参数（例如 -f，-l）

- --skip-tags

  仅运行其标签与这些值不匹配的 play 和任务。此参数可以多次指定。

- --ssh-common-args <SSH_COMMON_ARGS>

  指定要传递给 sftp/scp/ssh 的常用参数（例如 ProxyCommand）

- --ssh-extra-args <SSH_EXTRA_ARGS>

  指定要传递给 ssh 的额外参数（例如 -R）

- --track-subs

  子模块将跟踪最新的更改。这等效于为 git submodule update 指定 –remote 标志

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- --verify-commit

  验证签出的提交的 GPG 签名，如果失败则中止运行 playbook。这需要相应的 VCS 模块来支持此类操作

- --version

  显示程序的版本号、配置文件位置、配置的模块搜索路径、模块位置、可执行文件位置并退出

- -C <CHECKOUT>, --checkout <CHECKOUT>

  要签出的分支/标签/提交。默认为存储库模块的行为。

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码

- -K, --ask-become-pass

  询问权限提升密码

- -M, --module-path

  将冒号分隔的路径添加到模块库的前面（默认值为 {{ ANSIBLE_HOME ~ “/plugins/modules:/usr/share/ansible/plugins/modules” }}）。此参数可以多次指定。

- -T <TIMEOUT>, --timeout <TIMEOUT>

  以秒为单位覆盖连接超时（默认值取决于连接）

- -U <URL>, --url <URL>

  playbook 仓库的 URL

- -c <CONNECTION>, --connection <CONNECTION>

  使用的连接类型 (默认值=ssh)

- -d <DEST>, --directory <DEST>

  Ansible 将签出代码库到的目录路径。

- -e, --extra-vars

  设置额外的变量，格式为 key=value 或 YAML/JSON，如果文件名以 @ 开头。此参数可以多次指定。

- -f, --force

  即使无法更新代码库，也运行剧本。

- -h, --help

  显示此帮助消息并退出。

- -i, --inventory, --inventory-file

  指定清单主机路径或以逗号分隔的主机列表。 --inventory-file 已弃用。此参数可以多次指定。

- -k, --ask-pass

  询问连接密码。

- -l <SUBSET>, --limit <SUBSET>

  将选定的主机进一步限制到附加模式。

- -m <MODULE_NAME>, --module-name <MODULE_NAME>

  代码库模块名称，Ansible 将使用它来签出代码库。选项包括 ('git', 'subversion', 'hg', 'bzr')。默认为 git。

- -o, --only-if-changed

  只有在代码库已更新时才运行剧本。

- -s <SLEEP>, --sleep <SLEEP>

  在开始之前休眠随机时间间隔（介于 0 到 n 秒之间）。这是一种分散 git 请求的有用方法。

- -t, --tags

  仅运行带有这些值的标记的剧集和任务。此参数可以多次指定。

- -u <REMOTE_USER>, --user <REMOTE_USER>

  以该用户身份连接 (默认值=None)

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件当前最多评估到 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。

## 参数

- playbook.yml

  要作为 Ansible 剧本运行的 YAML 格式文件之一的名称。这可以是签出中的相对路径。默认情况下，Ansible 将根据主机的完全限定域名、主机主机名以及最终名为 local.yml 的剧本查找剧本。

## 环境变量

可以指定以下环境变量。

ANSIBLE_INVENTORY – 覆盖默认的 Ansible 清单文件。

ANSIBLE_LIBRARY – 覆盖默认的 Ansible 模块库路径。

ANSIBLE_CONFIG – 覆盖默认的 Ansible 配置文件。

ansible.cfg 中的大多数选项都提供了更多选项。

## 文件

/etc/ansible/hosts – 默认清单文件。

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用。

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置。
