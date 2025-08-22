---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.6 使用 Ansible 网络角色
order: 5
---

# 使用 Ansible 网络角色

角色是一组 Ansible 默认值、文件、任务、模板、变量和其他协同工作的 Ansible 组件。正如你在运行你的第一个命令和剧本中看到的那样，从命令转向剧本使得运行多个任务并按相同顺序重复相同任务变得容易。从剧本转向角色使得重用和共享你的有序任务更加容易。你可以查看Ansible Galaxy，它允许你共享你的角色并使用其他人的角色，无论是直接使用还是作为灵感。

- 理解角色
  - 一个 DNS 剧本示例
  - 将剧本转换为角色
  - 变量优先级
    - 最低优先级
    - 最高优先级
  - 更新已安装的角色

## 理解角色

那么，角色到底是什么，为什么你应该关心它呢？Ansible 角色基本上是分解成已知文件结构的剧本。从剧本转向角色使得共享、阅读和更新你的 Ansible 工作流程更加容易。用户可以编写自己的角色。例如，你不需要编写自己的 DNS 剧本。相反，你只需要指定一个 DNS 服务器和一个角色来为你配置它。

为了进一步简化你的工作流程，Ansible 网络团队编写了一系列针对常见网络用例的角色。使用这些角色意味着你无需重新发明轮子。无需编写和维护你自己的create_vlan 剧本或角色，你可以专注于设计、编写和维护描述你的网络拓扑和清单的解析器模板，并让 Ansible 的网络角色完成工作。请参阅 Ansible Galaxy 上的与网络相关的角色。

### 一个 DNS 剧本示例

为了演示角色的概念，下面的示例playbook.yml是一个包含两个任务剧本的单个 YAML 文件。这个 Ansible 剧本配置 Cisco IOS XE 设备上的主机名，然后配置 DNS（域名系统）服务器。

```
---
- name: configure cisco routers
  hosts: routers
  connection: ansible.netcommon.network_cli
  gather_facts: no
  vars:
    dns: "8.8.8.8 8.8.4.4"

  tasks:
   - name: configure hostname
     cisco.ios.ios_config:
       lines: hostname {{ inventory_hostname }}

   - name: configure DNS
     cisco.ios.ios_config:
       lines: ip name-server {{dns}}
```



如果你使用ansible-playbook命令运行此剧本，你将看到下面的输出。此示例使用-l选项将剧本限制为仅在rtr1节点上执行。

```
[user@ansible ~]$ ansible-playbook playbook.yml -l rtr1

PLAY [configure cisco routers] *************************************************

TASK [configure hostname] ******************************************************
changed: [rtr1]

TASK [configure DNS] ***********************************************************
changed: [rtr1]

PLAY RECAP *********************************************************************
rtr1                       : ok=2    changed=2    unreachable=0    failed=0
```



此剧本配置了主机名和 DNS 服务器。你可以在 Cisco IOS XE rtr1 路由器上验证该配置。

```
rtr1#sh run | i name
hostname rtr1
ip name-server 8.8.8.8 8.8.4.4
```



### 将剧本转换为角色

下一步是将此剧本转换为可重用的角色。你可以手动创建目录结构，也可以使用ansible-galaxy init创建角色的标准框架。

```
[user@ansible ~]$ ansible-galaxy init system_demo
[user@ansible ~]$ cd system_demo/
[user@ansible system_demo]$ tree
.
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
  └── main.yml
```



此第一个演示仅使用tasks和vars目录。目录结构如下所示：

```
[user@ansible system_demo]$ tree
.
├── tasks
│   └── main.yml
└── vars
    └── main.yml
```



接下来，将原始 Ansible 剧本中的vars和tasks部分的内容移入角色。首先，将这两个任务移入tasks/main.yml文件。

```
[user@ansible system_demo]$ cat tasks/main.yml
---
- name: configure hostname
  cisco.ios.ios_config:
    lines: hostname {{ inventory_hostname }}

- name: configure DNS
  cisco.ios.ios_config:
    lines: ip name-server {{dns}}
```



接下来，将变量移入vars/main.yml文件。

```
[user@ansible system_demo]$ cat vars/main.yml
---
dns: "8.8.8.8 8.8.4.4"
```



最后，修改原始 Ansible 剧本以移除tasks和vars部分，并添加关键字roles以及角色的名称，在本例中为system_demo。你将拥有这个剧本：

```
---
- name: configure cisco routers
  hosts: routers
  connection: ansible.netcommon.network_cli
  gather_facts: false

  roles:
    - system_demo
```



总而言之，此演示现在共有三个目录和三个 YAML 文件。有一个system_demo文件夹，它代表该角色。system_demo包含两个文件夹，tasks和vars。每个各自的文件夹中都有一个main.yml。vars/main.yml包含来自playbook.yml的变量。tasks/main.yml包含来自playbook.yml的任务。playbook.yml文件已被修改为调用角色而不是直接指定 vars 和 tasks。这是当前工作目录的树结构：

