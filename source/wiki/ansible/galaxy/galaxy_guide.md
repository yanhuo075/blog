---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 9.1 Galaxy 用户指南
order: 0
---

# Galaxy 用户指南

Ansible Galaxy 指的是 Galaxy 网站，这是一个免费的网站，用于查找、下载和共享社区开发的集合和角色。

使用 Galaxy 通过 Ansible 社区的优质内容快速启动您的自动化项目。Galaxy 提供了预先打包的工作单元，例如 角色 和 集合。集合格式提供了全面的自动化包，其中可能包括多个 playbook、角色、模块和插件。有关 Galaxy 的完整详细信息，请参阅 Galaxy 文档。

- 在 Galaxy 上查找集合
- 在 Galaxy 上查找角色
  - 获取关于角色的更多信息
- 从 Galaxy 安装角色
  - 安装角色
  - 安装特定版本的角色
  - 从文件安装多个角色
  - 从同一个 requirements.yml 文件安装角色和集合
  - 从多个文件安装多个角色
  - 依赖关系
  - 列出已安装的角色
  - 删除已安装的角色



## 在 Galaxy 上查找集合

在 Galaxy 上查找集合

1. 单击左侧导航栏中的 集合 > 集合。
2. 键入您的搜索词。您可以按关键字、标签和命名空间进行筛选。

Galaxy 会显示与您的搜索条件匹配的集合列表。

有关安装和使用集合的完整详细信息，请参阅 使用 Ansible 集合。



## 在 Galaxy 上查找角色

查找独立角色（即不属于集合的角色）

1. 单击左侧导航栏中的 角色 > 角色。
2. 键入您的搜索词。您可以按关键字、标签和命名空间进行筛选。

Galaxy 会显示与您的搜索条件匹配的角色列表。

您可以选择使用 ansible-galaxy CLI 命令按标签、平台、作者和多个关键字搜索 Galaxy 数据库。

```
$ ansible-galaxy role search elasticsearch --author geerlingguy
```



搜索命令将返回与您的搜索匹配的前 1000 个结果列表

```
Found 6 roles matching your search:

 Name                             Description
 ----                             -----------
geerlingguy.elasticsearch         Elasticsearch for Linux.
geerlingguy.elasticsearch-curator Elasticsearch curator for Linux.
geerlingguy.filebeat              Filebeat for Linux.
geerlingguy.fluentd               Fluentd for Linux.
geerlingguy.kibana                Kibana for Linux.
```



### 获取关于角色的更多信息

使用 info 命令查看关于特定角色的更多详细信息

```
$ ansible-galaxy role info username.role_name
```



这将返回 Galaxy 中找到的关于该角色的一切信息

```
Role: username.role_name
    description: Installs and configures a thing, a distributed, highly available NoSQL thing.
    active: True
    commit: c01947b7bc89ebc0b8a2e298b87ab416aed9dd57
    commit_message: Adding travis
    commit_url: https://github.com/username/repo_name/commit/c01947b7bc89ebc0b8a2e298b87ab
    company: My Company, Inc.
    created: 2015-12-08T14:17:52.773Z
    download_count: 1
    forks_count: 0
    github_branch: main
    github_repo: repo_name
    github_user: username
    id: 6381
    is_valid: True
    issue_tracker_url:
    license: Apache
    min_ansible_version: 2.15
    modified: YYYY-MM-DDTHH:MM:SS.000Z
    namespace: username
    open_issues_count: 0
    path: /Users/username/projects/roles
    role_type: ANS
    stargazers_count: 0
    travis_status_url: https://travis-ci.org/username/repo_name.svg?branch=main
```





## 从 Galaxy 安装角色

ansible-galaxy 命令与 Ansible 捆绑在一起，您可以使用它从 Galaxy 或直接从基于 Git 的 SCM 安装角色。您还可以使用它来创建新角色、删除角色或在 Galaxy 网站上执行任务。

默认情况下，命令行工具使用服务器地址 https://galaxy.ansible.com 与 Galaxy 网站 API 通信。如果您运行自己的内部 Galaxy 服务器并希望使用它而不是默认服务器，请传递 --server 选项，后跟此 Galaxy 服务器的地址。您可以通过在 ansible.cfg 文件中设置 Galaxy 服务器值来永久设置此选项。有关在 ansible.cfg 中设置值的详细信息，请参阅 GALAXY_SERVER。

