---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.9 Ansible 网络自动化示例
order: 8
---

# Ansible 网络自动化示例

本文档介绍了一些使用 Ansible 管理网络基础设施的示例。

- 先决条件
- 清单文件中的组和变量
  - 用于密码加密的 Ansible Vault
  - 通用清单变量
  - 权限提升
  - 跳转主机
- 示例 1：使用 playbook 收集 facts 并创建备份文件
  - 步骤 1：创建清单
  - 步骤 2：创建 playbook
  - 步骤 3：运行 playbook
  - 步骤 4：检查 playbook 结果
- 示例 2：使用与平台无关的模块简化 playbook
  - 使用特定于平台的模块的示例 playbook
  - 使用 cli_command 与平台无关的模块的简化 playbook
  - 使用 ansible.netcommon.cli_command 的多个提示
- 实施说明
  - 演示变量
  - 获取运行配置
- 故障排除

## 先决条件

此示例需要以下内容

- 已安装 Ansible 2.10（或更高版本）。有关详细信息，请参阅 安装 Ansible。
- 一个或多个与 Ansible 兼容的网络设备。
- 对 YAML 的基本了解 YAML 语法。
- 对 Jinja2 模板的基本了解。有关详细信息，请参阅 模板 (Jinja2)。
- 基本 Linux 命令行使用。
- 网络交换机和路由器配置的基本知识。

## 清单文件中的组和变量

inventory 文件是一个 YAML 或类似 INI 的配置文件，用于定义主机到组的映射。

在我们的示例中，清单文件定义了组 eos、ios、vyos 和一个名为 switches 的“组的组”。有关子组和清单文件的更多详细信息，请参阅 Ansible 清单组文档。

由于 Ansible 是一种灵活的工具，因此有多种方法可以指定连接信息和凭据。我们建议在清单文件中使用 [my_group:vars] 功能。

```
[all:vars]
# these defaults can be overridden for any group in the [group:vars] section
ansible_connection=ansible.netcommon.network_cli
ansible_user=ansible

[switches:children]
eos
ios
vyos

[eos]
veos01 ansible_host=veos-01.example.net
veos02 ansible_host=veos-02.example.net
veos03 ansible_host=veos-03.example.net
veos04 ansible_host=veos-04.example.net

[eos:vars]
ansible_become=yes
ansible_become_method=enable
ansible_network_os=arista.eos.eos
ansible_user=my_eos_user
ansible_password=my_eos_password

[ios]
ios01 ansible_host=ios-01.example.net
ios02 ansible_host=ios-02.example.net
ios03 ansible_host=ios-03.example.net

[ios:vars]
ansible_become=yes
ansible_become_method=enable
ansible_network_os=cisco.ios.ios
ansible_user=my_ios_user
ansible_password=my_ios_password

[vyos]
vyos01 ansible_host=vyos-01.example.net
vyos02 ansible_host=vyos-02.example.net
vyos03 ansible_host=vyos-03.example.net

[vyos:vars]
ansible_network_os=vyos.vyos.vyos
ansible_user=my_vyos_user
ansible_password=my_vyos_password
```



如果使用 ssh-agent，则不需要 ansible_password 行。如果使用 ssh 密钥，但不使用 ssh-agent，并且有多个密钥，请使用 ansible_ssh_private_key_file=/path/to/correct/key 在 [group:vars] 部分中指定每个连接要使用的密钥。有关 ansible_ssh_ 选项的更多信息，请参阅 连接到主机：行为清单参数。

### 用于密码加密的 Ansible Vault

Ansible 的“Vault”功能允许您将敏感数据（如密码或密钥）保存在加密文件中，而不是以纯文本形式保存在 playbook 或角色中。然后，可以将这些 Vault 文件分发或放置在源代码控制中。有关更多信息，请参阅 使用加密的变量和文件。

如果您在变量中指定 SSH 密码（使用 Ansible Vault 加密），它将如下所示

```
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: vyos.vyos.vyos
ansible_user: my_vyos_user
ansible_ssh_pass: !vault |
                  $ANSIBLE_VAULT;1.1;AES256
                  39336231636137663964343966653162353431333566633762393034646462353062633264303765
                  6331643066663534383564343537343334633031656538370a333737656236393835383863306466
                  62633364653238323333633337313163616566383836643030336631333431623631396364663533
                  3665626431626532630a353564323566316162613432373738333064366130303637616239396438
                  9853
```



