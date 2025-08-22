---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.8 Ansible 技巧与窍门
order: 7
---

# Ansible技巧与窍门

欢迎来到Ansible技巧与窍门指南。这些技巧和窍门帮助我们优化了Ansible的使用，我们在这里将其作为建议提供。我们希望它们能帮助您组织内容、编写playbook、维护清单和执行Ansible。但是最终，您应该以最适合您的组织和目标的方式使用Ansible。

- 一般技巧
  - 保持简单
  - 使用版本控制
  - 自定义CLI输出
  - 避免依赖配置的内容
- Playbook技巧
  - 使用空格
  - 始终命名play、任务和块
  - 始终提及状态
  - 使用注释
  - 使用完全限定的集合名称
- 清单技巧
  - 对云使用动态清单
  - 按功能分组清单
  - 将生产和预发布清单分开
  - 安全地查看保管的变量
- 执行技巧
  - 使用执行环境
  - 先在预发布环境中尝试
  - 分批更新
  - 处理操作系统和发行版差异
- Ansible示例配置
  - 示例目录布局
  - 替代目录布局
  - 示例组和主机变量
  - 按功能组织的示例playbook
  - 基于功能的角色中的示例任务和处理程序文件
  - 示例配置启用什么
  - 组织部署或配置
  - 使用本地Ansible模块



## 通用技巧

这些概念适用于所有 Ansible 活动和工件。

### 保持简单

只要可以，就简单地做事。

仅在必要时使用高级功能，并选择最符合您用例的功能。 例如，您可能不需要同时使用 vars，vars_files，vars_prompt 和 --extra-vars，同时还使用外部清单文件。

如果某件事感觉很复杂，那它可能就是复杂的。花时间寻找更简单的解决方案。

### 使用版本控制

将您的剧本、角色、清单和变量文件保存在 git 或其他版本控制系统中，并在您进行更改时向存储库提交有意义的注释。 版本控制为您提供了一个审计跟踪，描述了您何时以及为何更改了自动化基础设施的规则。

### 自定义 CLI 输出

您可以使用 回调插件 更改 Ansible CLI 命令的输出。

### 避免依赖于配置的内容

为确保您的自动化项目易于理解、修改和与他人共享，您应该避免依赖于配置的内容。 例如，与其将 ansible.cfg 作为项目的根目录引用，不如使用 playbook_dir 或 role_name 等魔术变量来确定项目目录中已知位置的相对路径。 这有助于保持自动化内容的灵活性、可重用性和易于维护。 有关更多信息，请参阅 特殊变量。



## 剧本技巧

这些技巧有助于使剧本和角色更易于阅读、维护和调试。

### 使用空格

大量使用空格，例如，在每个块或任务之前添加一个空行，使剧本易于扫描。

### 始终命名剧本、任务和区块

剧本、任务和区块的 - name: 是可选的，但非常有用。 在其输出中，Ansible 会显示其运行的每个命名实体的名称。 选择描述每个剧本、任务和区块的作用及其原因的名称。

### 始终提及状态

对于许多模块，state 参数是可选的。

不同的模块对 state 有不同的默认设置，并且一些模块支持多个 state 设置。 显式设置 state: present 或 state: absent 使剧本和角色更清晰。

### 使用注释

即使有任务名称和显式状态，有时剧本或角色的一部分（或清单/变量文件）也需要更多解释。 添加注释（任何以 # 开头的行）有助于其他人（以及将来可能您自己）了解剧本或任务（或变量设置）的作用、执行方式以及原因。

### 使用完全限定的集合名称

使用 完全限定的集合名称 (FQCN) 以避免在搜索每个任务的正确模块或插件时出现歧义。

对于 内置模块和插件，请使用 ansible.builtin 集合名称作为前缀，例如 ansible.builtin.copy。



## 清单技巧

这些技巧有助于保持您的清单井然有序。

### 在云中使用动态清单

对于云提供商和其他维护基础设施规范列表的系统，请使用 动态清单 来检索这些列表，而不是手动更新静态清单文件。 对于云资源，您可以使用标签来区分生产和暂存环境。

### 按功能对清单分组

一个系统可以属于多个组。 请参阅 如何构建您的清单 和 模式：定位主机和组。 如果您创建以组中节点的函数命名的组，例如 webservers 或 dbservers，您的剧本可以根据函数定位机器。 您可以使用组变量系统分配特定于功能的变量，并设计 Ansible 角色来处理特定于功能的用例。 请参阅 角色。

