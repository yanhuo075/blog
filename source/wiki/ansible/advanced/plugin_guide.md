---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.5 Ansible 插件使用
order: 4
---

# 使用插件

插件是增强 Ansible 核心功能的代码片段。Ansible 使用插件架构来实现丰富、灵活和可扩展的功能集。

Ansible 附带许多方便的插件，您可以轻松编写自己的插件。

本节介绍 Ansible 中包含的各种类型的插件。

- 动作插件
- Become 插件
- 缓存插件
- 回调插件
- Cliconf 插件
- 连接插件
- 文档片段
- 过滤器插件
- Httpapi 插件
- 清单插件
- 查找插件
- 模块
- 模块实用程序
- Netconf 插件
- Shell 插件
- 策略插件
- 终端插件
- 测试插件
- 变量插件



## Action 插件

- 启用 Action 插件
- 使用 Action 插件
- 插件列表

Action 插件与模块协同工作，以执行 playbook 任务所需的动作。它们通常在后台自动执行，在模块执行之前进行必要的预备工作。

“normal” action 插件用于尚无 action 插件的模块。如有必要，您可以创建自定义 action 插件。



### 启用 Action 插件

您可以通过将自定义 action 插件放入与您的 playbook 相邻的action_plugins目录、角色内部，或将其放入ansible.cfg中配置的 action 插件目录源之一来启用它。



### 使用 Action 插件

使用关联模块时，Action 插件默认情况下会执行；无需其他操作。

### 插件列表

您无法直接列出 action 插件，它们显示为其对应的模块。

使用ansible-doc -l查看可用模块的列表。使用ansible-doc <name>查看特定插件的文档和示例。这应指出该模块是否具有相应的 action 插件。



## Become 插件

- 启用 Become 插件
- 使用 Become 插件
- 插件列表

2.8 版本新增。

Become 插件确保 Ansible 在运行与目标机器交互的基本命令以及执行 playbook 中指定任务所需的模块时，可以使用某些权限提升系统。

这些实用程序（sudo、su、doas 等）通常允许您“成为”另一个用户，以该用户的权限执行命令。



### 启用 Become 插件

Ansible 附带的 Become 插件已启用。自定义插件可以通过将它们放置到 playbook 旁边的 become_plugins 目录中、角色内部，或者将它们放置到 ansible.cfg 中配置的某个 Become 插件目录源中来添加。



### 使用 Become 插件

除了 Ansible 配置设置 中的默认配置设置或 --become-method 命令行选项之外，您还可以使用 playbook 中的 become_method 关键字，或者如果您需要“主机特定”设置，则可以使用连接变量 ansible_become_method 来选择要使用的插件。

您可以使用插件本身中详细说明的其他配置选项（链接如下）进一步控制每个插件的设置。



### 插件列表

您可以使用 ansible-doc -t become -l 查看可用插件的列表。使用 ansible-doc -t become <plugin name> 查看特定插件的文档和示例。



## 缓存插件

- 启用事实缓存插件
- 启用清单缓存插件
- 使用缓存插件
- 插件列表

缓存插件允许 Ansible 存储收集到的事实或清单源数据，而不会因从源检索它们而导致性能下降。

默认的缓存插件是 内存 插件，它只缓存 Ansible 当前执行的数据。其他具有持久存储的插件可用于跨运行缓存数据。一些缓存插件写入文件，另一些写入数据库。

您可以对清单和事实使用不同的缓存插件。如果您启用清单缓存而没有设置特定于清单的缓存插件，Ansible 将事实缓存插件用于事实和清单。如有必要，您可以 创建自定义缓存插件。



### 启用事实缓存插件

事实缓存始终启用。但是，一次只能激活一个事实缓存插件。您可以在 Ansible 配置中选择用于事实缓存的缓存插件，方法是使用环境变量

```
export ANSIBLE_CACHE_PLUGIN=jsonfile
```



或在 ansible.cfg 文件中

```
[defaults]
fact_caching=redis
```



如果缓存插件位于集合中，请使用完全限定名称

```
[defaults]
fact_caching = namespace.collection_name.cache_plugin_name
```



要启用自定义缓存插件，请将其保存在 ansible.cfg 中配置的目录源之一或集合中，然后通过 FQCN 引用它。

您还需要配置每个插件特有的其他设置。有关更多详细信息，请参阅各个插件文档或 Ansible 配置。

### 启用清单缓存插件

清单缓存默认情况下处于禁用状态。要缓存清单数据，您必须启用清单缓存，然后选择要使用的特定缓存插件。并非所有清单插件都支持缓存，因此请检查您要使用的清单插件的文档。您可以使用环境变量启用清单缓存

```
export ANSIBLE_INVENTORY_CACHE=True
```



或在 ansible.cfg 文件中

```
[inventory]
cache=True
```



或者，如果清单插件接受 YAML 配置源，则在配置文件中

```
# dev.aws_ec2.yaml
plugin: aws_ec2
cache: True
```