```
[user@ansible ~]$ tree
.
├── playbook.yml
└── system_demo
    ├── tasks
    │   └── main.yml
    └── vars
        └── main.yml
```



运行剧本会导致行为相同，但输出略有不同。

```
[user@ansible ~]$ ansible-playbook playbook.yml -l rtr1

PLAY [configure cisco routers] *************************************************

TASK [system_demo : configure hostname] ****************************************
ok: [rtr1]

TASK [system_demo : configure DNS] *********************************************
ok: [rtr1]

PLAY RECAP *********************************************************************
rtr1             : ok=2    changed=0    unreachable=0    failed=0
```



如上所示，每个任务现在都以角色名称作为前缀，在本例中为system_demo。当运行包含多个角色的剧本时，这将有助于查明任务的调用位置。此剧本返回ok而不是changed，因为它与我们开始使用的单个文件剧本的行为相同。

与之前一样，剧本将在 Cisco IOS-XE 路由器上生成以下配置：

```
rtr1#sh run | i name
hostname rtr1
ip name-server 8.8.8.8 8.8.4.4
```



这就是为什么 Ansible 角色可以简单地认为是解构后的剧本。它们简单、有效且可重用。现在，其他用户可以简单地包含system_demo角色，而不必创建自定义的“硬编码”剧本。

### 变量优先级

如果你想更改 DNS 服务器怎么办？你不应该更改角色结构内的vars/main.yml。Ansible 有许多地方可以为给定的剧本指定变量。有关变量和优先级的详细信息，请参阅使用变量。实际上有 21 个地方可以放置变量。虽然乍一看这个列表似乎令人难以招架，但绝大多数用例只需要知道最低优先级变量的位置以及如何传递最高优先级变量。有关应在何处放置变量的更多指导，请参阅变量优先级：我应该将变量放在哪里？。

#### 最低优先级

最低优先级是角色内的defaults目录。这意味着你可以潜在地指定变量的其他 20 个位置都将具有比defaults更高的优先级，无论如何。要立即使来自system_demo角色的 vars 具有最低优先级，请将vars目录重命名为defaults。

```
[user@ansible system_demo]$ mv vars defaults
[user@ansible system_demo]$ tree
.
├── defaults
│   └── main.yml
├── tasks
│   └── main.yml
```



向剧本添加一个新的vars部分以覆盖默认行为（其中变量dns设置为 8.8.8.8 和 8.8.4.4）。在此演示中，将dns设置为 1.1.1.1，因此playbook.yml变为：

```
---
- name: configure cisco routers
  hosts: routers
  connection: ansible.netcommon.network_cli
  gather_facts: no
  vars:
    dns: 1.1.1.1
  roles:
    - system_demo
```



在rtr2上运行此更新后的剧本。

```
[user@ansible ~]$ ansible-playbook playbook.yml -l rtr2
```



rtr2 Cisco 路由器上的配置如下所示：

```
rtr2#sh run | i name-server
ip name-server 1.1.1.1
```



剧本中配置的变量现在优先于defaults目录。事实上，你在任何其他地方配置的变量都将优先于defaults目录中的值。

#### 最高优先级

在角色的defaults目录中指定变量将始终具有最低优先级，而使用-e或--extra-vars=选项指定vars作为额外变量将始终具有最高优先级，无论其他设置如何。使用-e选项重新运行Playbook会覆盖defaults目录（8.8.4.4和8.8.8.8）以及Playbook中新创建的包含1.1.1.1 DNS服务器的vars。

```
[user@ansible ~]$ ansible-playbook playbook.yml -e "dns=192.168.1.1" -l rtr3
```



Cisco IOS XE路由器的结果将只包含最高优先级的设置192.168.1.1。

```
rtr3#sh run | i name-server
ip name-server 192.168.1.1
```



这有什么用？你为什么应该关心？额外变量通常由网络运营商用来覆盖默认值。一个强大的例子是AWX上的作业模板调查功能或Red Hat Ansible自动化平台。通过Web UI，可以提示网络运营商使用Web表单填写参数。对于非技术Playbook编写者来说，这可以非常简单地使用他们的Web浏览器执行Playbook。

### 更新已安装的角色

Ansible Galaxy页面列出了角色的所有可用版本。要将本地安装的角色更新到新的或不同的版本，请使用带有版本号和--force选项的ansible-galaxy install命令。您可能还需要手动更新任何依赖的角色以支持此版本。请参阅Galaxy中角色的自述文件选项卡以了解依赖角色的最低版本要求。

```
[user@ansible]$ ansible-galaxy install mynamespace.my_role,v2.7.1 --force
```