### 安装角色

使用 ansible-galaxy 命令从 Galaxy 网站 下载角色

```
$ ansible-galaxy role install namespace.role_name
```



#### 设置角色安装位置

默认情况下，Ansible 将角色下载到默认路径列表 ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles 中第一个可写入的目录。这会将角色安装在运行 ansible-galaxy 的用户的主目录中。

您可以使用以下选项之一来覆盖此设置

- 在您的会话中设置环境变量 ANSIBLE_ROLES_PATH。
- 对 ansible-galaxy 命令使用 --roles-path 选项。
- 在 ansible.cfg 文件中定义 roles_path。

以下示例演示了如何使用 --roles-path 将角色安装到当前工作目录中

```
$ ansible-galaxy role install --roles-path . geerlingguy.apache
```



### 安装特定版本的角色

当 Galaxy 服务器导入角色时，它会将任何与 语义版本格式匹配的 Git 标签作为版本导入。反过来，您可以通过指定导入的标签之一来下载特定版本的角色。

查看角色的可用版本

1. 在 Galaxy 搜索页面上找到该角色。
2. 单击名称以查看更多详细信息，包括可用的版本。

要从 Galaxy 安装特定版本的角色，请附加一个逗号和 GitHub 发布标签的值。例如

```
$ ansible-galaxy role install geerlingguy.apache,3.2.0
```



也可以直接指向 Git 存储库并指定分支名称或提交哈希作为版本。例如，以下命令将安装特定的提交

```
$ ansible-galaxy role install git+https://github.com/geerlingguy/ansible-role-apache.git,0b7cd353c0250e87a26e0499e59e7fd265cc2f25
```



### 从文件安装多个角色

您可以通过在 requirements.yml 文件中包含角色来安装多个角色。该文件的格式为 YAML，文件扩展名必须是 .yml 或 .yaml。

使用以下命令安装 requirements.yml: 中包含的角色：

```
$ ansible-galaxy install -r requirements.yml
```



同样，扩展名很重要。如果省略 .yml 扩展名，则 ansible-galaxy CLI 会假定该文件采用较旧的、现已弃用的“基本”格式。

文件中的每个角色都将具有以下一个或多个属性

> - src
>
>   角色的来源。如果从 Galaxy 下载，请使用 namespace.role_name 格式；否则，请提供指向基于 Git 的 SCM 内的存储库的 URL。请参见以下示例。这是一个必需属性。
>
> - scm
>
>   指定 SCM。在撰写本文时，只允许使用 git 或 hg。请参见以下示例。默认为 git。
>
> - version
>
>   要下载的角色版本。提供发布标签值、提交哈希或分支名称。默认为存储库中设置为默认值的分支，否则默认为 master。
>
> - name
>
>   将角色下载到特定名称。从 Galaxy 下载时默认为 Galaxy 名称，否则默认为存储库名称。

使用以下示例作为在 requirements.yml 中指定角色的指南

```
# from galaxy
- name: yatesr.timezone

# from locally cloned Git repository (git+file:// requires full paths)
- src: git+file:///home/bennojoy/nginx

# from GitHub
- src: https://github.com/bennojoy/nginx

# from GitHub, overriding the name and specifying a specific tag
- name: nginx_role
  src: https://github.com/bennojoy/nginx
  version: main

# from GitHub, specifying a specific commit hash
- src: https://github.com/bennojoy/nginx
  version: "ee8aa41"

# from a webserver, where the role is packaged in a tar.gz
- name: http-role-gz
  src: https://some.webserver.example.com/files/main.tar.gz

# from a webserver, where the role is packaged in a tar.bz2
- name: http-role-bz2
  src: https://some.webserver.example.com/files/main.tar.bz2

# from a webserver, where the role is packaged in a tar.xz (Python 3.x only)
- name: http-role-xz
  src: https://some.webserver.example.com/files/main.tar.xz

# from Bitbucket
- src: git+https://bitbucket.org/willthames/git-ansible-galaxy
  version: v1.4

# from Bitbucket, alternative syntax and caveats
- src: https://bitbucket.org/willthames/hg-ansible-galaxy
  scm: hg

# from GitLab or other git-based scm, using git+ssh
- src: [email protected]:mygroup/ansible-core.git
  scm: git
  version: "0.1"  # quoted, so YAML doesn't parse this as a floating-point value
```



