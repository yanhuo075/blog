---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.5 构建清单
order: 4
---

# 构建您的清单

运行 playbook 而无需清单需要多个命令行标志。此外，针对单个设备运行 playbook 不会比手动进行相同更改带来巨大的效率提升。充分利用 Ansible 功能的下一步是使用清单文件将您的受管节点组织到组中，其中包含诸如 ansible_network_os 和 SSH 用户之类的信息。功能齐全的清单文件可以作为您网络的真相来源。使用清单文件，单个 playbook 可以使用单个命令维护数百个网络设备。此页面将逐步向您展示如何构建清单文件。

- 基本清单
- 向清单添加变量
- 清单中的组变量
- 变量语法
- 按平台分组清单
- 验证清单
- 使用 ansible-vault 保护敏感变量

## 基本清单

首先，以逻辑方式分组您的清单。最佳实践是根据其是什么（应用程序、堆栈或微服务）、在哪里（数据中心或区域）以及何时（开发阶段）对服务器和网络设备进行分组。

- 是什么：db、web、leaf、spine
- 在哪里：east、west、floor_19、building_A
- 何时：dev、test、staging、prod

在组名中避免使用空格、连字符和前导数字（使用 floor_19，而不是 19th_floor）。组名区分大小写。

这个小型示例数据中心说明了基本组结构。您可以使用语法 [metagroupname:children] 并列出作为元组成员的组来分组组。在这里，组 network 包括所有 leaf 和所有 spine；组 datacenter 包括所有网络设备以及所有 web 服务器。

```
---

leafs:
  hosts:
    leaf01:
      ansible_host: 10.16.10.11
    leaf02:
      ansible_host: 10.16.10.12

spines:
  hosts:
    spine01:
      ansible_host: 10.16.10.13
    spine02:
      ansible_host: 10.16.10.14

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 10.16.10.15
    webserver02:
      ansible_host: 10.16.10.16

datacenter:
  children:
    network:
    webservers:
```



您也可以使用 INI 格式创建相同的清单。

```
[leafs]
leaf01
leaf02

[spines]
spine01
spine02

[network:children]
leafs
spines

[webservers]
webserver01
webserver02

[datacenter:children]
network
webservers
```



## 向清单添加变量

接下来，您可以为在第一个 Ansible 命令中需要的许多变量设置值，这样您就可以在 ansible-playbook 命令中跳过它们。在此示例中，清单包含每个网络设备的 IP、操作系统和 SSH 用户。如果您的网络设备只能通过 IP 访问，则必须将 IP 添加到清单文件。如果您使用主机名访问网络设备，则不需要 IP。

```
---

leafs:
  hosts:
    leaf01:
      ansible_host: 10.16.10.11
      ansible_network_os: vyos.vyos.vyos
      ansible_user: my_vyos_user
    leaf02:
      ansible_host: 10.16.10.12
      ansible_network_os: vyos.vyos.vyos
      ansible_user: my_vyos_user

spines:
  hosts:
    spine01:
      ansible_host: 10.16.10.13
      ansible_network_os: vyos.vyos.vyos
      ansible_user: my_vyos_user
    spine02:
      ansible_host: 10.16.10.14
      ansible_network_os: vyos.vyos.vyos
      ansible_user: my_vyos_user

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 10.16.10.15
      ansible_user: my_server_user
    webserver02:
      ansible_host: 10.16.10.16
      ansible_user: my_server_user

datacenter:
  children:
    network:
    webservers:
```



## 清单中的组变量

当组中的设备共享相同的变量值（例如操作系统或 SSH 用户）时，您可以通过将这些值合并到组变量中来减少重复并简化维护。

```
---

leafs:
  hosts:
    leaf01:
      ansible_host: 10.16.10.11
    leaf02:
      ansible_host: 10.16.10.12
  vars:
    ansible_network_os: vyos.vyos.vyos
    ansible_user: my_vyos_user

spines:
  hosts:
    spine01:
      ansible_host: 10.16.10.13
    spine02:
      ansible_host: 10.16.10.14
  vars:
    ansible_network_os: vyos.vyos.vyos
    ansible_user: my_vyos_user

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 10.16.10.15
    webserver02:
      ansible_host: 10.16.10.16
  vars:
    ansible_user: my_server_user

datacenter:
  children:
    network:
    webservers:
```



