---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 3.1 构建清单
order: 0
---

# 构建清单

Ansible 使用称为清单的列表或列表组来自动化您基础设施中受管节点或“主机”上的任务。您可以在命令行上传递主机名，但大多数 Ansible 用户会创建清单文件。您的清单定义了您自动化的受管节点，并带有组，因此您可以同时在多个主机上运行自动化任务。定义清单后，您可以使用模式来选择您希望 Ansible 运行的主机或组。

最简单的清单是一个包含主机和组列表的单个文件。此文件的默认位置是 /etc/ansible/hosts。您可以使用 -i <path> 选项在命令行上或使用 inventory 在配置中指定不同的清单文件。

Ansible 清单插件支持一系列格式和来源，使您的清单灵活且可自定义。随着清单的扩展，您可能需要多个文件来组织主机和组。以下是除 /etc/ansible/hosts 文件之外的三个选项

- 您可以创建一个包含多个清单文件的目录。请参阅在目录中组织清单。这些可以使用不同的格式（YAML、ini 等）。
- 您可以动态拉取清单。例如，您可以使用动态清单插件来列出一个或多个云提供商中的资源。请参阅使用动态清单。
- 您可以将多个源用于清单，包括动态清单和静态文件。请参阅传递多个清单源。



## 清单基础：格式、主机和组

您可以根据您拥有的清单插件，以多种格式之一创建清单文件。最常见的格式是 INI 和 YAML。基本的 INI /etc/ansible/hosts 可能如下所示

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```



方括号中的标题是组名称，用于对主机进行分类，并决定您在什么时间以及出于什么目的控制哪些主机。组名称应遵循与创建有效的变量名称相同的准则。

这是 YAML 格式的相同基本清单文件

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
```



### 默认组

即使您未在清单文件中定义任何组，Ansible 也会创建两个默认组：all 和 ungrouped。all 组包含每个主机。ungrouped 组包含除 all 之外没有其他组的所有主机。每个主机将始终属于至少 2 个组（all 和 ungrouped 或 all 和其他一些组）。例如，在上面的基本清单中，主机 mail.example.com 属于 all 组和 ungrouped 组；主机 two.example.com 属于 all 组和 dbservers 组。尽管 all 和 ungrouped 始终存在，但它们可以是隐式的，并且不会出现在 group_names 之类的组列表中。



### 多个组中的主机

您可以将每个主机放在多个组中。例如，亚特兰大数据中心中的一台生产 Web 服务器可能会包含在名为 [prod]、[atlanta] 和 [webservers] 的组中。您可以创建跟踪以下内容的组

- 什么 - 应用程序、堆栈或微服务（例如，数据库服务器、Web 服务器等）。
- 哪里 - 数据中心或区域，以与本地 DNS、存储等通信（例如，东、西）。
- 何时 - 开发阶段，以避免在生产资源上进行测试（例如，prod、test）。

扩展之前的 YAML 清单以包含什么、何时和何地，如下所示

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
test:
  hosts:
    bar.example.com:
    three.example.com:
```



您可以看到 one.example.com 存在于 dbservers、east 和 prod 组中。



### 分组组：父/子组关系

您可以在组之间创建父/子关系。父组也称为嵌套组或组的组。例如，如果您的所有生产主机已经位于诸如 atlanta_prod 和 denver_prod 之类的组中，则可以创建一个包含这些较小组的 production 组。这种方法减少了维护工作，因为您可以通过编辑子组来向父组添加或删除主机。

要为组创建父/子关系

- 在 INI 格式中，使用 :children 后缀
- 在 YAML 格式中，使用 children: 条目

这里展示了与上面相同的清单，但为了 prod 和 test 组进行了简化，使用了父组。这两个清单文件会产生相同的结果。

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  children:
    east:
test:
  children:
    west:
```



子组有一些需要注意的属性

- 任何属于子组的 host 都会自动成为父组的成员。
- 组可以有多个父组和子组，但不能有循环关系。
- Host 也可以属于多个组，但在运行时，一个 host 只有一个实例。Ansible 会合并来自多个组的数据。



### 添加主机范围

如果你有很多具有相似模式的 host，你可以添加一个范围而不是单独列出每个主机名。

在 INI 中

```
[webservers]
www[01:50].example.com
```



在 YAML 中

```
# ...
  webservers:
    hosts:
      www[01:50].example.com:
```



