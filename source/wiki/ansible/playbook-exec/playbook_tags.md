---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.4 标签 
order: 3
---

# 标签

如果你的剧本很大，则可能需要仅运行剧本的特定部分，而不是运行整个剧本。你可以使用 Ansible 标签来实现此目的。使用标签来执行或跳过选定的任务是一个两步过程。

> 1. 将标签添加到你的任务中，可以单独添加，也可以从块、剧本、角色或导入中继承标签。
> 2. 运行剧本时选择或跳过标签。

- 使用 tags 关键字添加标签
  - 向单个任务添加标签
    - 向处理程序添加标签
    - 向块添加标签
    - 向剧本添加标签
    - 向角色添加标签
    - 向 include 添加标签
    - 向导入添加标签
    - include 的标签继承：块和 apply 关键字
- 特殊标签
  - 始终和从不
- 运行剧本时选择或跳过标签
  - 标签优先级
  - 预览使用标签的结果
  - 选择性地运行可重用文件中的已标记任务
  - 标签继承：向多个任务添加标签
  - 全局配置标签

## 使用 tags 关键字添加标签

你可以向单个任务或 include 添加标签。你还可以通过在块、剧本、角色或导入级别定义标签来向多个任务添加标签。关键字 tags 解决了所有这些用例。tags 关键字始终定义标签并将它们添加到任务；它不会选择或跳过要执行的任务。你只能在运行剧本时通过命令行根据标签选择或跳过任务。有关更多详细信息，请参阅 运行剧本时选择或跳过标签。

### 向单个任务添加标签

最简单的级别是，你可以将一个或多个标签应用于单个任务。你可以在剧本、任务文件中或角色内向任务添加标签。以下是一个使用不同标签标记两个任务的示例。

```
tasks:
- name: Install the servers
  ansible.builtin.yum:
    name:
    - httpd
    - memcached
    state: present
  tags:
  - packages
  - webservers

- name: Configure the service
  ansible.builtin.template:
    src: templates/src.j2
    dest: /etc/foo.conf
  tags:
  - configuration
```



你可以将相同的标签应用于多个单个任务。此示例使用相同的标签“ntp”标记多个任务。

```
---
# file: roles/common/tasks/main.yml

- name: Install ntp
  ansible.builtin.yum:
    name: ntp
    state: present
  tags: ntp

- name: Configure ntp
  ansible.builtin.template:
    src: ntp.conf.j2
    dest: /etc/ntp.conf
  notify:
  - restart ntpd
  tags: ntp

- name: Enable and run ntpd
  ansible.builtin.service:
    name: ntpd
    state: started
    enabled: true
  tags: ntp

- name: Install NFS utils
  ansible.builtin.yum:
    name:
    - nfs-utils
    - nfs-util-lib
    state: present
  tags: filesharing
```



如果你使用 --tags ntp 在剧本中运行这四个任务，Ansible 将运行三个标有 ntp 标签的任务，并跳过没有该标签的一个任务。



#### 向处理程序添加标签

处理程序是任务的一种特殊情况，它们仅在收到通知时才执行，因此它们忽略所有标签，既不能被选中也不能被排除。



#### 向块添加标签

如果你想将标签应用于剧本中的许多任务（但不是全部任务），请使用 块并在该级别定义标签。例如，我们可以编辑上面显示的 NTP 示例以使用块。

```
# myrole/tasks/main.yml
- name: ntp tasks
  tags: ntp
  block:
  - name: Install ntp
    ansible.builtin.yum:
      name: ntp
      state: present

  - name: Configure ntp
    ansible.builtin.template:
      src: ntp.conf.j2
      dest: /etc/ntp.conf
    notify:
    - restart ntpd

  - name: Enable and run ntpd
    ansible.builtin.service:
      name: ntpd
      state: started
      enabled: true

- name: Install NFS utils
  ansible.builtin.yum:
    name:
    - nfs-utils
    - nfs-util-lib
    state: present
  tags: filesharing
```



请注意，tag 选择优先于大多数其他逻辑，包括 block 错误处理。如果在 block 中的任务上设置了标签，但在 rescue 或 always 部分中没有设置，则如果你的标签选择不涵盖这些部分中的任务，则将阻止这些部分触发。

