---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.5 ansible-galaxy
order: 4
---

# ansible-galaxy

执行各种与角色和集合相关的操作。



## 概要

```
usage: ansible-galaxy [-h] [--version] [-v] TYPE ...
```



## 描述

用于管理 Ansible 角色和集合的命令。

所有 CLI 工具都不设计为自身并发运行。使用外部调度程序和/或锁定机制来确保没有冲突的操作。

## 常用选项

- --version

  显示程序的版本号、配置文件位置、已配置的模块搜索路径、模块位置、可执行文件位置并退出

- -h, --help

  显示此帮助信息并退出

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件目前最多评估到 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。



## 操作

### 集合

对 Ansible Galaxy 集合执行操作。必须与其他操作（如下面列出的 init/install）组合使用。



#### 下载集合

下载集合及其依赖项作为压缩包，以便进行脱机安装。

- --clear-response-cache

  清除现有的服务器响应缓存。

- --no-cache

  不使用服务器响应缓存。

- --pre

  包含预发布版本。默认情况下会忽略语义版本预发布版本

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -n, --no-deps

  不下载列为依赖项的集合。

- -p <DOWNLOAD_PATH>, --download-path <DOWNLOAD_PATH>

  下载集合的目录。

- -r <REQUIREMENTS>, --requirements-file <REQUIREMENTS>

  包含要下载的集合列表的文件。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 初始化集合

创建符合 Galaxy 元数据格式的角色或集合的骨架框架。需要角色或集合名称。集合名称必须采用 <namespace>.<collection> 的格式。

- --collection-skeleton <COLLECTION_SKELETON>

  新集合应基于的集合骨架的路径。

- --init-path <INIT_PATH>

  将在其中创建骨架集合的路径。默认为当前工作目录。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -e, --extra-vars

  设置额外的变量，格式为 key=value 或 YAML/JSON，如果文件名以 @ 开头，则表示为文件路径。此参数可以多次指定。

- -f, --force

  强制覆盖已存在的角色或集合。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 集合构建

构建一个 Ansible Galaxy 集合构件，可以存储在 Ansible Galaxy 等中央仓库中。默认情况下，此命令从当前工作目录构建。可以选择传入集合输入路径（其中包含 galaxy.yml 文件）。

- --output-path <OUTPUT_PATH>

  构建集合的路径。默认为当前工作目录。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -f, --force

  强制覆盖已存在的角色或集合。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 集合发布

将集合发布到 Ansible Galaxy。需要发布集合的 tarball 文件路径。

- --import-timeout <IMPORT_TIMEOUT>

  等待集合导入过程完成的时间。

- --no-wait

  不要等待导入验证结果。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 集合安装

安装一个或多个角色（ansible-galaxy role install）或一个或多个集合（ansible-galaxy collection install）。您可以传入一个列表（角色或集合）或使用下面列出的文件选项（两者互斥）。如果您传入一个列表，它可以是一个名称（将通过 Galaxy API 和 github 下载），也可以是一个本地 tar 归档文件。

- --clear-response-cache

  清除现有的服务器响应缓存。

- --disable-gpg-verify

  从 Galaxy 服务器安装集合时禁用 GPG 签名验证。

- --force-with-deps

  强制覆盖现有集合及其依赖项。

- --ignore-signature-status-code

  ==SUPPRESS==。此参数可以多次指定。