### 从同一个 requirements.yml 文件安装角色和集合

您可以从同一个 requirements 文件安装角色和集合

```
---
roles:
  # Install a role from Ansible Galaxy.
  - name: geerlingguy.java
    version: "1.9.6" # note that ranges are not supported for roles

collections:
  # Install a collection from Ansible Galaxy.
  - name: community.general
    version: ">=7.0.0"
    source: https://galaxy.ansible.com
```



### 从多个文件安装多个角色

对于大型项目，requirements.yml 文件中的 include 指令可以将一个大型文件拆分为多个较小的文件。

例如，一个项目可能有一个 requirements.yml 文件和一个 webserver.yml 文件。

以下是 webserver.yml 文件的内容

```
# from github
- src: https://github.com/bennojoy/nginx

# from Bitbucket
- src: git+https://bitbucket.org/willthames/git-ansible-galaxy
  version: v1.4
```



以下显示了 requirements.yml 文件的内容，该文件现在包含了 webserver.yml 文件

```
# from galaxy
- name: yatesr.timezone
- include: <path_to_requirements>/webserver.yml
```



要安装两个文件中的所有角色，请在命令行中传递根文件，在本例中为 requirements.yml，如下所示

```
$ ansible-galaxy role install -r requirements.yml
```





### 依赖关系

角色也可以依赖于其他角色，并且当您安装具有依赖关系的角色时，这些依赖关系将自动安装到 roles_path。

定义角色依赖关系有两种方法

- 使用 meta/requirements.yml
- 使用 meta/main.yml

#### 使用 meta/requirements.yml

2.10 版本新增。

您可以创建文件 meta/requirements.yml，并使用与 从文件安装多个角色 部分中描述的 requirements.yml 相同的格式定义依赖关系。

从那里，您可以在任务中导入或包含指定的角色。

#### 使用 meta/main.yml

或者，您可以通过在 dependencies 部分下提供角色列表，在 meta/main.yml 文件中指定角色依赖关系。如果角色的来源是 Galaxy，您可以简单地使用 namespace.role_name 格式指定角色。您也可以使用 requirements.yml 中更复杂的格式，允许您提供 src、scm、version 和 name。

以这种方式安装的依赖关系，根据下面描述的其他因素，也将在剧本执行期间在此角色执行之前执行。要更好地理解在剧本执行期间如何处理依赖关系，请参阅 角色。

以下显示了一个包含依赖角色的示例 meta/main.yml 文件

```
---
dependencies:
  - geerlingguy.java

galaxy_info:
  author: geerlingguy
  description: Elasticsearch for Linux.
  company: "Midwestern Mac, LLC"
  license: "license (BSD, MIT)"
  min_ansible_version: 2.4
  galaxy_tags:
    - web
    - system
    - monitoring
    - logging
    - lucene
    - elk
    - elasticsearch
```



标签是沿着依赖链 向下 继承的。为了将标签应用于一个角色及其所有依赖项，标签应该应用于该角色，而不是应用于角色中的所有任务。

作为依赖项列出的角色受制于条件和标签过滤，并且可能不会完全执行，具体取决于应用的标签和条件。

如果角色的来源是 Galaxy，请使用 namespace.role_name 格式指定角色

```
dependencies:
  - geerlingguy.apache
  - geerlingguy.ansible
```



或者，您可以使用 requirements.yml 中使用的复杂形式指定角色依赖关系，如下所示

```
dependencies:
  - name: geerlingguy.ansible
  - name: composer
    src: git+https://github.com/geerlingguy/ansible-role-composer.git
    version: 775396299f2da1f519f0d8885022ca2d6ee80ee8
```



### 列出已安装的角色

使用 list 显示安装在 roles_path 中的每个角色的名称和版本。

```
$ ansible-galaxy role list
  - namespace-1.foo, v2.7.2
  - namespace2.bar, v2.6.2
```



### 删除已安装的角色

使用 remove 从 roles_path 中删除角色

```
$ ansible-galaxy role remove namespace.role_name
```
