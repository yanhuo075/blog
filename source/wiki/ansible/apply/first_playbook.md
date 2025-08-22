---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.4 运行第一个Playbook
order: 3
---

# 运行您的第一个命令和 Playbook

利用本快速教程学习并应用您所学的概念。安装 Ansible，手动执行网络配置命令，使用 Ansible 执行相同的命令，然后创建一个 Playbook，以便您可以随时在多个网络设备上执行该命令。

- 先决条件
- 安装 Ansible
- 建立与受管节点的手动连接
- 运行您的第一个网络 Ansible 命令
- 创建和运行您的第一个网络 Ansible Playbook
- 从网络设备收集事实

## 先决条件

在学习本教程之前，您需要：

- 安装 Ansible 2.10（或更高版本）
- 一个或多个与 Ansible 兼容的网络设备
- 基本的 Linux 命令行知识
- 网络交换机和路由器配置的基本知识

## 安装 Ansible

使用您首选的方法安装 Ansible。请参阅 安装 Ansible。然后返回本教程。

确认 Ansible 版本（必须 >= 2.10）

```
ansible --version
```



## 建立与受管节点的手动连接

要确认您的凭据，请手动连接到网络设备并检索其配置。将示例用户名和设备名称替换为您真实的凭据。例如，对于 VyOS 路由器：

```
ssh [email protected]
show config
exit
```



此手动连接还会建立网络设备的身份验证，并将它的 RSA 密钥指纹添加到您的已知主机列表中。（如果您之前已连接到该设备，则您已建立其身份验证。）

## 运行您的第一个网络 Ansible 命令

无需手动连接并在网络设备上运行命令，您可以使用单个简化的 Ansible 命令检索其配置：

```
ansible all -i vyos.example.net, -c ansible.netcommon.network_cli -u my_vyos_user -k -m vyos.vyos.vyos_facts -e ansible_network_os=vyos.vyos.vyos
```



- 此命令中的标志设置七个值：

  应应用命令的主机组（在本例中为所有）清单（-i，目标设备或设备 - 不带尾随逗号 -i 指向清单文件）连接方法（-c，连接和执行 Ansible 的方法）用户（-u，SSH 连接的用户名）SSH 连接方法（-k，提示输入密码）模块（-m，要运行的 Ansible 模块，使用完全限定的集合名称 (FQCN)）额外变量（-e，在本例中，设置网络操作系统值）

注意：如果您使用 ssh-agent 和 ssh 密钥，Ansible 会自动加载它们。您可以省略 -k 标志。

## 创建和运行您的第一个网络 Ansible Playbook

如果您想每天运行此命令，您可以将其保存在 Playbook 中，并使用 ansible-playbook 而不是 ansible 来运行它。Playbook 可以存储您在命令行中使用标志提供的许多参数，从而减少在命令行中输入的内容。为此，您需要两个文件 - 一个 Playbook 和一个清单文件。

1. 下载 first_playbook.yml，其内容如下所示：

```
---

- name: Network Getting Started First Playbook
  connection: ansible.netcommon.network_cli
  gather_facts: false
  hosts: all
  tasks:

    - name: Get config for VyOS devices
      vyos.vyos.vyos_facts:
        gather_subset: all

    - name: Display the config
      debug:
        msg: "The hostname is {{ ansible_net_hostname }} and the OS is {{ ansible_net_version }}"
```



Playbook 设置了上述命令行中的七个值中的三个：组 (hosts: all)、连接方法 (connection: ansible.netcommon.network_cli) 和模块（在每个任务中）。通过在 Playbook 中设置这些值，您可以在命令行中省略它们。Playbook 还添加了第二个任务来显示配置输出。当模块在 Playbook 中运行时，输出将保存在内存中以供将来的任务使用，而不是写入控制台。此处的调试任务允许您在 shell 中查看结果。

1. 使用以下命令运行 Playbook：

```
ansible-playbook -i vyos.example.net, -u ansible -k -e ansible_network_os=vyos.vyos.vyos first_playbook.yml
```



Playbook 包含一个包含两个任务的剧集，并应生成如下输出：