在定义数字范围的 host 时，你可以指定一个步长（序列号之间的增量）。

在 INI 中

```
[webservers]
www[01:50:2].example.com
```



在 YAML 中

```
# ...
  webservers:
    hosts:
      www[01:50:2].example.com:
```



上面的示例将匹配子域名 www01、www03、www05、...、www49，但不匹配 www00、www02、www50 等，因为步长（增量）是每个步骤 2 个单位。

对于数字模式，可以根据需要包含或删除前导零。范围是包含的。你还可以定义字母范围

```
[databases]
db-[a:f].example.com
```



## 传递多个清单源

你可以通过命令行传递多个清单参数，或者配置 ANSIBLE_INVENTORY，来同时定位多个清单源（目录、动态清单脚本或清单插件支持的文件）。当你希望同时针对通常分离的环境（如 staging 和 production）执行特定操作时，这会很有用。

从命令行定位两个清单源：

```
ansible-playbook get_logs.yml -i staging -i production
```



## 在目录中组织清单

你可以将多个清单源整合到一个目录中。最简单的形式是使用包含多个文件的目录，而不是单个清单文件。当单个文件变得太长时，维护起来会变得困难。如果你有多个团队和多个自动化项目，为每个团队或项目创建一个清单文件可以让每个人轻松找到对他们重要的 host 和组。

你还可以在清单目录中组合多个清单源类型。这对于组合静态和动态 host 并将它们作为一个清单进行管理很有用。以下清单目录组合了一个清单插件源、一个动态清单脚本和一个包含静态 host 的文件

```
inventory/
  openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  dynamic-inventory.py   # add additional hosts with dynamic inventory script
  on-prem                # add static hosts and groups
  parent-groups          # add static hosts and groups
```



你可以按如下方式定位此清单目录

```
ansible-playbook example.yml -i inventory
```



你还可以在你的 ansible.cfg 文件中配置清单目录。有关更多详细信息，请参阅 配置 Ansible。

### 管理清单加载顺序

Ansible 会按照文件名以 ASCII 顺序加载清单源。如果你在一个文件或目录中定义了父组，在其他文件或目录中定义了子组，则必须先加载定义子组的文件。如果先加载父组，你将看到错误 Unable to parse /path/to/source_of_parent_groups as an inventory source。

例如，如果你有一个名为 groups-of-groups 的文件定义了一个 production 组，其中子组定义在名为 on-prem 的文件中，则 Ansible 无法解析 production 组。要避免此问题，你可以通过向文件添加前缀来控制加载顺序

```
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-on-prem                # add static hosts and groups
  04-groups-of-groups       # add parent groups
```



你可以在 清单设置示例中找到如何组织你的清单和分组你的 host 的示例。



## 向清单添加变量

你可以在清单中存储与特定 host 或组相关的变量值。首先，你可以直接将变量添加到你的主清单文件中的 host 和组中。

为了简单起见，我们记录了在主清单文件中添加变量的方法。但是，将变量存储在单独的 host 和组变量文件中是一种更健壮的方法来描述你的系统策略。在主清单文件中设置变量只是一种简写方式。有关在 ‘host_vars’ 目录中的单独文件中存储变量值的指南，请参阅 组织 host 和组变量。有关详细信息，请参阅 组织 host 和组变量。



## 为一个机器分配变量：host 变量

你可以轻松地为单个 host 分配一个变量，然后在 playbook 中使用它。你可以在你的清单文件中直接执行此操作。

在 INI 中

```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```



在 YAML 中

```
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
```



非标准的 SSH 端口等唯一值可以很好地用作 host 变量。你可以通过在主机名后添加端口号和冒号将它们添加到你的 Ansible 清单中

```
badwolf.example.com:5309
```



连接变量也可以很好地用作 host 变量

```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```





### 清单别名

你还可以使用 host 变量在清单中定义别名

在 INI 中

```
jumper ansible_port=5555 ansible_host=192.0.2.50
```



在 YAML 中

```
# ...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```



在此示例中，针对主机别名“jumper”运行 Ansible 将连接到端口 5555 上的 192.0.2.50。请参阅 行为清单参数，以进一步自定义与主机的连接。

## 以 INI 格式定义变量

使用 key=value 语法以 INI 格式传递的值，根据它们声明的位置，会有不同的解释