### 分离生产和暂存清单

您可以通过为每个环境使用单独的清单文件或目录，将生产环境与开发、测试和暂存环境分开。 这样，您可以使用 -i 来选择您的目标。 将所有环境保存在一个文件中可能会导致意外！ 例如，使用清单时，清单中使用的所有 vault 密码都必须可用。 如果一个清单同时包含生产和开发环境，则使用该清单的开发人员将能够访问生产机密。



### 安全地显示加密的变量

您应该使用 Ansible Vault 加密敏感或机密变量。 但是，加密变量名称和变量值使得难以找到值的来源。 为规避此问题，您可以使用 ansible-vault encrypt_string 单独加密变量，或者添加以下间接层以保持变量名称可访问（例如通过 grep），而不会暴露任何机密

1. 创建一个以组命名的 group_vars/ 子目录。
2. 在此子目录中，创建两个名为 vars 和 vault 的文件。
3. 在 vars 文件中，定义所有需要的变量，包括任何敏感变量。
4. 将所有敏感变量复制到 vault 文件中，并在这些变量前加上 vault_。
5. 调整 vars 文件中的变量，使用 Jinja2 语法指向匹配的 vault_ 变量：例如 db_password: "{{ vault_db_password }}"。
6. 加密 vault 文件以保护其内容。
7. 在你的 Playbook 中使用 vars 文件中的变量名。

当运行 Playbook 时，Ansible 会在未加密的文件中找到变量，然后从加密的文件中提取敏感变量值。变量和 vault 文件的数量或其名称没有限制。

请注意，在清单中使用此策略仍然需要所有 vault 密码都可用（例如，对于 ansible-playbook 或 AWX/Ansible Tower），当使用该清单运行时。



## 执行技巧

这些技巧适用于使用 Ansible，而不是 Ansible 工件。

### 使用执行环境

使用称为 执行环境 的便携式容器镜像来降低复杂性。

### 首先在暂存环境中尝试

在生产环境中推出更改之前，在暂存环境中测试更改始终是一个好主意。你的环境不需要相同的大小，并且你可以使用组变量来控制环境之间的差异。你还可以使用 --syntax-check 标志在暂存环境中检查任何语法错误，如下例所示

```
ansible-playbook --syntax-check
```



### 批量更新

使用 serial 关键字来控制一次批量更新的机器数量。请参阅 控制任务的运行位置：委托和本地操作。



### 处理操作系统和发行版差异

组变量文件和 group_by 模块协同工作，以帮助 Ansible 在需要不同设置、软件包和工具的各种操作系统和发行版上执行。 group_by 模块创建符合特定条件的动态主机组。此组不需要在清单文件中定义。这种方法允许你在不同的操作系统或发行版上执行不同的任务。

例如，以下 Play 会根据操作系统名称将所有系统分类到动态组中

```
- name: Talk to all hosts just so we can learn about them
  hosts: all
  tasks:

    - name: Classify hosts depending on their OS distribution
      ansible.builtin.group_by:
        key: os_{{ ansible_facts['distribution'] }}
```



后续的 Play 可以将这些组作为 hosts 行上的模式，如下所示

```
- hosts: os_CentOS
  gather_facts: False
  tasks:

    # Tasks for CentOS hosts only go in this play.
    - name: Ping my CentOS hosts
      ansible.builtin.ping:
```



你还可以在组变量文件中添加组特定的设置。在以下示例中，CentOS 机器的 asdf 值为 '42'，而其他机器的为 '10'。你还可以使用组变量文件将角色应用于系统以及设置变量。

```
---
# file: group_vars/all
asdf: 10

---
# file: group_vars/os_CentOS.yml
asdf: 42
```



当只需要特定于操作系统的变量而不是任务时，你可以使用相同的设置和 include_vars

```
- name: Use include_vars to include OS-specific variables and print them
  hosts: all
  tasks:

    - name: Set OS distribution dependent variables
      ansible.builtin.include_vars: "os_{{ ansible_facts['distribution'] }}.yml"

    - name: Print the variable
      ansible.builtin.debug:
        var: asdf
```



这将从 group_vars/os_CentOS.yml 文件中提取变量。