```
$ ansible-playbook -i vyos.example.net, -u ansible -k -e ansible_network_os=vyos.vyos.vyos first_playbook.yml

PLAY [Network Getting Started First Playbook]
***************************************************************************************************************************

TASK [Get config for VyOS devices]
***************************************************************************************************************************
ok: [vyos.example.net]

TASK [Display the config]
***************************************************************************************************************************
ok: [vyos.example.net] => {
    "msg": "The hostname is vyos and the OS is VyOS 1.1.8"
}
```



1. 现在您可以检索设备配置了，尝试使用 Ansible 更新它。下载 first_playbook_ext.yml，它是第一个 Playbook 的扩展版本。

```
---

- name: Network Getting Started First Playbook Extended
  connection: ansible.netcommon.network_cli
  gather_facts: false
  hosts: all
  tasks:

    - name: Get config for VyOS devices
      vyos.vyos.vyos_facts:
        gather_subset: all

    - name: Display the config
      debug:
        msg: "The hostname is {{ ansible_net_hostname }} and the OS is {{ ansible_net_version }}"

    - name: Update the hostname
      vyos.vyos.vyos_config:
        backup: yes
        lines:
          - set system host-name vyos-changed

    - name: Get changed config for VyOS devices
      vyos.vyos.vyos_facts:
        gather_subset: all

    - name: Display the changed config
      debug:
        msg: "The new hostname is {{ ansible_net_hostname }} and the OS is {{ ansible_net_version }}"
```



扩展的第一个 Playbook 在一个剧集中有五个任务。使用上面使用的相同命令运行它。输出显示 Ansible 对配置所做的更改。

```
$ ansible-playbook -i vyos.example.net, -u ansible -k -e ansible_network_os=vyos.vyos.vyos first_playbook_ext.yml

PLAY [Network Getting Started First Playbook Extended]
************************************************************************************************************************************

TASK [Get config for VyOS devices]
**********************************************************************************************************************************
ok: [vyos.example.net]

TASK [Display the config]
*************************************************************************************************************************************
ok: [vyos.example.net] => {
    "msg": "The hostname is vyos and the OS is VyOS 1.1.8"
}

TASK [Update the hostname]
*************************************************************************************************************************************
changed: [vyos.example.net]

TASK [Get changed config for VyOS devices]
*************************************************************************************************************************************
ok: [vyos.example.net]

TASK [Display the changed config]
*************************************************************************************************************************************
ok: [vyos.example.net] => {
    "msg": "The new hostname is vyos-changed and the OS is VyOS 1.1.8"
}

PLAY RECAP
************************************************************************************************************************************
vyos.example.net           : ok=5    changed=1    unreachable=0    failed=0
```





## 从网络设备收集事实

gather_facts 关键字现在支持以标准化的键/值对收集网络设备事实。您可以将这些网络事实提供给后续任务以管理网络设备。

您还可以将新的 gather_network_resources 参数与网络 *_facts 模块（例如 arista.eos.eos_facts）一起使用，以仅返回设备配置的一个子集，如下所示。

```
- hosts: arista
  gather_facts: True
  gather_subset: interfaces
  module_defaults:
    arista.eos.eos_facts:
      gather_network_resources: interfaces
```



Playbook 返回以下接口事实：

```
"network_resources": {
      "interfaces": [
          {
              "description": "test-interface",
              "enabled": true,
              "mtu": "512",
              "name": "Ethernet1"
          },
          {
              "enabled": true,
              "mtu": "3000",
              "name": "Ethernet2"
          },
          {
              "enabled": true,
              "name": "Ethernet3"
          },
          {
              "enabled": true,
              "name": "Ethernet4"
          },
          {
              "enabled": true,
              "name": "Ethernet5"
          },
          {
              "enabled": true,
              "name": "Ethernet6"
          },
      ]
  }
```



请注意，gather_network_resources 将所有受支持资源（接口、BGP、OSPF 等）的配置数据呈现为事实，而 gather_subset 主要用于获取操作数据。

您可以存储这些事实并直接在另一个任务中使用它们，例如使用 eos_interfaces 资源模块。