## 变量语法

清单、playbook 和 group_vars 文件（如下所述）中变量值的语法有所不同。即使 playbook 和 group_vars 文件都是用 YAML 编写的，您在每个文件中使用变量的方式也不同。

- 在 ini 样式的清单文件中，您必须使用语法 key=value 表示变量值：ansible_network_os=vyos.vyos.vyos。
- 在任何扩展名为 .yml 或 .yaml 的文件中（包括 playbook 和 group_vars 文件），您必须使用 YAML 语法：key: value。
- 在 group_vars 文件中，使用完整的 key 名称：ansible_network_os: vyos.vyos.vyos。
- 在 playbook 中，使用简写形式的 key 名称，去掉 ansible 前缀：network_os: vyos.vyos.vyos。

## 按平台分组清单

随着清单的增长，您可能希望按平台对设备进行分组。这使您可以轻松地为该平台上的所有设备指定特定于平台的变量。

```
---

leafs:
  hosts:
    leaf01:
      ansible_host: 10.16.10.11
    leaf02:
      ansible_host: 10.16.10.12

spines:
  hosts:
    spine01:
      ansible_host: 10.16.10.13
    spine02:
      ansible_host: 10.16.10.14

network:
  children:
    leafs:
    spines:
  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: vyos.vyos.vyos
    ansible_user: my_vyos_user

webservers:
  hosts:
    webserver01:
      ansible_host: 10.16.10.15
    webserver02:
      ansible_host: 10.16.10.16
  vars:
    ansible_user: my_server_user

datacenter:
  children:
    network:
    webservers:
```



使用此设置，您只需使用两个标志即可运行 first_playbook.yml。

```
ansible-playbook -i inventory.yml -k first_playbook.yml
```



使用 -k 标志，您可以在提示符处提供 SSH 密码。或者，您可以使用 ansible-vault 将 SSH 和其他机密和密码安全地存储在您的 group_vars 文件中。有关详细信息，请参阅 使用 ansible-vault 保护敏感变量。

## 验证清单

您可以使用 ansible-inventory CLI 命令来显示 Ansible 所看到的清单。

```
ansible-inventory -i test.yml --list
  {
    "_meta": {
        "hostvars": {
            "leaf01": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.16.10.11",
                "ansible_network_os": "vyos.vyos.vyos",
                "ansible_user": "my_vyos_user"
            },
            "leaf02": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.16.10.12",
                "ansible_network_os": "vyos.vyos.vyos",
                "ansible_user": "my_vyos_user"
            },
            "spine01": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.16.10.13",
                "ansible_network_os": "vyos.vyos.vyos",
                "ansible_user": "my_vyos_user"
            },
            "spine02": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.16.10.14",
                "ansible_network_os": "vyos.vyos.vyos",
                "ansible_user": "my_vyos_user"
            },
            "webserver01": {
                "ansible_host": "10.16.10.15",
                "ansible_user": "my_server_user"
            },
            "webserver02": {
                "ansible_host": "10.16.10.16",
                "ansible_user": "my_server_user"
            }
        }
    },
    "all": {
        "children": [
            "datacenter",
            "ungrouped"
        ]
    },
    "datacenter": {
        "children": [
            "network",
            "webservers"
        ]
    },
    "leafs": {
        "hosts": [
            "leaf01",
            "leaf02"
        ]
    },
    "network": {
        "children": [
            "leafs",
            "spines"
        ]
    },
    "spines": {
        "hosts": [
            "spine01",
            "spine02"
        ]
    },
    "webservers": {
        "hosts": [
            "webserver01",
            "webserver02"
        ]
    }
  }
```





## 使用 ansible-vault 保护敏感变量