一次只能激活一个清单缓存插件。您可以使用环境变量设置它

```
export ANSIBLE_INVENTORY_CACHE_PLUGIN=jsonfile
```



或在 ansible.cfg 文件中

```
[inventory]
cache_plugin=jsonfile
```



或者，如果清单插件接受 YAML 配置源，则在配置文件中

```
# dev.aws_ec2.yaml
plugin: aws_ec2
cache_plugin: jsonfile
```



要使用插件路径中的自定义插件缓存清单，请遵循 有关缓存插件的开发者指南。

要使用集合中的缓存插件缓存清单，请使用 FQCN

```
[inventory]
cache_plugin=collection_namespace.collection_name.cache_plugin
```



如果您在没有选择特定于清单的缓存插件的情况下为清单插件启用缓存，Ansible 将回退到使用您配置的事实缓存插件缓存清单。有关更多详细信息，请参阅各个清单插件文档或 Ansible 配置。



### 使用缓存插件

启用缓存插件后，它们将自动使用。



### 插件列表

您可以使用 ansible-doc -t cache -l 查看可用插件的列表。使用 ansible-doc -t cache <plugin name> 查看特定于插件的文档和示例。



## 回调插件

- 回调插件示例
- 启用回调插件
- 为 ansible-playbook 设置回调插件
- 为临时命令设置回调插件
- 回调插件类型
- 插件列表

回调插件允许在响应事件时向 Ansible 添加新行为。默认情况下，回调插件控制运行命令行程序时看到的大部分输出，但也可以用于添加其他输出、与其他工具集成以及将事件编组到存储后端。如有必要，您可以创建自定义回调插件。



### 回调插件示例

log_plays 回调插件演示了如何将剧本事件记录到日志文件，而 mail 回调插件会在剧本失败时发送电子邮件。

say 回调插件会根据剧本事件回应计算机合成的语音。



### 启用回调插件

您可以根据插件的 NEEDS_ENABLED 属性，通过将其放入 ansible.cfg 中配置的回调目录源之一或放入集合中并在配置中通过 FQCN 引用它来激活自定义回调。

插件按字母数字顺序加载。例如，在名为 1_first.py 的文件中实现的插件将在名为 2_second.py 的插件文件之前运行。

Ansible 附带的大多数回调插件默认情况下都是禁用的，需要在您的 ansible.cfg 文件中启用才能正常工作。例如：

```
#callbacks_enabled = timer, mail, profile_roles, collection_namespace.collection_name.custom_callback
```



### 为 ansible-playbook 设置回调插件

您只能有一个插件作为控制台输出的主要管理器。如果要替换默认插件，应在子类中定义 CALLBACK_TYPE = stdout，然后在 ansible.cfg 中配置 stdout 插件。例如：

```
stdout_callback = dense
```



或者对于我的自定义回调：

```
stdout_callback = mycallback
```



这默认情况下只影响 ansible-playbook。

### 为临时命令设置回调插件

临时命令 ansible 特别使用不同的 stdout 回调插件，因此在 Ansible 配置设置 中需要添加额外的设置才能使用上面定义的 stdout 回调插件：

```
[defaults]
bin_ansible_callbacks=True
```



您也可以将其设置为环境变量：

```
export ANSIBLE_LOAD_CALLBACK_PLUGINS=1
```





### 回调插件类型

有三种类型的回调插件：

- stdout 回调插件：

  这些插件处理主要的控制台输出。一次只能激活一个这样的插件。

- 聚合回调插件：

  聚合回调插件可以在 stdout 回调插件旁边添加额外的控制台输出。这可以是剧本运行结束时的聚合信息、每个任务的额外输出或其他任何内容。

- 通知回调插件：

  通知回调插件会通知其他应用程序、服务或系统。这可以是任何内容，从记录到数据库，到在即时消息应用程序中告知错误，或在服务器无法访问时发送电子邮件。



### 插件列表

您可以使用 ansible-doc -t callback -l 查看可用的插件列表。使用 ansible-doc -t callback <plugin name> 查看特定插件的文档和示例。



## Cliconf 插件

- 添加 cliconf 插件
- 使用 cliconf 插件
- 查看 cliconf 插件

Cliconf 插件是对网络设备 CLI 接口的抽象。它们为 Ansible 提供了一个标准接口，用于在这些网络设备上执行任务。

这些插件通常与网络设备平台一一对应。Ansible 会根据ansible_network_os变量自动加载相应的 cliconf 插件。



### 添加 cliconf 插件

您可以通过将自定义插件放入cliconf_plugins目录来扩展 Ansible 以支持其他网络设备。



### 使用 cliconf 插件

要使用的 cliconf 插件将根据ansible_network_os变量自动确定。无需覆盖此功能。

大多数 cliconf 插件无需配置即可运行。一些插件有一些额外的选项，可以设置这些选项来影响任务如何转换为 CLI 命令。

插件是自文档化的。每个插件都应该记录其配置选项。