- 当与 host 内联声明时，INI 值被解释为 Python 字面结构（字符串、数字、元组、列表、字典、布尔值、None）。Host 行每行接受多个 key=value 参数。因此，它们需要一种方法来指示空格是值的一部分，而不是分隔符。包含空格的值可以用引号（单引号或双引号）引起来。有关详细信息，请参阅 Python shlex 解析规则。
- 当在 :vars 部分中声明时，INI 值被解释为字符串。例如，var=FALSE 将创建一个等于 ‘FALSE’ 的字符串。与 host 行不同，:vars 部分每行只接受一个条目，因此 = 之后的所有内容必须是该条目的值。

如果 INI 清单中设置的变量值必须是某种类型（例如，字符串或布尔值），请始终在你的任务中使用过滤器指定类型。当使用变量时，不要依赖 INI 清单中设置的类型。

考虑使用 YAML 格式进行清单源以避免对变量的实际类型产生混淆。YAML 清单插件会一致且正确地处理变量值。



## 为多个机器分配变量：组变量

如果组中的所有 host 共享一个变量值，则你可以一次将该变量应用于整个组。

在 INI 中

```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```



在 YAML 中

```
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```



组变量是一种一次将变量应用于多个 host 的便捷方法。但是，在执行之前，Ansible 总是将变量（包括清单变量）展平到 host 级别。如果一个 host 是多个组的成员，则 Ansible 会从所有这些组中读取变量值。如果在不同的组中为同一变量分配了不同的值，则 Ansible 会根据内部的 合并规则选择要使用的值。



### 继承变量值：用于组的组的组变量

你可以将变量应用于父组（嵌套组或组的组）以及子组。语法相同：INI 格式为 :vars，YAML 格式为 vars:

在 INI 中

```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```



在 YAML 中

```
usa:
  children:
    southeast:
      children:
        atlanta:
          hosts:
            host1:
            host2:
        raleigh:
          hosts:
            host2:
            host3:
      vars:
        some_server: foo.southeast.example.com
        halon_system_timeout: 30
        self_destruct_countdown: 60
        escape_pods: 2
    northeast:
    northwest:
    southwest:
```



子组的变量将具有更高的优先级（覆盖）父组的变量。



## 组织 host 和组变量

虽然你可以将变量存储在主清单文件中，但存储单独的 host 和组变量文件可能有助于你更轻松地组织变量值。你还可以在 host 和组变量文件中使用列表和哈希数据，这是你在主清单文件中无法做到的。

Host 和组变量文件必须使用 YAML 语法。有效的文件扩展名包括 ‘.yml’、‘.yaml’、‘.json’ 或没有文件扩展名。如果你不熟悉 YAML，请参阅 YAML 语法。

Ansible 通过搜索相对于清单文件或 playbook 文件的路径来加载主机和组变量文件。如果您的清单文件位于 /etc/ansible/hosts，其中包含一个名为 ‘foosball’ 的主机，该主机属于 ‘raleigh’ 和 ‘webservers’ 两个组，那么该主机将使用以下位置的 YAML 文件中的变量

```
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```



例如，如果您按照数据中心对清单中的主机进行分组，并且每个数据中心都使用自己的 NTP 服务器和数据库服务器，则可以创建一个名为 /etc/ansible/group_vars/raleigh 的文件来存储 raleigh 组的变量

```
---
ntp_server: acme.example.org
database_server: storage.example.org
```



您还可以创建以组或主机命名的目录。Ansible 将按字典顺序读取这些目录中的所有文件。以下是 ‘raleigh’ 组的示例

```
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```



‘raleigh’ 组中的所有主机都将可以使用这些文件中定义的变量。当单个文件变得太大，或者您想在某些组变量上使用 Ansible Vault 时，这非常有用，可以保持变量的组织性。

对于 ansible-playbook，您还可以将 group_vars/ 和 host_vars/ 目录添加到您的 playbook 目录中。其他 Ansible 命令（例如，ansible、ansible-console 等）只会查找清单目录中的 group_vars/ 和 host_vars/。如果您希望其他命令从 playbook 目录加载组和主机变量，则必须在命令行上提供 --playbook-dir 选项。如果您从 playbook 目录和清单目录加载清单文件，则 playbook 目录中的变量将覆盖清单目录中设置的变量。

将您的清单文件和变量保存在 Git 仓库（或其他版本控制）中是跟踪您的清单和主机变量更改的绝佳方法。



## 变量如何合并