- --ignore-signature-status-codes

  一个用空格分隔的状态码列表，在签名验证期间忽略这些状态码（例如，NO_PUBKEY FAILURE）。可以在 L(https://github.com/gpg/gnupg/blob/master/doc/DETAILS#general-status-codes).Note) 查看这些选项的描述。注意：在位置参数之后指定这些参数，或使用 – 分隔它们。此参数可以多次指定。

- --keyring <KEYRING>

  签名验证期间使用的密钥环。

- --no-cache

  不使用服务器响应缓存。

- --offline

  安装集合构件（tarball）而不联系任何分发服务器。这不适用于远程 Git 仓库中的集合或指向远程 tarball 的 URL。

- --pre

  包含预发布版本。默认情况下会忽略语义版本预发布版本

- --required-valid-signature-count <REQUIRED_VALID_SIGNATURE_COUNT>

  必须成功验证集合的签名数量。这应该是一个正整数，或者 -1 表示必须使用所有签名来验证集合。在值前加上 + 表示如果找不到集合的有效签名则失败（例如 +all）。

- --signature

  一个额外的签名源，用于在从 Galaxy 服务器安装集合之前验证 MANIFEST.json 的真实性。与位置集合名称一起使用（与 –requirements-file 互斥）。此参数可以多次指定。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -U, --upgrade

  升级已安装的集合构件。这还将更新依赖项，除非提供 –no-deps。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -f, --force

  强制覆盖已存在的角色或集合。

- -i, --ignore-errors

  忽略安装期间的错误并继续下一个指定的集合。这不会忽略依赖冲突错误。

- -n, --no-deps

  不下载列为依赖项的集合。

- -p <COLLECTIONS_PATH>, --collections-path <COLLECTIONS_PATH>

  包含您集合的目录的路径。

- -r <REQUIREMENTS>, --requirements-file <REQUIREMENTS>

  包含要安装的集合列表的文件。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 集合列表

列出已安装的集合或角色。

- --format <OUTPUT_FORMAT>

  显示集合列表的格式

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -p, --collections-path

  除默认 COLLECTIONS_PATHS 外，还要搜索集合的一个或多个目录。用“:”分隔多个路径。此参数可以多次指定。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 集合校验

将校验和与服务器上找到的集合和已安装的副本进行比较。这不会校验依赖项。

- --ignore-signature-status-code

  ==SUPPRESS==。此参数可以多次指定。

- --ignore-signature-status-codes

  一个用空格分隔的状态码列表，在签名验证期间忽略这些状态码（例如，NO_PUBKEY FAILURE）。可以在 L(https://github.com/gpg/gnupg/blob/master/doc/DETAILS#general-status-codes).Note) 查看这些选项的描述。注意：在位置参数之后指定这些参数，或使用 – 分隔它们。此参数可以多次指定。

- --keyring <KEYRING>

  签名验证期间使用的密钥环。

- --offline

  在不联系服务器获取规范清单哈希的情况下，在本地验证集合完整性。

- --required-valid-signature-count <REQUIRED_VALID_SIGNATURE_COUNT>

  必须成功验证集合的签名数量。这应该是一个正整数或 all，表示必须使用所有签名来验证集合。在值前加 + 表示如果找不到集合的有效签名则失败（例如 +all）。

- --signature

  在使用 MANIFEST.json 验证集合其余内容之前，用于验证 MANIFEST.json 真实性的附加签名源。与位置集合名称结合使用（与 –requirements-file 互斥）。此参数可以多次指定。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -i, --ignore-errors

  忽略验证期间的错误并继续执行下一个指定的集合。

- -p, --collections-path

  除默认 COLLECTIONS_PATHS 外，还要搜索集合的一个或多个目录。用“:”分隔多个路径。此参数可以多次指定。

- -r <REQUIREMENTS>, --requirements-file <REQUIREMENTS>

  包含要验证的集合列表的文件。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



### 角色

对 Ansible Galaxy 角色执行操作。必须与下面列出的其他操作（如删除/安装/初始化）结合使用。



#### 角色初始化

创建符合 Galaxy 元数据格式的角色或集合的骨架框架。需要角色或集合名称。集合名称必须采用 <namespace>.<collection> 的格式。

- --init-path <INIT_PATH>

  将在其中创建骨架角色的路径。默认为当前工作目录。

- --offline

  创建角色时不查询 Galaxy API。

- --role-skeleton <ROLE_SKELETON>

  新角色应基于的角色骨架的路径。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- --type <ROLE_TYPE>

  使用替代角色类型进行初始化。有效类型包括：“container”、“apb”和“network”。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -e, --extra-vars

  设置额外的变量，格式为 key=value 或 YAML/JSON，如果文件名以 @ 开头，则表示为文件路径。此参数可以多次指定。

- -f, --force

  强制覆盖已存在的角色或集合。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 删除角色

从本地系统中删除作为参数传递的角色列表。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -p, --roles-path

  包含角色的目录路径。默认值为第一个可写目录，通过 DEFAULT_ROLES_PATH 配置：{{ ANSIBLE_HOME ~ “/roles:/usr/share/ansible/roles:/etc/ansible/roles” }}。此参数可以多次指定。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 删除角色

从 Ansible Galaxy 删除角色。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 列出角色

列出已安装的集合或角色。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -p, --roles-path

  包含角色的目录路径。默认值为第一个可写目录，通过 DEFAULT_ROLES_PATH 配置：{{ ANSIBLE_HOME ~ “/roles:/usr/share/ansible/roles:/etc/ansible/roles” }}。此参数可以多次指定。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 搜索角色

在 Ansible Galaxy 服务器上搜索角色。

- --author <AUTHOR>

  GitHub 用户名。

- --galaxy-tags <GALAXY_TAGS>

  要过滤的 Galaxy 标签列表。

- --platforms <PLATFORMS>

  要过滤的操作系统平台列表。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 导入角色

用于将角色导入 Ansible Galaxy。

- --branch <REFERENCE>

  要导入的分支名称。默认为仓库的默认分支（通常为 master）。

- --no-wait

  不等待导入结果。

- --role-name <ROLE_NAME>

  角色的名称，如果与仓库名称不同。

- --status

  检查给定 github_user/github_repo 的最新导入请求的状态。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 角色设置

为 Ansible Galaxy 角色设置来自 Github 或 Travis 的集成。

- --list

  列出所有集成。

- --remove <REMOVE_ID>

  删除与提供的 ID 值匹配的集成。使用 –list 查看 ID 值。

- --timeout <TIMEOUT>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -p, --roles-path

  包含角色的目录路径。默认值为第一个可写目录，通过 DEFAULT_ROLES_PATH 配置：{{ ANSIBLE_HOME ~ “/roles:/usr/share/ansible/roles:/etc/ansible/roles” }}。此参数可以多次指定。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 角色信息

打印已安装角色的详细信息以及来自 Galaxy API 的信息。

- --offline

  创建角色时不查询 Galaxy API。

- --timeout <超时时间>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -p, --roles-path

  包含角色的目录路径。默认值为第一个可写目录，通过 DEFAULT_ROLES_PATH 配置：{{ ANSIBLE_HOME ~ “/roles:/usr/share/ansible/roles:/etc/ansible/roles” }}。此参数可以多次指定。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL



#### 角色安装

安装一个或多个角色（ansible-galaxy role install）或一个或多个集合（ansible-galaxy collection install）。您可以传入一个列表（角色或集合）或使用下面列出的文件选项（两者互斥）。如果您传入一个列表，它可以是一个名称（将通过 Galaxy API 和 github 下载），也可以是一个本地 tar 归档文件。

- --force-with-deps

  强制覆盖现有角色及其依赖项。

- --timeout <超时时间>

  等待 Galaxy 服务器操作的时间，默认为 60 秒。

- --token <API_KEY>, --api-key <API_KEY>

  Ansible Galaxy API 密钥，可在 https://galaxy.ansible.com/me/preferences 找到。

- -c, --ignore-certs

  忽略 SSL 证书验证错误。

- -f, --force

  强制覆盖已存在的角色或集合。

- -g, --keep-scm-meta

  打包角色时使用 tar 代替 scm 归档选项。

- -i, --ignore-errors

  忽略错误并继续安装下一个指定的 roles。

- -n, --no-deps

  不下载列为依赖项的角色。

- -p, --roles-path

  包含角色的目录路径。默认值为第一个可写目录，通过 DEFAULT_ROLES_PATH 配置：{{ ANSIBLE_HOME ~ “/roles:/usr/share/ansible/roles:/etc/ansible/roles” }}。此参数可以多次指定。

- -r <需求文件>, --role-file <需求文件>

  包含要安装的角色列表的文件。

- -s <API_SERVER>, --server <API_SERVER>

  Galaxy API 服务器 URL

## 环境变量

可以指定以下环境变量。

ANSIBLE_CONFIG – 覆盖默认的 Ansible 配置文件

ansible.cfg 中还有更多选项。

## 文件

/etc/ansible/ansible.cfg – 配置文件，如果存在则使用

~/.ansible.cfg – 用户配置文件，如果存在则覆盖默认配置