### 查看 cliconf 插件

这些插件已迁移到 [Ansible Galaxy](https://galaxy.ansible.com/) 上的集合。如果您使用 `pip`安装了 Ansible 2.10 或更高版本，则可以使用多个 cliconf 插件。您可以使用 `ansible-doc -t cliconf -l` 查看可用插件列表。使用 `ansible-doc -t cliconf <plugin name>` 查看特定插件的文档和示例。





## 连接插件

- ssh 插件
- 使用连接插件
- 插件列表

连接插件允许 Ansible 连接到目标主机，以便在其上执行任务。Ansible 自带许多连接插件，但每次只能为每个主机使用一个。

默认情况下，Ansible 自带多个连接插件。最常用的包括 paramiko SSH、原生 ssh（简称 ssh）和 local 连接类型。所有这些都可以在 playbook 和 /usr/bin/ansible 中使用，以决定如何与远程机器通信。如有必要，您可以 创建自定义连接插件。要更改任务的连接插件，可以使用 connection 关键字。

这些连接类型的基础知识在 入门 部分进行了介绍。



### ssh 插件

由于 SSH 是系统管理中使用的默认协议，也是 Ansible 中使用最多的协议，因此命令行工具中包含了 SSH 选项。有关更多详细信息，请参阅 ansible-playbook。



### 使用连接插件

您可以使用 配置 在全局范围内设置连接插件，在命令行 (-c, --connection) 中设置，作为 playbook 中的 关键字 设置，或者通过设置 变量（通常在清单中）来设置。例如，对于 Windows 机器，您可能希望将 winrm 插件设置为清单变量。

大多数连接插件都可以使用最少的配置进行操作。默认情况下，它们使用 清单主机名 并使用默认值查找目标主机。

插件具有自文档功能。每个插件都应记录其配置选项。以下是大多数连接插件通用的连接变量

- ansible_host

  要连接到的主机名，如果与 清单 主机名不同。

- ansible_port

  ssh 端口号，对于 ssh 和 paramiko_ssh，默认为 22。

- ansible_user

  用于登录的默认用户名。大多数插件默认为“运行 Ansible 的当前用户”。

每个插件也可能具有特定版本的变量，该变量会覆盖通用版本。例如，ansible_ssh_host 用于 ssh 插件。



### 插件列表

您可以使用 ansible-doc -t connection -l 查看可用插件的列表。使用 ansible-doc -t connection <plugin name> 查看特定插件的文档和示例。



## 文档片段

- 启用文档片段
- 使用文档片段

文档片段允许您在一个地方记录多个插件或模块的常用参数。



### 启用文档片段

您可以通过将自定义文档片段放入与您的集合或角色相邻的doc_fragments目录中来添加它，就像任何其他插件一样。



### 使用文档片段

只有集合开发者和维护者使用文档片段。有关使用文档片段的更多信息，请参见[文档片段](https://docs.ansible.org.cn/ansible/latest/dev_guide/developing_modules_documenting.html#module-docs-fragments)或[在集合中使用文档片段](https://docs.ansible.org.cn/ansible/latest/dev_guide/developing_collections_shared.html#docfragments-collections)。



## 过滤器插件

- 启用过滤器插件
- 使用过滤器插件
- 插件列表

过滤器插件用于操作数据。使用正确的过滤器，您可以提取特定值、转换数据类型和格式、执行数学计算、拆分和连接字符串、插入日期和时间等等。Ansible 使用 Jinja2 附带的 标准过滤器，并添加了一些专门的过滤器插件。您可以创建自定义 Ansible 过滤器作为插件。



### 启用过滤器插件

您可以通过将自定义过滤器插件放入与您的 playbook 相邻的 filter_plugins 目录中、角色内部，或者将其放入在 ansible.cfg 中配置的过滤器插件目录源之一中来添加它。



### 使用过滤器插件

您可以在 Ansible 中可以使用模板的任何地方使用过滤器：在 playbook 中、在变量文件中，或者在 template 模块的 Jinja2 模板中。有关使用过滤器插件的更多信息，请参阅使用过滤器操作数据。过滤器可以返回任何类型的数据，但是如果您想始终返回布尔值（true 或 false），则应该查看测试。

```
vars:
   yaml_string: "{{ some_variable|to_yaml }}"
```



过滤器是在 Ansible 中操作数据的首选方式，您可以识别过滤器，因为它通常以 | 开头，其左侧的表达式是过滤器的第一个输入。其他参数可以像传递给大多数编程函数一样传递到过滤器本身。这些参数可以是 位置参数（按顺序传递）或 命名参数（以键=值对传递）。当同时传递两种类型时，位置参数应放在前面。

```
passing_positional: {{ (x == 32) | ternary('x is 32', 'x is not 32') }}
passing_extra_named_parameters: {{ some_variable | to_yaml(indent=8, width=1337) }}
passing_both: {{ some_variable| ternary('true value', 'false value', none_val='NULL') }}
```



在文档中，过滤器始终会有一个 C(_input) 选项，该选项对应于 c(|) 左侧的表达式。文档中的 C(位置:) 字段将显示哪些选项是位置的以及它们的必需顺序。

### 插件列表

您可以使用 ansible-doc -t filter -l 查看可用插件的列表。使用 ansible-doc -t filter <插件名称> 查看特定于插件的文档和示例。



## Httpapi 插件

- 添加 Httpapi 插件
- 使用 Httpapi 插件
- 查看 Httpapi 插件

Httpapi 插件告诉 Ansible 如何与远程设备的基于 HTTP 的 API 交互并在设备上执行任务。

每个插件代表一种特定的 API 方言。 一些是特定于平台的 (Arista eAPI、Cisco NXAPI)，而另一些可能适用于各种平台 (RESTCONF)。 Ansible 会根据 ansible_network_os 变量自动加载适当的 httpapi 插件。



### 添加 httpapi 插件

您可以通过将自定义插件放入 httpapi_plugins 目录来扩展 Ansible 以支持其他 API。 有关详细信息，请参阅开发 httpapi 插件。



### 使用 httpapi 插件

要使用的 httpapi 插件由 ansible_network_os 变量自动确定。

大多数 httpapi 插件都可以在没有配置的情况下运行。 每个插件可能会定义其他选项。

插件是自我记录的。 每个插件都应记录其配置选项。

以下示例 playbook 显示了 Arista 网络设备的 httpapi 插件，假设库存变量设置为 ansible_network_os=eos 以触发 httpapi 插件

```
- hosts: leaf01
  connection: httpapi
  gather_facts: false
  tasks:

    - name: type a simple arista command
      eos_command:
        commands:
          - show version | json
      register: command_output

    - name: print command output to terminal window
      debug:
        var: command_output.stdout[0]["version"]
```



请参阅 GitHub 上完整的运行示例。



### 查看 httpapi 插件

这些插件已迁移到 Ansible Galaxy 上的集合。 如果您使用 pip 安装了 Ansible 2.10 或更高版本，则可以访问多个 httpapi 插件。 您可以使用 ansible-doc -t httpapi -l 查看可用插件列表。 使用 ansible-doc -t httpapi <插件 名称> 查看特定于插件的文档和示例。



## 清单插件

- 启用清单插件
- 使用清单插件
- 插件列表

清单插件允许用户指向数据源，以编译 Ansible 用于定位任务的主机清单，可以使用 -i /path/to/file 和/或 -i 'host1, host2' 命令行参数或来自其他配置源。如有必要，您可以创建自定义清单插件。



### 启用清单插件

Ansible 附带的大多数清单插件默认启用，或可以通过 auto 插件使用。

在某些情况下，例如，如果清单插件不使用 YAML 配置文件，您可能需要启用特定的插件。您可以通过在 ansible.cfg 文件中的 [inventory] 部分设置 enable_plugins 来完成此操作。修改此项将覆盖默认的已启用插件列表。以下是 Ansible 附带的默认已启用插件列表

```
[inventory]
enable_plugins = host_list, script, auto, yaml, ini, toml
```



如果插件位于集合中，并且未被 auto 语句选取，则可以追加完全限定的名称

```
[inventory]
enable_plugins = host_list, script, auto, yaml, ini, toml, namespace.collection_name.inventory_plugin_name
```



或者，如果它是本地插件，可能存储在 DEFAULT_INVENTORY_PLUGIN_PATH 设置的路径中，您可以按如下方式引用它

```
[inventory]
enable_plugins = host_list, script, auto, yaml, ini, toml, my_plugin
```



如果您使用支持 YAML 配置源的插件，请确保名称与清单源文件中 plugin 条目中提供的名称匹配。对于其他插件，您必须将其保存在 ansible.cfg 中配置的目录源之一中并启用它，或者添加到集合中，然后通过 FQCN 引用它。



### 使用清单插件

要使用清单插件，您必须提供清单源。大多数情况下，这是一个包含主机信息的文件或带有插件选项的 YAML 配置文件。您可以使用 -i 标志来提供清单源或配置默认清单路径。

```
ansible hostname -i inventory_source -m ansible.builtin.ping
```



要开始使用具有 YAML 配置源的清单插件，请创建一个具有针对所讨论插件记录的可接受文件名模式的文件，然后添加 plugin: plugin_name。如果插件在集合中，请使用完全限定的名称。

```
# demo.aws_ec2.yml
plugin: amazon.aws.aws_ec2
```



每个插件都应记录任何命名限制。此外，YAML 配置文件必须以扩展名 yml 或 yaml 结尾，才能默认使用 auto 插件启用（否则，请参阅上面关于启用插件的部分）。

提供任何必需的选项后，您可以使用 ansible-inventory -i demo.aws_ec2.yml --graph 查看填充的清单

```
@all:
  |--@aws_ec2:
  |  |--ec2-12-345-678-901.compute-1.amazonaws.com
  |  |--ec2-98-765-432-10.compute-1.amazonaws.com
  |--@ungrouped:
```



如果您在剧本相邻的集合中使用清单插件，并且想使用 ansible-inventory 测试您的设置，请使用 --playbook-dir 标志。

您的清单源可能是清单配置文件的目录。构造的清单插件仅对清单中已有的主机进行操作，因此您可能希望在特定点（例如最后）解析构造的清单配置。Ansible 以递归方式按字母顺序解析目录。您无法配置解析方法，因此请命名您的文件以使其可预测地工作。直接扩展构造功能的清单插件可以通过添加构造选项以及清单插件选项来解决该限制。否则，您可以使用带有多个源的 -i 来强制执行特定顺序，例如 -i demo.aws_ec2.yml -i clouds.yml -i constructed.yml。

您可以使用带有构造的 keyed_groups 选项的主机变量创建动态组。groups 选项也可用于创建组，compose 创建和修改主机变量。以下是利用构造功能的 aws_ec2 示例

```
# demo.aws_ec2.yml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-east-2
keyed_groups:
  # add hosts to tag_Name_value groups for each aws_ec2 host's tags.Name variable
  - key: tags.Name
    prefix: tag_Name_
    separator: ""
  # If you have a tag called "Role" which has the value "Webserver", this will add the group
  # role_Webserver and add any hosts that have that tag assigned to it.
  - key: tags.Role
    prefix: role
groups:
  # add hosts to the group development if any of the dictionary's keys or values is the word 'devel'
  development: "'devel' in (tags|list)"
  # add hosts to the "private_only" group if the host doesn't have a public IP associated to it
  private_only: "public_ip_address is not defined"
compose:
  # use a private address where a public one isn't assigned
  ansible_host: public_ip_address|default(private_ip_address)
  # alternatively, set the ansible_host variable to connect with the private IP address without changing the hostname
  # ansible_host: private_ip_address
  # if you *must* set a string here (perhaps to identify the inventory source if you have multiple
  # accounts you want to use as sources), you need to wrap this in two sets of quotes, either ' then "
  # or " then '
  some_inventory_wide_string: '"Yes, you need both types of quotes here"'
```



现在 ansible-inventory -i demo.aws_ec2.yml --graph 的输出

```
@all:
  |--@aws_ec2:
  |  |--ec2-12-345-678-901.compute-1.amazonaws.com
  |  |--ec2-98-765-432-10.compute-1.amazonaws.com
  |  |--...
  |--@development:
  |  |--ec2-12-345-678-901.compute-1.amazonaws.com
  |  |--ec2-98-765-432-10.compute-1.amazonaws.com
  |--@role_Webserver
  |  |--ec2-12-345-678-901.compute-1.amazonaws.com
  |--@tag_Name_ECS_Instance:
  |  |--ec2-98-765-432-10.compute-1.amazonaws.com
  |--@tag_Name_Test_Server:
  |  |--ec2-12-345-678-901.compute-1.amazonaws.com
  |--@ungrouped
```



如果主机在上面的配置中没有变量（换句话说，tags.Name、tags、private_ip_address），则该主机将不会添加到清单插件创建的组之外的组，并且 ansible_host 主机变量将不会被修改。

支持缓存的清单插件可以使用 ansible.cfg 文件的 [defaults] 部分中定义的 fact 缓存的常规设置，或者在 [inventory] 部分中定义特定于清单的设置。各个插件可以在其配置文件中定义特定于插件的缓存设置

```
# demo.aws_ec2.yml
plugin: amazon.aws.aws_ec2
cache: true
cache_plugin: ansible.builtin.jsonfile
cache_timeout: 7200
cache_connection: /tmp/aws_inventory
cache_prefix: aws_ec2
```



以下是在 ansible.cfg 文件中设置清单缓存以及用于缓存插件和超时的一些 fact 缓存默认值的示例

```
[defaults]
fact_caching = ansible.builtin.jsonfile
fact_caching_connection = /tmp/ansible_facts
cache_timeout = 3600

[inventory]
cache = yes
cache_connection = /tmp/ansible_inventory
```



### 插件列表

您可以使用 ansible-doc -t inventory -l 查看可用插件的列表。使用 ansible-doc -t inventory <plugin name> 查看特定于插件的文档和示例。



## 查找插件

- 启用查找插件
- 使用查找插件
- 强制查找返回列表：query 和 wantlist=True
- 插件列表

查找插件是 Jinja2 模板语言的 Ansible 特定扩展。您可以使用查找插件来访问来自外部源（文件、数据库、键/值存储、API 和其他服务）的数据，在您的 Playbook 中。与所有模板一样，查找在 Ansible 控制机器上执行和评估。Ansible 使用标准模板系统使查找插件返回的数据可用。您可以使用查找插件从外部源加载变量或模板信息。您可以创建自定义查找插件。



### 启用查找插件

Ansible 会启用它可以找到的所有查找插件。您可以通过将自定义查找插件放到与您的 Play 相邻的 lookup_plugins 目录中，或者放在您已安装的集合的 plugins/lookup/ 目录中，或者放在一个独立的 Role 中，或者在 ansible.cfg 中配置的查找目录源之一中来激活它。



### 使用查找插件

您可以在 Ansible 中可以使用模板的任何地方使用查找插件：在 Play 中，在变量文件中，或者用于 模板 模块的 Jinja2 模板中。有关使用查找插件的更多信息，请参阅查找。

```
vars:
  file_contents: "{{ lookup('file', 'path/to/file.txt') }}"
```



查找是循环的组成部分。无论您在哪里看到 with_，下划线后面的部分都是查找的名称。因此，查找预计会输出列表；例如，with_items 使用 items 查找

```
tasks:
  - name: count to 3
    debug: msg={{ item }}
    with_items: [1, 2, 3]
```



您可以将查找与过滤器、测试 甚至彼此结合使用，以进行一些复杂的数据生成和操作。例如

```
tasks:
  - name: Complicated chained lookups and filters
    debug: msg="find the answer here:\n{{ lookup('url', 'https://google.com/search/?q=' + item|urlencode)|join(' ') }}"
    with_nested:
      - "{{ lookup('consul_kv', 'bcs/' + lookup('file', '/the/question') + ', host=localhost, port=2000')|shuffle }}"
      - "{{ lookup('sequence', 'end=42 start=2 step=2')|map('log', 4)|list) }}"
      - ['a', 'c', 'd', 'c']
```



2.6 版本新增功能。

您可以通过将 errors 设置为 ignore、warn 或 strict 来控制所有查找插件中错误的行为方式。默认设置为 strict，如果查找返回错误，则会导致任务失败。例如

忽略查找错误

```
- name: if this file does not exist, I do not care .. file plugin itself warns anyway ...
  debug: msg="{{ lookup('file', '/nosuchfile', errors='ignore') }}"
```



```
[WARNING]: Unable to find '/nosuchfile' in expected paths (use -vvvvv to see paths)

ok: [localhost] => {
    "msg": ""
}
```



获取警告而不是失败

```
- name: if this file does not exist, let me know, but continue
  debug: msg="{{ lookup('file', '/nosuchfile', errors='warn') }}"
```



```
[WARNING]: Unable to find '/nosuchfile' in expected paths (use -vvvvv to see paths)

[WARNING]: An unhandled exception occurred while running the lookup plugin 'file'. Error was a <class 'ansible.errors.AnsibleError'>, original message: could not locate file in lookup: /nosuchfile

ok: [localhost] => {
    "msg": ""
}
```



获取致命错误（默认）

```
- name: if this file does not exist, FAIL (this is the default)
  debug: msg="{{ lookup('file', '/nosuchfile', errors='strict') }}"
```



```
[WARNING]: Unable to find '/nosuchfile' in expected paths (use -vvvvv to see paths)

fatal: [localhost]: FAILED! => {"msg": "An unhandled exception occurred while running the lookup plugin 'file'. Error was a <class 'ansible.errors.AnsibleError'>, original message: could not locate file in lookup: /nosuchfile"}
```





### 强制查找返回列表：query 和 wantlist=True

2.5 版本新增功能。

在 Ansible 2.5 中，添加了一个名为 query 的新 Jinja2 函数，用于调用查找插件。lookup 和 query 之间的主要区别在于 query 始终返回一个列表。 lookup 的默认行为是返回一个逗号分隔的值的字符串。lookup 可以使用 wantlist=True 显式配置为返回列表。

此功能为与新的 loop 关键字交互提供了一个更简单、更一致的接口，同时保持与其他 lookup 用法的向后兼容性。

以下示例等效

```
lookup('dict', dict_variable, wantlist=True)

query('dict', dict_variable)
```



如上所示，当使用 query 时，wantlist=True 的行为是隐式的。

此外，引入了 q 作为 query 的简写形式

```
q('dict', dict_variable)
```



### 插件列表

您可以使用 ansible-doc -t lookup -l 查看可用插件的列表。 使用 ansible-doc -t lookup <插件 名称> 查看插件特定的文档和示例。



## Netconf 插件

- 添加 Netconf 插件
- 使用 Netconf 插件
- 列出 Netconf 插件

Netconf 插件是对网络设备的 Netconf 接口的抽象。它们为 Ansible 提供了一个标准接口，以便在这些网络设备上执行任务。

这些插件通常与网络设备平台一一对应。Ansible 根据 ansible_network_os 变量自动加载相应的 netconf 插件。如果该平台支持 Netconf RFC 规范中定义的标准 Netconf 实现，Ansible 将加载 default netconf 插件。如果该平台支持专有的 Netconf RPC，Ansible 将加载特定于平台的 netconf 插件。



### 添加 Netconf 插件

您可以通过将自定义插件放入 netconf_plugins 目录来扩展 Ansible 以支持其他网络设备。



### 使用 Netconf 插件

要使用的 netconf 插件由 ansible_network_os 变量自动确定。没有理由覆盖此功能。

大多数 netconf 插件无需配置即可运行。少数插件具有额外的选项，可以设置这些选项来影响任务如何转换为 netconf 命令。可以在 netconf 插件中设置一个 ncclient 设备特定的处理程序名称，否则将按照 ncclient 设备处理程序使用 default 的值。

插件是自文档化的。每个插件都应记录其配置选项。



### 列出 Netconf 插件

这些插件已迁移到 Ansible Galaxy 上的集合。如果您使用 pip 安装了 Ansible 2.10 或更高版本，则可以访问多个 netconf 插件。您可以使用 ansible-doc -t netconf -l 查看可用插件的列表。使用 ansible-doc -t netconf <插件名称> 查看特定于插件的文档和示例。



## Shell 插件

- 启用 Shell 插件
- 使用 Shell 插件
- 插件列表

Shell 插件的作用是确保 Ansible 运行的基本命令格式正确，以便与目标机器配合使用，并允许用户配置与 Ansible 执行任务方式相关的某些行为。



### 启用 Shell 插件

您可以通过将自定义 shell 插件放入与您的 playbook 相邻的 shell_plugins 目录中、放入角色中，或者将其放置在 ansible.cfg 中配置的 shell 插件目录源之一中来添加该插件。



### 使用 Shell 插件

除了 Ansible 配置设置中的默认配置设置外，您还可以使用连接变量 ansible_shell_type 来选择要使用的插件。 在这种情况下，您还需要更新 ansible_shell_executable 以匹配。

### 插件列表

您可以使用插件本身中详细介绍的其他配置选项来进一步控制每个插件的设置。 您可以使用 `ansible-doc -t shell -l` 查看可用插件的列表。 使用 `ansible-doc -t shell <插件名称>` 查看特定于插件的文档和示例。



## 策略插件

- 启用策略插件
- 使用策略插件
- 插件列表

策略插件通过处理任务和主机调度来控制 Playbook 执行的流程。有关使用策略插件和其他控制执行顺序的方法的更多信息，请参阅控制 Playbook 执行：策略等等。



### 启用策略插件

Ansible 随附的所有策略插件默认启用。 您可以通过将自定义策略插件放入在ansible.cfg中配置的查找目录源之一来启用它。



### 使用策略插件

在一个 Play 中只能使用一个策略插件，但您可以在一个 Playbook 或 Ansible 运行中的每个 Play 中使用不同的插件。 默认情况下，Ansible 使用 linear 插件。您可以使用环境变量在 Ansible 配置中更改此默认设置

```
export ANSIBLE_STRATEGY=free
```



或在 ansible.cfg 文件中更改

```
[defaults]
strategy=linear
```



您还可以在 Play 中使用 strategy 关键字 来指定策略插件

```
- hosts: all
  strategy: debug
  tasks:
    - copy:
        src: myhosts
        dest: /etc/hosts
      notify: restart_tomcat

    - package:
        name: tomcat
        state: present

  handlers:
    - name: restart_tomcat
      service:
        name: tomcat
        state: restarted
```



### 插件列表

您可以使用 ansible-doc -t strategy -l 来查看可用插件列表。使用 ansible-doc -t strategy <插件名称> 来查看插件的特定文档和示例。



## 终端插件

- 添加终端插件
- 使用终端插件

终端插件包含如何确保特定网络设备的 SSH shell 正确初始化以与 Ansible 一起使用的信息。这通常包括禁用自动分页、检测输出中的错误，并在设备支持且需要时启用特权模式。

这些插件与网络设备平台一一对应。Ansible 根据 ansible_network_os 变量自动加载相应的终端插件。



### 添加终端插件

你可以通过将自定义插件放入 terminal_plugins 目录来扩展 Ansible 以支持其他网络设备。



### 使用终端插件

Ansible 根据 `ansible_network_os` 变量自动确定要使用哪个终端插件。没有理由需要覆盖此功能。

终端插件在没有配置的情况下运行。所有控制终端的选项都暴露在 `network_cli` 连接插件中。

插件是自我文档化的。每个插件都应记录其配置选项。



## 测试插件

- 启用测试插件
- 使用测试插件
  - 在列表上使用测试插件
- 插件列表

测试插件评估模板表达式并返回 True 或 False。 使用测试插件，您可以创建 条件 来实现您的任务、块、剧本、playbook 和角色的逻辑。 Ansible 使用作为 Jinja 一部分提供的 标准测试，并添加了一些专门的测试插件。 您可以创建自定义 Ansible 测试插件。



### 启用测试插件

您可以通过将自定义测试插件放入与您的剧本相邻的 test_plugins 目录、角色内部，或将其放入 ansible.cfg 中配置的测试插件目录源之一中来添加自定义测试插件。



### 使用测试插件

您可以在 Ansible 中可以使用模板的任何地方使用测试：在剧本中、变量文件中或 template 模块的 Jinja2 模板中。 有关使用测试插件的更多信息，请参阅测试。

测试总是返回 True 或 False，它们始终是布尔值，如果您需要不同的返回类型，则应该查看过滤器。

您可以通过在模板中使用 is 语句来识别测试插件，它们也可以用作 select 系列过滤器的一部分。

```
vars:
  is_ready: '{{ task_result is success }}'

tasks:
- name: conditionals are always in 'template' context
  action: dostuff
  when: task_result is failed
```



测试始终具有一个 _input，这通常是 is 左侧的内容。测试还可以像大多数编程函数一样采用其他参数。这些参数可以是 位置 的（按顺序传递）或 命名 的（作为键=值对传递）。当传递两种类型时，位置参数应该放在前面。

```
tasks:
- name: pass a positional parameter to match test
  action: dostuff
  when: myurl is match("https://example.com/users/.*/resources")

- name: pass named parameter to truthy test
  action: dostuff
  when: myvariable is truthy(convert_bool=True)

- name: pass both types to 'version' test
  action: dostuff
  when: sample_semver_var is version('2.0.0-rc.1+build.123', 'lt', version_type='semver')
```



### 在列表上使用测试插件

如上所述，使用测试的一种方法是使用 select 系列过滤器 (select, reject, selectattr, rejectattr)。

```
# give me only defined variables from a list of variables, using 'defined' test
good_vars: "{{ all_vars|select('defined') }}"

# this uses the 'equalto' test to filter out non 'fixed' type of addresses from a list
only_fixed_addresses:  "{{ all_addresses|selectattr('type', 'equalto', 'fixed') }}"

# this does the opposite of the previous one
only_fixed_addresses:  "{{ all_addresses|rejectattr('type', 'equalto', 'fixed') }}"
```



### 插件列表

您可以使用 `ansible-doc -t test -l` 查看可用插件的列表。 使用 `ansible-doc -t test <插件名称>` 查看特定于插件的文档和示例。



## Vars 插件

- 启用 Vars 插件
- 使用 Vars 插件
- 插件列表

Vars 插件将额外的变量数据注入到 Ansible 运行中，这些数据并非来自清单源、playbook 或命令行。像“host_vars”和“group_vars”这样的 playbook 结构使用 vars 插件工作。有关 Ansible 中变量的更多详细信息，请参见 使用变量。

Vars 插件在 Ansible 2.0 中部分实现，并在 Ansible 2.4 开始完全重写。

Ansible 附带的 host_group_vars 插件支持读取 为一台机器分配变量：主机变量 和 为多台机器分配变量：组变量 中的变量。



### 启用 Vars 插件

您可以通过将自定义 vars 插件放入 playbook 旁的 vars_plugins 目录、角色内部，或将其放入 ansible.cfg 中配置的目录源之一来激活它。为了使 vars 插件在清单构建期间运行，您不能在 playbook 或角色中启用它，因为这些直到稍后才会加载。如果它们仅在任务执行时运行，则对其提供位置没有限制。

大多数 vars 插件默认情况下是禁用的。要启用 vars 插件，请在 ansible.cfg 的 defaults 部分设置 vars_plugins_enabled，或将 ANSIBLE_VARS_ENABLED 环境变量设置为要执行的 vars 插件列表。默认情况下，Ansible 附带的 host_group_vars 插件已启用。

从 Ansible 2.10 开始，您可以在集合中使用 vars 插件。集合中的所有 vars 插件都必须显式启用，并且必须使用完全限定的集合名称，格式为 namespace.collection_name.vars_plugin_name。

```
[defaults]
vars_plugins_enabled = host_group_vars,namespace.collection_name.vars_plugin_name
```



### 使用 Vars 插件

默认情况下，启用 vars 插件后，会根据需要自动使用它们。

从 Ansible 2.10 开始，可以使 vars 插件在特定时间运行。ansible-inventory 不使用这些设置，并且始终加载 vars 插件。

全局设置 RUN_VARS_PLUGINS 可以通过在 defaults 部分使用 run_vars_plugins 在 ansible.cfg 中设置，也可以通过 ANSIBLE_RUN_VARS_PLUGINS 环境变量设置。默认选项 demand 会在任务需要变量时运行与清单源相关的任何已启用的 vars 插件。您可以使用选项 start 来代替在导入该清单源后运行与清单源相关的任何已启用的 vars 插件。

您还可以针对支持 stage 选项的 vars 插件，在每个插件的基础上控制 vars 插件的执行。要导入清单后运行 host_group_vars 插件，您可以将以下内容添加到 ansible.cfg

```
[vars_host_group_vars]
stage = inventory
```



### 插件列表

您可以使用 `ansible-doc -t vars -l` 查看可用的 vars 插件列表。使用 `ansible-doc -t vars <plugin name>` 查看特定插件的文档和示例。