默认情况下，变量在运行 playbook 之前会合并/展平到特定主机。这使 Ansible 专注于主机和任务，因此组不会在清单和主机匹配之外存在。默认情况下，Ansible 会覆盖变量，包括为组和/或主机定义的变量（请参阅 DEFAULT_HASH_BEHAVIOUR）。顺序/优先级为（从最低到最高）

- 所有组（因为它是所有其他组的“父级”）
- 父组
- 子组
- 主机

默认情况下，Ansible 按 ASCII 顺序合并同一父/子级别的组，并且最后一个加载的组的变量将覆盖之前组的变量。例如，a_group 将与 b_group 合并，并且匹配的 b_group 变量将覆盖 a_group 中的变量。

您可以通过设置组变量 ansible_group_priority 来更改同一级别组的合并顺序（在父/子顺序解析后）。数字越大，合并的时间越晚，优先级越高。如果未设置，则此变量默认为 1。例如

```
a_group:
  vars:
    testvar: a
    ansible_group_priority: 10
b_group:
  vars:
    testvar: b
```



在此示例中，如果两个组具有相同的优先级，则结果通常为 testvar == b，但由于我们为 a_group 赋予了更高的优先级，因此结果将为 testvar == a。



### 管理清单变量加载顺序

当使用多个清单源时，请记住，任何变量冲突都会根据 变量如何合并 和 变量优先级：我应该将变量放在哪里？ 中描述的规则进行解决。您可以控制清单源中变量的合并顺序，以获取您需要的变量值。

当您在命令行上传递多个清单源时，Ansible 会按照您传递这些参数的顺序合并变量。如果在暂存清单中的 [all:vars] 定义了 myvar = 1，而生产清单定义了 myvar = 2，那么

- 传递 -i staging -i production 以使用 myvar = 2 运行 playbook。
- 传递 -i production -i staging 以使用 myvar = 1 运行 playbook。

当您将多个清单源放在一个目录中时，Ansible 会按照文件名的 ASCII 顺序合并它们。您可以通过向文件添加前缀来控制加载顺序

```
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-static-inventory       # add static hosts
  group_vars/
    all.yml                 # assign variables to all hosts
```



如果 01-openstack.yml 为组 all 定义了 myvar = 1，02-dynamic-inventory.py 定义了 myvar = 2，而 03-static-inventory 定义了 myvar = 3，则 playbook 将使用 myvar = 3 运行。

有关清单插件和动态清单脚本的更多详细信息，请参阅 清单插件 和 使用动态清单。



## 连接到主机：行为清单参数

如上所述，设置以下变量将控制 Ansible 如何与远程主机交互。

主机连接

- ansible_connection

  与主机的连接类型。这可以是任何 Ansible 连接插件的名称。SSH 协议类型为 ssh 或 paramiko。默认值为 ssh。

所有连接的通用设置

- ansible_host

  要连接的主机名称，如果与您要为其提供的别名不同。如果您使用委派，请勿将其设置为依赖 inventory_hostname。

- ansible_port

  连接端口号，如果不是默认端口号（ssh 为 22）

- ansible_user

  连接到主机时使用的用户名

- ansible_password

  用于对主机进行身份验证的密码（永远不要以纯文本形式存储此变量；始终使用 vault。请参阅 安全可见的 Vaulted 变量）

SSH 连接的特定设置

- ansible_ssh_private_key_file

  SSH 使用的私钥文件。如果使用多个密钥并且您不想使用 SSH 代理，则非常有用。

- ansible_ssh_common_args

  此设置始终附加到 sftp、scp 和 ssh 的默认命令行。用于为特定主机（或组）配置 ProxyCommand。

- ansible_sftp_extra_args

  此设置始终附加到默认的 sftp 命令行。

- ansible_scp_extra_args

  此设置始终附加到默认的 scp 命令行。

- ansible_ssh_extra_args

  此设置始终附加到默认的 ssh 命令行。

- ansible_ssh_pipelining

  确定是否使用 SSH 管道。这会覆盖 ansible.cfg 中的 pipelining 设置。

- ansible_ssh_executable (版本 2.2 中新增)

  此设置会覆盖使用系统 ssh 的默认行为。这会覆盖 ansible.cfg 中 ssh_connection 下的 ssh_executable 设置。

权限提升（有关详细信息，请参阅 Ansible 权限提升）

- ansible_become

  等同于 ansible_sudo 或 ansible_su，允许强制执行权限提升。

- ansible_become_method

  允许设置权限提升方法。