```
- block:
 - debug: msg=run with tag, but always fail
   failed_when: true
   tags: example

 rescue:
 - debug: msg=I always run because the block always fails, except if you select to only run 'example' tag

 always:
 - debug: msg=I always run, except if you select to only run 'example' tag
```



此示例在不指定 --tags 的情况下调用时运行所有 3 个任务，但是如果你使用 --tags example 运行，则只运行第一个任务。



#### 向剧本添加标签

如果剧本中的所有任务都应获得相同的标签，则可以在剧本级别添加标签。例如，如果你有一个只包含 NTP 任务的剧本，则可以标记整个剧本。

```
- hosts: all
  tags: ntp
  tasks:
  - name: Install ntp
    ansible.builtin.yum:
      name: ntp
      state: present

  - name: Configure ntp
    ansible.builtin.template:
      src: ntp.conf.j2
      dest: /etc/ntp.conf
    notify:
    - restart ntpd

  - name: Enable and run ntpd
    ansible.builtin.service:
      name: ntpd
      state: started
      enabled: true

- hosts: fileservers
  tags: filesharing
  tasks:
  ...
```





#### 向角色添加标签

有三种方法可以向角色添加标签。

> 1. 通过在 roles 下设置标签，将相同标签添加到角色中的所有任务。请参阅本节中的示例。
> 2. 通过在你的剧本中静态 import_role 上设置标签，将相同标签添加到角色中的所有任务。请参阅 向导入添加标签 中的示例。
> 3. 向角色本身内的单个任务或块添加标签或标签。这是唯一允许你选择或跳过角色内某些任务的方法。要选择或跳过角色内的任务，必须在单个任务或块上设置标签，在你的剧本中使用动态 include_role，并将相同标签添加到 include。当你使用此方法，然后使用 --tags foo 运行你的剧本时，Ansible 将运行 include 本身以及角色中也具有标签 foo 的任何任务。有关详细信息，请参阅 向 include 添加标签。

当你使用 roles 关键字在你的剧本中静态地包含角色时，Ansible 会将你定义的任何标签添加到角色中的所有任务。例如：

```
roles:
  - role: webserver
    vars:
      port: 5000
    tags: [ web, foo ]
```



或

```
---
- hosts: webservers
  roles:
    - role: foo
      tags:
        - bar
        - baz
    # using YAML shorthand, this is equivalent to:
    # - { role: foo, tags: ["bar", "baz"] }
```





#### 向 include 添加标签

你可以将标签应用于剧本中的动态 include。与单个任务上的标签一样，include_* 任务上的标签仅适用于 include 本身，而不适用于包含文件或角色中的任何任务。如果你向动态 include 添加 mytag，然后使用 --tags mytag 运行该剧本，Ansible 将运行 include 本身，运行包含文件或角色中标有 mytag 的任何任务，并跳过包含文件或角色中没有该标签的任何任务。有关更多详细信息，请参阅 选择性地运行可重用文件中的已标记任务。

你可以像向任何其他任务添加标签一样向 include 添加标签。

```
---
# file: roles/common/tasks/main.yml

- name: Dynamic reuse of database tasks
  include_tasks: db.yml
  tags: db
```



你只能向角色的动态 include 添加标签。在此示例中，foo 标签将不会应用于 bar 角色内的任务。

```
---
- hosts: webservers
  tasks:
    - name: Include the bar role
      include_role:
        name: bar
      tags:
        - foo
```





#### 向导入添加标签

你还可以将标签应用于静态 import_role 和 import_tasks 语句导入的所有任务。

```
---
- hosts: webservers
  tasks:
    - name: Import the foo role
      import_role:
        name: foo
      tags:
        - bar
        - baz

    - name: Import tasks from foo.yml
      import_tasks: foo.yml
      tags: [ web, foo ]
```





#### include 的标签继承：块和 apply 关键字