### 通用清单变量

以下变量对于清单中的所有平台都是通用的，但可以为特定的清单组或主机覆盖这些变量。

- ansible_connection:

  Ansible 使用 ansible-connection 设置来确定如何连接到远程设备。使用 Ansible Networking 时，请将其设置为适当的网络连接选项，例如 ansible.netcommon.network_cli，以便 Ansible 将远程节点视为具有有限执行环境的网络设备。如果没有此设置，Ansible 将尝试使用 ssh 连接到远程设备并在网络设备上执行 Python 脚本，由于网络设备上通常没有 Python，因此会失败。

- ansible_network_os:

  通知 Ansible 此主机对应的网络平台。使用 ansible.netcommon.* 连接选项时，这是必需的。

- ansible_user:

  要连接到远程设备（交换机）的用户。如果没有此设置，将使用运行 ansible-playbook 的用户。指定网络设备上连接的用户

- ansible_password:

  用于以 ansible_user 身份登录的相应密码。如果未指定，将使用 SSH 密钥。

- ansible_become:

  是否应使用启用模式（特权模式），请参阅下一节。

- ansible_become_method:

  应使用哪种类型的 become，对于 network_cli，唯一有效的选择是 enable。

### 权限提升

某些网络平台（如 Arista EOS 和 Cisco IOS）具有不同特权模式的概念。某些网络模块（例如修改系统状态（包括用户）的模块）仅在高特权状态下才能工作。使用 connection: ansible.netcommon.network_cli 时，Ansible 支持 become。这允许为需要它们的特定任务提升权限。添加 become: yes 和 become_method: enable 通知 Ansible 在执行任务之前进入特权模式，如下所示

```
[eos:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=arista.eos.eos
ansible_become=yes
ansible_become_method=enable
```



有关更多信息，请参阅 使用 become 与网络模块 指南。

### 跳转主机

如果 Ansible 控制节点没有到远程设备的直接路由，并且需要使用跳转主机，请参阅 Ansible 网络代理命令 指南，了解如何实现此目的的详细信息。

## 示例 1：使用 playbook 收集 facts 并创建备份文件

Ansible facts 模块收集系统信息“facts”，这些信息可用于 playbook 的其余部分。

Ansible 网络功能附带了许多特定于网络的 facts 模块。在此示例中，我们使用 _facts 模块 arista.eos.eos_facts、cisco.ios.ios_facts 和 vyos.vyos.vyos_facts 来连接到远程网络设备。由于凭据没有通过模块参数显式传递，Ansible 会使用清单文件中的用户名和密码。

Ansible 的“网络 Facts 模块”从系统中收集信息，并将结果存储在以 ansible_net_ 为前缀的 facts 中。这些模块收集的数据记录在模块文档的 返回值 部分，在本例中为 arista.eos.eos_facts 和 vyos.vyos.vyos_facts。我们可以稍后在“显示一些 facts”任务中使用 facts，例如 ansible_net_version。

为了确保我们调用正确的模式（*_facts），该任务根据清单文件中定义的组有条件地运行。有关在 Ansible Playbook 中使用条件语句的更多信息，请参阅 使用 when 的基本条件语句。

在本示例中，我们将创建一个包含一些网络交换机的清单文件，然后运行一个 playbook 来连接到网络设备并返回有关它们的一些信息。

### 步骤 1：创建清单

首先，创建一个名为 inventory 的文件，其中包含

```
[switches:children]
eos
ios
vyos

[eos]
eos01.example.net

[ios]
ios01.example.net

[vyos]
vyos01.example.net
```



### 步骤 2：创建 playbook

接下来，创建一个名为 facts-demo.yml 的 playbook 文件，其中包含以下内容