- ansible_become_user

  等同于 ansible_sudo_user 或 ansible_su_user，允许您设置通过权限提升成为的用户。

- ansible_become_password

  等同于 ansible_sudo_password 或 ansible_su_password，允许您设置权限提升密码（永远不要以纯文本形式存储此变量；始终使用 Vault。请参阅 安全地保持 Vault 变量可见）

- ansible_become_exe

  等同于 ansible_sudo_exe 或 ansible_su_exe，允许您为所选的提升方法设置可执行文件。

- ansible_become_flags

  等同于 ansible_sudo_flags 或 ansible_su_flags，允许您设置传递给所选提升方法的标志。也可以在 ansible.cfg 中 privilege_escalation 下的 become_flags 选项中全局设置此项。

远程主机环境参数

- ansible_shell_type

  目标系统的 shell 类型。除非您已将 ansible_shell_executable 设置为非 Bourne (sh) 兼容的 shell，否则不应使用此设置。默认情况下，命令使用 sh 样式的语法格式化。将其设置为 csh 或 fish 将导致在目标系统上执行的命令遵循这些 shell 的语法。

- ansible_python_interpreter

  目标主机 Python 路径。这对于具有多个 Python 或 Python 不位于 /usr/bin/python 的系统（如 *BSD）或者 /usr/bin/python 不是 2.X 系列 Python 的系统非常有用。我们不使用 /usr/bin/env 机制，因为这需要正确设置远程用户的路径，并且还假设 python 可执行文件名为 python，而该可执行文件可能被命名为 python2.6 之类的名称。

- ansible*interpreter

  适用于 ruby 或 perl 之类的任何内容，其工作方式与 ansible_python_interpreter 相同。这将替换将在该主机上运行的模块的 shebang。

2.1 版本中新增。

- ansible_shell_executable

  此设置 Ansible 控制节点将在目标计算机上使用的 shell，覆盖 ansible.cfg 中的 executable，该默认值为 /bin/sh。仅当无法使用 /bin/sh 时才应更改此值（换句话说，如果目标计算机上未安装 /bin/sh 或无法从 sudo 运行）。

来自 Ansible-INI 主机文件的示例

```
some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
```



### 非 SSH 连接类型

如上一节所述，Ansible 通过 SSH 执行 playbook，但不仅限于此连接类型。通过特定于主机的参数 ansible_connection=<connector>，可以更改连接类型。有关可用插件和示例的完整列表，请参阅 插件列表。



## 清单设置示例

另请参阅 Ansible 设置示例，其中显示了清单以及 playbook 和其他 Ansible 工件。



### 示例：每个环境一个清单

如果需要管理多个环境，有时最好每个清单只定义一个环境的主机。这样，例如，当您想要更新一些“staging”服务器时，就更难意外地更改“test”环境内的节点状态。

对于上述示例，您可以拥有一个 inventory_test 文件

```
[dbservers]
db01.test.example.com
db02.test.example.com

[appservers]
app01.test.example.com
app02.test.example.com
app03.test.example.com
```



该文件仅包含属于“test”环境的主机。在另一个名为 inventory_staging 的文件中定义“staging”计算机

```
[dbservers]
db01.staging.example.com
db02.staging.example.com

[appservers]
app01.staging.example.com
app02.staging.example.com
app03.staging.example.com
```



要将名为 site.yml 的 playbook 应用于测试环境中的所有应用服务器，请使用以下命令

```
ansible-playbook -i inventory_test -l appservers site.yml
```



### 示例：按功能分组

在上一节中，您已经看到了使用组来聚集具有相同功能的主机的示例。例如，这允许您在仅影响数据库服务器的 playbook 或角色中定义防火墙规则

```
- hosts: dbservers
  tasks:
  - name: Allow access from 10.0.0.1
    ansible.builtin.iptables:
      chain: INPUT
      jump: ACCEPT
      source: 10.0.0.1
```



### 示例：按位置分组

其他任务可能侧重于特定主机的位置。假设 db01.test.example.com 和 app01.test.example.com 位于 DC1 中，而 db02.test.example.com 位于 DC2 中

```
[dc1]
db01.test.example.com
app01.test.example.com

[dc2]
db02.test.example.com
```



实际上，您甚至可能最终混合使用所有这些设置，因为您可能需要在一天更新特定数据中心中的所有节点，而在另一天更新所有应用程序服务器，而无论其位置如何。