ansible-vault 命令为文件和/或单个变量（如密码）提供加密。本教程将向您展示如何加密单个 SSH 密码。您可以使用以下命令加密其他敏感信息，例如数据库密码、特权升级密码等等。

首先，您必须为 ansible-vault 本身创建一个密码。它用作加密密钥，您可以使用它在 Ansible 项目中加密数十个不同的密码。运行 playbook 时，您可以使用单个密码（ansible-vault 密码）访问所有这些机密（加密值）。这是一个简单的示例。

1. 创建一个文件并将您的 ansible-vault 密码写入其中。

```
echo "my-ansible-vault-pw" > ~/my-ansible-vault-pw-file
```



1. 为您的 VyOS 网络设备创建加密的 ssh 密码，从您刚刚创建的文件中提取您的 ansible-vault 密码。

```
ansible-vault encrypt_string --vault-id my_user@~/my-ansible-vault-pw-file 'VyOS_SSH_password' --name 'ansible_password'
```



如果您更愿意键入 ansible-vault 密码而不是将其存储在文件中，您可以请求提示。

```
ansible-vault encrypt_string --vault-id my_user@prompt 'VyOS_SSH_password' --name 'ansible_password'
```



并键入 my_user 的 vault 密码。

--vault-id 标志允许为不同的用户或不同的访问级别使用不同的 vault 密码。输出包含来自 ansible-vault 命令的用户名 my_user 并使用 YAML 语法 key: value。

```
ansible_password: !vault |
       $ANSIBLE_VAULT;1.2;AES256;my_user
       66386134653765386232383236303063623663343437643766386435663632343266393064373933
       3661666132363339303639353538316662616638356631650a316338316663666439383138353032
       63393934343937373637306162366265383461316334383132626462656463363630613832313562
       3837646266663835640a313164343535316666653031353763613037656362613535633538386539
       65656439626166666363323435613131643066353762333232326232323565376635
Encryption successful
```



这是一个使用 YAML 清单摘录的示例，因为 INI 格式不支持内联 vault。

```
...

vyos: # this is a group in yaml inventory, but you can also do under a host
  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: vyos.vyos.vyos
    ansible_user: my_vyos_user
    ansible_password:  !vault |
         $ANSIBLE_VAULT;1.2;AES256;my_user
         66386134653765386232383236303063623663343437643766386435663632343266393064373933
         3661666132363339303639353538316662616638356631650a316338316663666439383138353032
         63393934343937373637306162366265383461316334383132626462656463363630613832313562
         3837646266663835640a313164343535316666653031353763613037656362613535633538386539
         65656439626166666363323435613131643066353762333232326232323565376635

 ...
```



要将内联 vault 变量与 INI 清单一起使用，您需要将其存储在 YAML 格式的“vars”文件中，它可以位于 host_vars/ 或 group_vars/ 中以自动获取或通过 vars_files 或 include_vars 从 playbook 中引用。

要使用此设置运行 playbook，请删除 -k 标志并为您的 vault-id 添加标志。

```
ansible-playbook -i inventory --vault-id my_user@~/my-ansible-vault-pw-file first_playbook.yml
```



或者使用提示代替 vault 密码文件。

```
ansible-playbook -i inventory --vault-id my_user@prompt first_playbook.yml
```



要查看原始值，您可以使用 debug 模块。请注意，如果您的 YAML 文件定义了 ansible_connection 变量（正如我们在示例中所使用的），那么当您执行以下命令时，它将生效。为避免这种情况，请创建一个不包含 ansible_connection 变量的文件副本。

```
cat vyos.yml | grep -v ansible_connection >> vyos_no_connection.yml

ansible localhost -m debug -a var="ansible_password" -e "@vyos_no_connection.yml" --ask-vault-pass
Vault password:

localhost | SUCCESS => {
    "ansible_password": "VyOS_SSH_password"
}
```



有关构建清单文件的更多详细信息，请参阅 清单介绍；有关 ansible-vault 的更多详细信息，请参阅 完整的 Ansible Vault 文档。

现在您已经了解了命令、playbook 和清单的基础知识，是时候探索一些更复杂的 Ansible 网络示例了。
