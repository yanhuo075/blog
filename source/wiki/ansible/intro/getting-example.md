---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 1.4 Ansible 使用示例
order: 3
---

# 快速开始

## 1.构建清单

清单将受管节点组织在集中式文件中，这些文件为 Ansible 提供系统信息和网络位置。使用清单文件，Ansible 可以使用单个命令管理大量主机。

要完成以下步骤，您需要至少一个主机系统的 IP 地址或完全限定域名 (FQDN)。出于演示目的，主机可以在容器或虚拟机中本地运行。您还必须确保您的公共 SSH 密钥已添加到每个主机上的 `authorized_keys` 文件中。

继续 Ansible 入门并按如下步骤构建清单

1. 在您在 上一步 中创建的 ansible_quickstart 目录中创建一个名为 inventory.ini 的文件。

2. 向 `inventory.ini` 文件添加一个新的 `[myhosts]` 组，并指定每个主机系统的 IP 地址或完全限定域名 (FQDN)。

   ```
   [myhosts]
   192.0.2.50
   192.0.2.51
   192.0.2.52
   ```

   

3. 验证您的清单。

   ```
   ansible-inventory -i inventory.ini --list
   ```

   

4. Ping 清单中的 `myhosts` 组。

   ```
   ansible myhosts -m ping -i inventory.ini
   ```

   

   ```
   192.0.2.50 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   192.0.2.51 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   192.0.2.52 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   ```

   

恭喜，您已成功构建清单。通过 创建剧本 继续 Ansible 入门。

### INI 或 YAML 格式的清单

您可以使用 `INI` 文件或 `YAML` 创建清单。在大多数情况下，例如上一步中的示例，对于少量受管节点，`INI` 文件简单易读。

随着受管节点数量的增加，使用 `YAML` 格式创建清单变得更合理。例如，以下是 `inventory.ini` 的等效项，它声明受管节点的唯一名称并使用 `ansible_host` 字段

```
myhosts:
  hosts:
    my_host_01:
      ansible_host: 192.0.2.50
    my_host_02:
      ansible_host: 192.0.2.51
    my_host_03:
      ansible_host: 192.0.2.52
```



### 构建清单的技巧

- 确保组名有意义且唯一。组名也区分大小写。

- 避免在组名中使用空格、连字符和前导数字（使用 `floor_19`，而不是 `19th_floor`）。

- 根据主机的**什么**、**哪里**和**何时**逻辑地将主机分组到您的清单中。

  - 什么

    根据拓扑结构对主机分组，例如：db、web、leaf、spine。

  - 哪里

    按地理位置对主机分组，例如：数据中心、区域、楼层、建筑物。

  - 何时

    按阶段对主机分组，例如：开发、测试、过渡、生产。

### 使用元组

使用以下语法创建一个元组来组织清单中的多个组

```
metagroupname:
  children:
```



以下清单说明了数据中心的结构基础。此示例清单包含一个 `network` 元组，其中包括所有网络设备，以及一个 `datacenter` 元组，其中包括 `network` 组和所有 Web 服务器。

```
leafs:
  hosts:
    leaf01:
      ansible_host: 192.0.2.100
    leaf02:
      ansible_host: 192.0.2.110

spines:
  hosts:
    spine01:
      ansible_host: 192.0.2.120
    spine02:
      ansible_host: 192.0.2.130

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
    webserver02:
      ansible_host: 192.0.2.150

datacenter:
  children:
    network:
    webservers:
```



### 创建变量

变量设置受管节点的值，例如 IP 地址、FQDN、操作系统和 SSH 用户，因此您无需在运行 Ansible 命令时传递它们。

变量可以应用于特定主机。

```
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
```



变量也可以应用于组中的所有主机。

```
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
  vars:
    ansible_user: my_server_user
```



## 2.创建剧本

剧本是 Ansible 用于部署和配置受管节点的自动化蓝图，采用 `YAML` 格式。

- 剧本

  一系列定义 Ansible 执行操作顺序的剧目，自上而下，以实现总体目标。

- 剧目

  一个有序的任务列表，映射到清单中的受管节点。

- 任务

  对单个模块的引用，定义 Ansible 执行的操作。

- 模块

  Ansible 在受管节点上运行的代码或二进制单元。Ansible 模块按集合分组，每个模块都有一个完全限定的集合名称 (FQCN)。

完成以下步骤以创建用于 ping 主机并打印“Hello world”消息的剧本

1. 在您之前创建的 `ansible_quickstart` 目录中创建一个名为 `playbook.yaml` 的文件，内容如下：

   ```
   - name: My first play
     hosts: myhosts
     tasks:
      - name: Ping my hosts
        ansible.builtin.ping:
   
      - name: Print message
        ansible.builtin.debug:
          msg: Hello world
   ```

   

2. 运行您的剧本。

   ```
   ansible-playbook -i inventory.ini playbook.yaml
   ```

   

Ansible 返回以下输出

```
PLAY [My first play] ****************************************************************************

TASK [Gathering Facts] **************************************************************************
ok: [192.0.2.50]
ok: [192.0.2.51]
ok: [192.0.2.52]

TASK [Ping my hosts] ****************************************************************************
ok: [192.0.2.50]
ok: [192.0.2.51]
ok: [192.0.2.52]

TASK [Print message] ****************************************************************************
ok: [192.0.2.50] => {
    "msg": "Hello world"
}
ok: [192.0.2.51] => {
    "msg": "Hello world"
}
ok: [192.0.2.52] => {
    "msg": "Hello world"
}

PLAY RECAP **************************************************************************************
192.0.2.50: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.0.2.51: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.0.2.52: ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```



在此输出中，您可以看到

- 您为剧目和每个任务指定的名称。您应始终使用易于验证和排查剧本问题的描述性名称。
- “收集事实”任务隐式运行。默认情况下，Ansible 会收集有关您的清单的信息，以便在剧本中使用。
- 每个任务的状态。每个任务的状态为 `ok`，表示它已成功运行。
- 剧目摘要，总结了每个主机上所有任务的结果。在此示例中，共有三个任务，因此 `ok=3` 表示每个任务都已成功运行。

恭喜，您已开始使用 Ansible！
