---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 3.4 清单使用模式
order: 3
---

# 清单使用模式：目标主机和组

当您通过临时命令或运行 playbook 来执行 Ansible 时，必须选择要针对其执行的管理节点或组。模式允许您针对清单中的特定主机和/或组运行命令和 playbook。Ansible 模式可以指单个主机、IP 地址、清单组、一组组或清单中的所有主机。模式非常灵活——您可以排除或需要主机的子集，使用通配符或正则表达式等等。Ansible 将在模式中包含的所有清单主机上执行。



## 使用模式

几乎在任何时候执行临时命令或 playbook 时，您都会使用模式。模式是临时命令中唯一没有标志的元素。它通常是第二个元素。

```
ansible <pattern> -m <module_name> -a "<module options>"
```



例如

```
ansible webservers -m service -a "name=httpd state=restarted"
```



在 playbook 中，模式是每个 play 的hosts:行的内容。

```
- name: <play_name>
  hosts: <pattern>
```



例如

```
- name: restart webservers
  hosts: webservers
```



由于您经常希望一次对多个主机运行命令或 playbook，因此模式通常引用清单组。上面的临时命令和 playbook 都将在webservers组中的所有机器上执行。



## 常用模式

此表列出了用于定位清单主机和组的常用模式。

| 描述     | 模式                         | 目标                                               |
| -------- | ---------------------------- | -------------------------------------------------- |
| 所有主机 | all (或 *)                   |                                                    |
| 一台主机 | host1                        |                                                    |
| 多台主机 | host1:host2 (或 host1,host2) |                                                    |
| 一个组   | webservers                   |                                                    |
| 多个组   | webservers:dbservers         | webservers 中的所有主机加上 dbservers 中的所有主机 |
| 排除组   | webservers:!atlanta          | webservers 中的所有主机，除了 atlanta 中的主机     |
| 组的交集 | webservers:&staging          | webservers 中同时也在 staging 中的任何主机         |

一旦您了解了基本模式，就可以将它们组合起来。此示例

```
webservers:dbservers:&staging:!phoenix
```



目标是 'webservers' 和 'dbservers' 组中的所有机器，这些机器也在 'staging' 组中，但 'phoenix' 组中的任何机器除外。

只要在您的清单中按 FQDN 或 IP 地址命名主机，您就可以将通配符模式与 FQDN 或 IP 地址一起使用。

```
192.0.*
*.example.com
*.com
```



您可以同时混合使用通配符模式和组。

```
one*.com:dbservers
```



## 模式的限制

模式依赖于清单。如果清单中未列出主机或组，则无法使用模式将其作为目标。如果您的模式包含清单中未出现的 IP 地址或主机名，您将看到类似这样的错误：

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: Could not match supplied host pattern, ignoring: *.not_in_inventory.com
```



您的模式必须与您的清单语法匹配。如果您将主机定义为别名

```
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
      ansible_host: 127.0.0.2
```



则必须在模式中使用该别名。在上面的示例中，您必须在模式中使用host1。如果您使用 IP 地址，您将再次收到错误。

```
[WARNING]: Could not match supplied host pattern, ignoring: 127.0.0.2
```



## 模式处理顺序

处理方式有点特殊，按照以下顺序进行：

1. : 和 ,
2. &
3. !

此位置仅考虑每个操作内的处理顺序：a:b:&c:!d:!e == &c:a:!d:b:!e == !d:a:!e:&c:b

所有这些都导致以下结果：

主机在/属于 (a 或 b) 并且主机在/属于 all(c) 并且主机不在/不属于 all(d, e)。

现在a:b:!e:!d:&c略有变化，因为!e在!d之前处理，但这并没有什么区别。

主机在/属于 (a 或 b) 并且主机在/属于 all(c) 并且主机不在/不属于 all(e, d)。

## 高级模式选项

上面描述的常用模式可以满足您的大部分需求，但 Ansible 提供了几种其他方法来定义要定位的主机和组。

### 在模式中使用变量

您可以使用变量来启用使用-e参数传递组说明符到 ansible-playbook。

```
webservers:!{{ excluded }}:&{{ required }}
```



### 在模式中使用组位置

您可以通过其在组中的位置来定义主机或主机的子集。例如，给定以下组：

```
[webservers]
cobweb
webbing
weber
```



您可以使用下标来选择 webservers 组中的单个主机或范围。

#### 切片特定项目

- 操作：s[i]
- 结果：s的第i个项目，其中索引原点为0。

如果 i 为负数，则索引相对于序列 s 的结尾：len(s) + i 将被替换。但是-0是0。

```
webservers[0]       # == cobweb
webservers[-1]      # == weber
```



#### 使用起始点和结束点进行切片

- 操作：s[i:j]
- 结果：从i到j的s的切片。

从 i 到 j 的 s 的切片定义为索引为 k 的项目序列，使得i <= k <= j。如果省略 i，则使用0。如果省略 j，则使用len(s)。省略 i 和 j 的切片会导致无效的主机模式。如果 i 大于 j，则切片为空。如果 i 等于 j，则替换为 s[i]。

```
webservers[0:2]     # == webservers[0],webservers[1],webservers[2]
                    # == cobweb,webbing,weber
webservers[1:2]     # == webservers[1],webservers[2]
                    # == webbing,weber
webservers[1:]      # == webbing,weber
webservers[:3]      # == cobweb,webbing,weber
```



### 在模式中使用正则表达式

您可以通过以~开头模式来将模式指定为正则表达式。

```
~(web|db).*\.example\.com
```



## 模式和临时命令

您可以使用命令行选项更改在临时命令中定义的模式的行为。您还可以使用--limit标志限制在特定运行中定位的主机。

- 限制为一台主机

```
$ ansible all -m <module> -a "<module options>" --limit "host1"
```



- 限制到多个主机

```
$ ansible all -m <module> -a "<module options>" --limit "host1,host2"
```



- 否定限制。注意，必须使用单引号以防止 bash 插值。

```
$ ansible all -m <module> -a "<module options>" --limit 'all:!host1'
```



- 限制到主机组

```
$ ansible all -m <module> -a "<module options>" --limit 'group1'
```



## 模式和 ansible-playbook 标志

您可以使用命令行选项更改 playbook 中定义的模式的行为。例如，您可以通过指定 -i 127.0.0.2,（注意尾随逗号）在一个主机上运行一个 playbook，该 playbook 定义了 hosts: all。即使您目标主机未在您的清单中定义，此方法也能工作，但是此方法不会读取与该主机绑定的变量的清单，并且 playbook所需的任何变量都需要在命令行手动指定。您还可以使用 --limit 标志限制您在特定运行中定位的主机，该标志将引用您的清单。

```
ansible-playbook site.yml --limit datacenter2
```



最后，您可以使用 --limit 通过在文件名之前添加 @ 来从文件中读取主机列表。

```
ansible-playbook site.yml --limit @retry_hosts.txt
```



如果 RETRY_FILES_ENABLED 设置为 True，则在 ansible-playbook 运行后将创建一个 .retry 文件，其中包含来自所有剧集的所有失败主机的列表。每次 ansible-playbook 运行完成时，都会覆盖此文件。

```
ansible-playbook site.yml --limit @site.retry
```



要将您对模式的了解应用于 Ansible 命令和 playbook，请阅读 临时命令介绍 和 Ansible playbook。