```
- name: "Demonstrate connecting to switches"
  hosts: switches
  gather_facts: no

  tasks:
    ###
    # Collect data
    #
    - name: Gather facts (eos)
      arista.eos.eos_facts:
      when: ansible_network_os == 'arista.eos.eos'

    - name: Gather facts (ios)
      cisco.ios.ios_facts:
      when: ansible_network_os == 'cisco.ios.ios'

    - name: Gather facts (vyos)
      vyos.vyos.vyos_facts:
      when: ansible_network_os == 'vyos.vyos.vyos'

    ###
    # Demonstrate variables
    #
    - name: Display some facts
      debug:
        msg: "The hostname is {{ ansible_net_hostname }} and the OS is {{ ansible_net_version }}"

    - name: Facts from a specific host
      debug:
        var: hostvars['vyos01.example.net']

    - name: Write facts to disk using a template
      copy:
        content: |
          #jinja2: lstrip_blocks: True
          EOS device info:
            {% for host in groups['eos'] %}
            Hostname: {{ hostvars[host].ansible_net_hostname }}
            Version: {{ hostvars[host].ansible_net_version }}
            Model: {{ hostvars[host].ansible_net_model }}
            Serial: {{ hostvars[host].ansible_net_serialnum }}
            {% endfor %}

          IOS device info:
            {% for host in groups['ios'] %}
            Hostname: {{ hostvars[host].ansible_net_hostname }}
            Version: {{ hostvars[host].ansible_net_version }}
            Model: {{ hostvars[host].ansible_net_model }}
            Serial: {{ hostvars[host].ansible_net_serialnum }}
            {% endfor %}

          VyOS device info:
            {% for host in groups['vyos'] %}
            Hostname: {{ hostvars[host].ansible_net_hostname }}
            Version: {{ hostvars[host].ansible_net_version }}
            Model: {{ hostvars[host].ansible_net_model }}
            Serial: {{ hostvars[host].ansible_net_serialnum }}
            {% endfor %}
        dest: /tmp/switch-facts
      run_once: yes

    ###
    # Get running configuration
    #

    - name: Backup switch (eos)
      arista.eos.eos_config:
        backup: yes
      register: backup_eos_location
      when: ansible_network_os == 'arista.eos.eos'

    - name: backup switch (vyos)
      vyos.vyos.vyos_config:
        backup: yes
      register: backup_vyos_location
      when: ansible_network_os == 'vyos.vyos.vyos'

    - name: Create backup dir
      file:
        path: "/tmp/backups/{{ inventory_hostname }}"
        state: directory
        recurse: yes

    - name: Copy backup files into /tmp/backups/ (eos)
      copy:
        src: "{{ backup_eos_location.backup_path }}"
        dest: "/tmp/backups/{{ inventory_hostname }}/{{ inventory_hostname }}.bck"
      when: ansible_network_os == 'arista.eos.eos'

    - name: Copy backup files into /tmp/backups/ (vyos)
      copy:
        src: "{{ backup_vyos_location.backup_path }}"
        dest: "/tmp/backups/{{ inventory_hostname }}/{{ inventory_hostname }}.bck"
      when: ansible_network_os == 'vyos.vyos.vyos'
```



### 步骤 3：运行 playbook

要运行 playbook，请从控制台提示符运行以下命令

```
ansible-playbook -i inventory facts-demo.yml
```



这应该返回类似于以下内容的输出

```
PLAY RECAP
eos01.example.net          : ok=7    changed=2    unreachable=0    failed=0
ios01.example.net          : ok=7    changed=2    unreachable=0    failed=0
vyos01.example.net         : ok=6    changed=2    unreachable=0    failed=0
```



### 步骤 4：检查 playbook 结果

接下来，查看我们创建的包含交换机 facts 的文件的内容

```
cat /tmp/switch-facts
```



您还可以查看备份文件

```
find /tmp/backups
```



如果 ansible-playbook 失败，请按照 网络调试和故障排除指南 中的调试步骤进行操作。



## 示例 2：使用平台无关模块简化 playbook

（此示例最初出现在 Sean Cavanaugh 的《深入探讨网络自动化的 cli_command》博客文章中 -@IPvSean）。

如果您的环境中存在两个或多个网络平台，则可以使用平台无关模块来简化 playbook。您可以使用平台无关模块，例如 ansible.netcommon.cli_command 或 ansible.netcommon.cli_config，来代替平台特定的模块，例如 arista.eos.eos_config、cisco.ios.ios_config 和 junipernetworks.junos.junos_config。这减少了 playbook 中所需的任务和条件语句的数量。