默认情况下，Ansible 不会将标签继承应用于使用include_role和include_tasks的动态重用。如果您向include添加标签，它们只应用于include本身，而不应用于包含文件或角色中的任何任务。这允许您在运行剧本时执行角色或任务文件中的选定任务 - 请参阅选择性地运行可重用文件中的标记任务。

如果您需要标签继承，您可能需要使用导入。但是，在一个剧本中同时使用include和import可能会导致难以诊断的错误。因此，如果您的剧本使用include_*来重用角色或任务，并且您在一个include上需要标签继承，Ansible 提供了两种解决方法。您可以使用apply关键字

```
- name: Apply the db tag to the include and to all tasks in db.yml
  include_tasks:
    file: db.yml
    # adds 'db' tag to tasks within db.yml
    apply:
      tags: db
  # adds 'db' tag to this 'include_tasks' itself
  tags: db
```



或者您可以使用一个块

```
- block:
   - name: Include tasks from db.yml
     include_tasks: db.yml
  tags: db
```





## 特殊标签

Ansible 为特殊行为保留了一些标签名称：always、never、tagged、untagged和all。always和never主要用于标记任务本身，其他三个用于选择要运行或跳过的标签。

### Always 和 Never

Ansible 保留了一些标签名称用于特殊行为，其中两个是always和never。如果您将always标签分配给任务或剧本，除非您明确跳过它（--skip-tags always）或该任务上定义的另一个标签，否则 Ansible 将始终运行该任务或剧本。

例如

```
tasks:
- name: Print a message
  ansible.builtin.debug:
    msg: "Always runs"
  tags:
  - always

- name: Print a message
  ansible.builtin.debug:
    msg: "runs when you use specify tag1, all(default) or tagged"
  tags:
  - tag1

- name: Print a message
  ansible.builtin.debug:
    msg: "always runs unless you explicitly skip, like if you use ``--skip-tags tag2``"
  tags:
     - always
     - tag2
```



2.5 版本中的新功能。

如果您将never标签分配给任务或剧本，除非您明确请求它（--tags never）或为该任务定义的另一个标签，否则 Ansible 将跳过该任务或剧本。

例如

```
tasks:
  - name: Run the rarely-used debug task, either with ``--tags debug`` or ``--tags never``
    ansible.builtin.debug:
     msg: '{{ showmevar }}'
    tags: [ never, debug ]
```



上例中很少使用的调试任务仅在您明确请求debug或never标签时才运行。



## 运行剧本时选择或跳过标签

将标签添加到任务、include、块、剧本、角色和导入后，您可以在运行ansible-playbook时根据其标签选择性地执行或跳过任务。Ansible 运行或跳过所有其标签与您在命令行中传递的标签匹配的任务。如果您在块或剧本级别、使用roles或使用导入添加了标签，则该标签将应用于块、剧本、角色或导入的角色或文件中的每个任务。如果您有一个带有多个标签的角色，并且您希望在不同时间调用角色的子集，请使用动态 include，或将角色拆分为多个角色。

ansible-playbook 提供五个与标签相关的命令行选项

- --tags all - 运行所有任务，已标记和未标记的任务，除非never（默认行为）。
- --tags tag1,tag2 - 只运行带有tag1或tag2标签的任务（也包括带有always标签的任务）。
- --skip-tags tag3,tag4 - 运行所有任务，除了带有tag3或tag4或never标签的任务。
- --tags tagged - 只运行至少有一个标签的任务（never优先）。
- --tags untagged - 只运行没有标签的任务（always优先）。

例如，要只运行在一个非常长的剧本中标记为configuration或packages的任务和块

```
ansible-playbook example.yml --tags "configuration,packages"
```



要运行所有任务，除了那些标记为packages的任务

```
ansible-playbook example.yml --skip-tags "packages"
```



要运行所有任务，即使那些被排除因为标记为never的任务

```
ansible-playbook example.yml --tags "all,never"
```



运行带有 tag1 或 tag3 的任务，但跳过也带有 tag4 的任务

```
ansible-playbook example.yml --tags "tag1,tag3" --skip-tags "tag4"
```





### 标签优先级

跳过always优先于显式标签，例如，如果您同时指定--tags和--skip-tags，后者优先。例如--tags tag1,tag3,tag4 --skip-tags tag3将只运行标记为tag1或tag4的任务，而不是tag3，即使任务具有其他标签。

