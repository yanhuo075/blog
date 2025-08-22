---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 9.2 Galaxy 开发者指南
order: 1
---

# Galaxy 开发者指南

你可以在 Galaxy 上托管集合和角色，以便与 Ansible 社区共享。Galaxy 内容以预先打包的工作单元格式呈现，例如角色和集合。你可以创建角色来配置基础设施、部署应用程序以及执行你每天执行的所有任务。更进一步，你可以创建集合，其中提供了全面的自动化包，其中可能包括多个 playbook、角色、模块和插件。

- 为 Galaxy 创建集合
- 为 Galaxy 创建角色
  - 强制
  - 容器启用
  - 使用自定义角色骨架
  - 使用 Galaxy 进行身份验证
  - 导入角色
  - 删除角色
  - Travis 集成



## 为 Galaxy 创建集合

集合是 Ansible 内容的分发格式。你可以使用集合来打包和分发 playbook、角色、模块和插件。你可以通过 Ansible Galaxy 发布和使用集合。

有关如何创建集合的详细信息，请参阅开发集合。



## 为 Galaxy 创建角色

使用 init 命令初始化新角色的基本结构，从而节省创建角色所需的各种目录和 main.yml 文件的时间

```
$ ansible-galaxy role init role_name
```



以上命令将在当前工作目录中创建以下目录结构

```
role_name/
    README.md
    defaults/
        main.yml
    files/
    handlers/
        main.yml
    meta/
        main.yml
    tasks/
        main.yml
    templates/
    tests/
        inventory
        test.yml
    vars/
        main.yml
```



如果要为角色创建存储库，则存储库根目录应为 role_name。

### 强制

如果当前工作目录中已存在与角色名称匹配的目录，则 init 命令将导致错误。要忽略该错误，请使用 --force 选项。强制将创建上述子目录和文件，替换任何匹配的内容。

### 容器启用

如果要创建容器启用角色，请将 --type container 传递给 ansible-galaxy init。这将创建与上述相同的目录结构，但使用适用于容器启用角色的默认文件填充它。例如，README.md 具有略微不同的结构，.travis.yml 文件使用 Ansible Container 测试角色，并且 meta 目录包含 container.yml 文件。

### 使用自定义角色骨架

可以按如下方式提供自定义角色骨架目录

```
$ ansible-galaxy init --role-skeleton=/path/to/skeleton role_name
```



当提供骨架时，init 将

- 将所有文件和目录从骨架复制到新角色
- 在 templates 文件夹外部找到的任何 .j2 文件都将呈现为模板。目前唯一有用的变量是 role_name
- 不会复制 .git 文件夹和任何 .git_keep 文件

或者，可以使用 ansible.cfg 配置 role_skeleton 和忽略的文件

```
[galaxy]
role_skeleton = /path/to/skeleton
role_skeleton_ignore = ^.git$,^.*/.git_keep$
```



### 使用 Galaxy 进行身份验证

使用 import、delete 和 setup 命令来管理你在 Galaxy 网站上的角色需要以 API 密钥的形式进行身份验证，你必须在 Galaxy 网站上创建一个帐户。

要创建身份验证令牌

1. 单击 集合 > API 令牌。
2. 单击 加载令牌，然后复制它。
3. 将你的令牌保存在 GALAXY_TOKEN_PATH 中设置的路径中。

### 导入角色

import``命令 需要 你 使用 API 令牌 进行 身份验证。 你 可以将 它 包含 在 你的 ``ansible.cfg 文件中，或者使用 --token 命令选项。你只允许删除你在 GitHub 中具有存储库访问权限的角色。

要导入新角色

```
$ ansible-galaxy role import github_user github_repo
```



默认情况下，该命令将等待 Galaxy 完成导入过程，并在导入过程进行时显示结果

```
Successfully submitted import request 41
Starting import 41: role_name=myrole repo=githubuser/ansible-role-repo ref=
Retrieving GitHub repo githubuser/ansible-role-repo
Accessing branch: devel
Parsing and validating meta/main.yml
Parsing galaxy_tags
Parsing platforms
Adding dependencies
Parsing and validating README.md
Adding repo tags as role versions
Import completed
Status SUCCESS : warnings=0 errors=0
```



有关其他命令选项，请参阅 ansible-galaxy。

### 删除角色

delete 命令要求你使用 API 令牌进行身份验证。你可以将其包含在你的 ansible.cfg 文件中，或者使用 --token 命令选项。你只允许删除你在 GitHub 中具有存储库访问权限的角色。

使用以下命令删除角色

```
$ ansible-galaxy role delete github_user github_repo
```



这只会从 Galaxy 中删除角色。它不会删除或更改实际的 GitHub 存储库。

### Travis 集成

你可以在 Galaxy 中的角色和 Travis 之间创建集成或连接。建立连接后，Travis 中的构建将自动触发 Galaxy 中的导入，使用有关角色的最新信息更新搜索索引。

你可以使用带有 API 令牌的 setup 命令创建集成。你还需要一个 Travis 帐户和你的 Travis 令牌。准备就绪后，使用以下命令创建集成

```
$ ansible-galaxy role setup travis github_user github_repo xxx-travis-token-xxx
```



setup 命令需要你的 Travis 令牌，但该令牌不存储在 Galaxy 中。它与 GitHub 用户名和存储库一起使用，以创建 Travis 文档中描述的哈希值。该哈希值存储在 Galaxy 中，用于验证从 Travis 收到的通知。

setup 命令使 Galaxy 能够响应通知。要配置 Travis 在你的存储库上运行构建并发送通知，请按照 Travis 入门指南进行操作。

要指示 Travis 在构建完成后通知 Galaxy，请将以下内容添加到你的 .travis.yml 文件中

```
notifications:
    webhooks: https://galaxy.ansible.com/api/v1/notifications/
```



#### 列出 Travis 集成

使用 --list 选项显示你的 Travis 集成

```
$ ansible-galaxy setup --list travis github_user github_repo xxx-travis-token-xxx


ID         Source     Repo
---------- ---------- ----------
2          travis     github_user/github_repo
1          travis     github_user/github_repo
```



#### 移除 Travis 集成

使用 --remove 选项禁用和移除 Travis 集成

> ```
> $ ansible-galaxy role setup --remove ID
> ```

提供要禁用的集成的 ID。你可以使用 --list 选项找到该 ID。