### 使用平台特定模块的示例 playbook

此示例假设有三个平台，Arista EOS、Cisco NXOS 和 Juniper JunOS。如果没有平台无关模块，示例 playbook 可能包含以下三个带有平台特定命令的任务

```
---
- name: Run Arista command
  arista.eos.eos_command:
    commands: show ip int br
  when: ansible_network_os == 'arista.eos.eos'

- name: Run Cisco NXOS command
  cisco.nxos.nxos_command:
    commands: show ip int br
  when: ansible_network_os == 'cisco.nxos.nxos'

- name: Run Vyos command
  vyos.vyos.vyos_command:
    commands: show interface
  when: ansible_network_os == 'vyos.vyos.vyos'
```



### 使用 cli_command 平台无关模块简化的 playbook

您可以将这些平台特定模块替换为平台无关的 ansible.netcommon.cli_command 模块，如下所示

```
---
- hosts: network
  gather_facts: false
  connection: ansible.netcommon.network_cli

  tasks:
    - name: Run cli_command on Arista and display results
      block:
      - name: Run cli_command on Arista
        ansible.netcommon.cli_command:
          command: show ip int br
        register: result

      - name: Display result to terminal window
        debug:
          var: result.stdout_lines
      when: ansible_network_os == 'arista.eos.eos'

    - name: Run cli_command on Cisco IOS and display results
      block:
      - name: Run cli_command on Cisco IOS
        ansible.netcommon.cli_command:
          command: show ip int br
        register: result

      - name: Display result to terminal window
        debug:
          var: result.stdout_lines
      when: ansible_network_os == 'cisco.ios.ios'

    - name: Run cli_command on Vyos and display results
      block:
      - name: Run cli_command on Vyos
        ansible.netcommon.cli_command:
          command: show interfaces
        register: result

      - name: Display result to terminal window
        debug:
          var: result.stdout_lines
      when: ansible_network_os == 'vyos.vyos.vyos'
```



如果您按平台类型使用组和 group_vars，则此 playbook 可以进一步简化为

```
---
- name: Run command and print to terminal window
  hosts: routers
  gather_facts: false

  tasks:
    - name: Run show command
      ansible.netcommon.cli_command:
        command: "{{show_interfaces}}"
      register: command_output
```



您可以在 平台无关示例中看到使用 group_vars 以及配置备份示例的完整示例。

### 将多个提示与 ansible.netcommon.cli_command 结合使用

ansible.netcommon.cli_command 还支持多个提示。

```
---
- name: Change password to default
  ansible.netcommon.cli_command:
    command: "{{ item }}"
    prompt:
      - "New password"
      - "Retype new password"
    answer:
      - "mypassword123"
      - "mypassword123"
    check_all: True
  loop:
    - "configure"
    - "rollback"
    - "set system root-authentication plain-text-password"
    - "commit"
```



有关此命令的完整文档，请参阅 ansible.netcommon.cli_command。

## 实现说明

### 演示变量

尽管这些任务不是将数据写入磁盘所必需的，但在本示例中，它们用于演示访问有关给定设备或指定主机的 facts 的一些方法。

Ansible hostvars 允许您访问来自指定主机的变量。如果没有它，我们将返回当前主机的详细信息，而不是指定主机的详细信息。

有关更多信息，请参阅 有关 Ansible：魔术变量的信息。

### 获取正在运行的配置

arista.eos.eos_config 和 vyos.vyos.vyos_config 模块具有 backup: 选项，当设置此选项时，该模块会在进行任何更改之前，从远程设备创建当前 running-config 的完整备份。备份文件将写入 playbook 根目录中的 backup 文件夹中。如果该目录不存在，则会创建它。

为了演示如何将备份文件移动到不同的位置，我们注册结果并将文件移动到存储在 backup_path 中的路径。

请注意，当以这种方式使用任务中的变量时，我们使用双引号（"）和双花括号（{{...}}），以告诉 Ansible 这表示一个变量。

## 故障排除

如果您收到连接错误，请仔细检查清单和 playbook 中是否存在拼写错误或遗漏的行。如果问题仍然发生，请按照 网络调试和故障排除指南 中的调试步骤进行操作。