### 预览使用标签的结果

运行角色或剧本时，您可能不知道或不记得哪些任务具有哪些标签，或者根本不存在哪些标签。Ansible 为ansible-playbook 提供了两个命令行标志，可帮助您管理标记的剧本

- --list-tags - 生成可用标签列表
- --list-tasks - 与--tags tagname或--skip-tags tagname一起使用时，生成标记任务的预览

例如，如果您不知道配置任务的标签是config还是conf，您可以在不运行任何任务的情况下显示所有可用标签

```
ansible-playbook example.yml --list-tags
```



如果您不知道哪些任务具有configuration和packages标签，您可以传递这些标签并添加--list-tasks。Ansible 将列出任务，但不执行任何任务。

```
ansible-playbook example.yml --tags "configuration,packages" --list-tasks
```



这些命令行标志有一个限制：它们不能显示动态包含的文件或角色中的标签或任务。有关静态导入和动态 include 之间差异的更多信息，请参阅比较 include 和 import：动态和静态重用。



### 选择性地运行可重用文件中的标记任务

如果您有一个在任务或块级别定义了标签的角色或任务文件，如果您使用动态 include 而不是静态导入，则可以在剧本中选择性地运行或跳过这些标记的任务。您必须在包含的任务和 include 语句本身使用相同的标签。例如，您可以创建一个包含一些标记的任务和一些未标记的任务的文件

```
# mixed.yml
tasks:
- name: Run the task with no tags
  ansible.builtin.debug:
    msg: this task has no tags

- name: Run the tagged task
  ansible.builtin.debug:
    msg: this task is tagged with mytag
  tags: mytag

- block:
  - name: Run the first block task with mytag
    ...
  - name: Run the second block task with mytag
    ...
  tags:
  - mytag
```



您可以在剧本中包含上面的任务文件

```
# myplaybook.yml
- hosts: all
  tasks:
  - name: Run tasks from mixed.yml
    include_tasks:
      name: mixed.yml
    tags: mytag
```



当您使用ansible-playbook -i hosts myplaybook.yml --tags "mytag"运行剧本时，Ansible 将跳过没有标签的任务，运行标记的单个任务，并运行块中的两个任务。它还可以运行事实收集（隐式任务），因为它被标记为always。



### 标签继承：向多个任务添加标签

如果您想将相同的标签应用于多个任务，而无需为每个任务都添加tags行，则可以在 playbook 或 block 的级别定义标签，或者在添加角色或导入文件时定义标签。Ansible 会将标签沿依赖链向下应用于所有子任务。对于角色和导入，Ansible 会将roles部分或导入设置的标签附加到角色或导入文件中各个任务或块上设置的任何标签。这称为标签继承。标签继承很方便，因为您不必为每个任务都添加标签。但是，这些标签仍然单独应用于各个任务。

对于 playbook、block、role关键字和静态导入，Ansible 应用标签继承，将您定义的标签添加到 playbook、block、角色或导入文件中每个任务中。但是，标签继承不应用于使用include_role和include_tasks的动态重用。对于动态重用（include），您定义的标签仅应用于 include 本身。如果您需要标签继承，请使用静态导入。如果您由于 playbook 的其余部分使用 include 而无法使用导入，请参阅include 的标签继承：block 和 apply 关键字，了解解决此问题的几种方法。

你可以将标签应用于剧本中的动态 include。与单个任务上的标签一样，include_* 任务上的标签仅适用于 include 本身，而不适用于包含文件或角色中的任何任务。如果你向动态 include 添加 mytag，然后使用 --tags mytag 运行该剧本，Ansible 将运行 include 本身，运行包含文件或角色中标有 mytag 的任何任务，并跳过包含文件或角色中没有该标签的任何任务。有关更多详细信息，请参阅 选择性地运行可重用文件中的已标记任务。



### 全局配置标签

如果您默认运行或跳过某些标签，则可以使用 Ansible 配置中的TAGS_RUN和TAGS_SKIP选项来设置这些默认值。
