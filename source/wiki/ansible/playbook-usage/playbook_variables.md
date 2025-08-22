---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.18 使用变量
order: 17
---

# 使用变量

Ansible 使用变量来管理不同系统之间的差异。使用 Ansible，您可以使用单个命令在多个不同的系统上执行任务和剧本。为了表示这些不同系统之间的差异，您可以使用标准 YAML 语法创建变量，包括列表和字典。您可以在剧本中、在清单中、在可重用的文件中或角色中或在命令行中定义这些变量。您还可以通过将任务的返回值注册为新变量来在剧本运行期间创建变量。

创建变量后，无论通过在文件中定义、在命令行中传递还是将任务的返回值注册为新变量，您都可以在模块参数中、在条件“when”语句中、在模板中以及在循环中使用这些变量。

了解本页的概念和示例后，请阅读有关Ansible 事实的信息，这些事实是从远程系统检索到的变量。

- 创建有效的变量名
- 简单变量
  - 定义简单变量
  - 引用简单变量
- 何时需要引用变量 (YAML 陷阱)
- 布尔变量
- 列表变量
  - 将变量定义为列表
  - 引用列表变量
- 字典变量
  - 将变量定义为键值对字典
  - 引用键值对字典变量
- 组合变量
  - 组合列表变量
  - 组合字典变量
  - 使用 merge_variables 查找
- 注册变量
- 引用嵌套变量
- 使用 Jinja2 过滤器转换变量
- 变量设置位置
  - 在清单中定义变量
  - 在剧本中定义变量
  - 在包含的文件和角色中定义变量
  - 在运行时定义变量
    - key=value 格式
    - JSON 字符串格式
    - 来自 JSON 或 YAML 文件的变量
- 变量优先级：我应该在哪里放置变量？
  - 了解变量优先级
  - 变量作用域
  - 关于变量设置位置的提示
- 使用高级变量语法



## 创建有效的变量名

并非所有字符串都是有效的 Ansible 变量名。变量名只能包含字母、数字和下划线。Python 关键字或剧本关键字不是有效的变量名。变量名不能以数字开头。

变量名可以以下划线开头。在许多编程语言中，以下划线开头的变量是私有的。Ansible 中并非如此。以下划线开头的变量与任何其他变量的处理方式完全相同。不要依赖此约定来实现隐私或安全性。

此表给出了有效和无效变量名的示例

| 有效的变量名 | 无效的变量名                           |
| ------------ | -------------------------------------- |
| foo          | *foo, Python 关键字，例如async和lambda |
| foo_env      | 剧本关键字，例如environment            |
| foo_port     | foo-port, foo port, foo.port           |
| foo5, _foo   | 5foo, 12                               |

## 简单变量

简单变量将变量名与单个值组合在一起。您可以在各种地方使用此语法（以及下面显示的列表和字典的语法）。有关在清单、剧本、可重用文件、角色或命令行中设置变量的详细信息，请参见变量设置位置。

### 定义简单变量

您可以使用标准 YAML 语法定义简单变量。例如

```
remote_install_path: /opt/my_app_config
```





### 引用简单变量

定义变量后，使用 Jinja2 语法引用它。Jinja2 变量使用双花括号。例如，表达式My amp goes to {{ max_amp_value }}演示了变量替换的最基本形式。您可以在剧本中使用 Jinja2 语法。例如

```
ansible.builtin.template:
  src: foo.cfg.j2
  dest: '{{ remote_install_path }}/foo.cfg'
```



在此示例中，变量定义了文件的位置，该位置在不同系统之间可能有所不同。



## 何时需要引用变量 (YAML 陷阱)

如果以{{ foo }}开头，则必须引用整个表达式才能创建有效的 YAML 语法。如果不引用整个表达式，YAML 解析器将无法解释语法 - 它可能是一个变量，也可能是 YAML 字典的开头。有关编写 YAML 的指南，请参见YAML 语法文档。

如果您像这样不加引号地使用变量

```
- hosts: app_servers
  vars:
    app_path: {{ base_path }}/22
```



您将看到：ERROR! Syntax Error while loading YAML. 如果添加引号，Ansible 将正常工作

```
- hosts: app_servers
  vars:
    app_path: "{{ base_path }}/22"
```





## 布尔变量

Ansible 接受多种布尔变量值：true/false、1/0、yes/no、True/False 等。有效字符串的匹配不区分大小写。虽然文档示例为了与 ansible-lint 的默认设置兼容而重点关注 true/false，但您可以使用以下任何值。

| 有效值                                             | 描述                 |
| -------------------------------------------------- | -------------------- |
| True、'true'、't'、'yes'、'y'、'on'、'1'、1、1.0   | 真值 (Truthy values) |
| False、'false'、'f'、'no'、'n'、'off'、'0'、0、0.0 | 假值 (Falsy values)  |



## 列表变量

列表变量将变量名与多个值组合在一起。多个值可以存储为项目列表或方括号 [] 中，用逗号分隔。

### 将变量定义为列表

您可以使用 YAML 列表定义具有多个值的变量。例如

```
region:
  - northeast
  - southeast
  - midwest
```



### 引用列表变量

当您使用定义为列表（也称为数组）的变量时，您可以使用该列表中的单个特定字段。列表中的第一个项目是项目 0，第二个项目是项目 1。例如

```
region: "{{ region[0] }}"
```



此表达式的值为“northeast”。



## 字典变量

字典以键值对的形式存储数据。通常，字典用于存储相关数据，例如 ID 或用户配置文件中包含的信息。

### 将变量定义为键值字典

您可以使用 YAML 字典定义更复杂的变量。YAML 字典将键映射到值。例如

```
foo:
  field1: one
  field2: two
```



### 引用键值字典变量

当您使用定义为键值字典（也称为哈希）的变量时，您可以使用方括号表示法或点表示法使用该字典中的单个特定字段。

```
foo['field1']
foo.field1
```



这两个示例都引用相同的值（“one”）。方括号表示法始终有效。点表示法可能会导致问题，因为某些键与 Python 字典的属性和方法冲突。如果您使用的键以两个下划线开头和结尾（在 Python 中保留用于特殊含义）或任何已知的公共属性，请使用方括号表示法。

add、append、as_integer_ratio、bit_length、capitalize、center、clear、conjugate、copy、count、decode、denominator、difference、difference_update、discard、encode、endswith、expandtabs、extend、find、format、fromhex、fromkeys、get、has_key、hex、imag、index、insert、intersection、intersection_update、isalnum、isalpha、isdecimal、isdigit、isdisjoint、is_integer、islower、isnumeric、isspace、issubset、issuperset、istitle、isupper、items、iteritems、iterkeys、itervalues、join、keys、ljust、lower、lstrip、numerator、partition、pop、popitem、real、remove、replace、reverse、rfind、rindex、rjust、rpartition、rsplit、rstrip、setdefault、sort、split、splitlines、startswith、strip、swapcase、symmetric_difference、symmetric_difference_update、title、translate、union、update、upper、values、viewitems、viewkeys、viewvalues、zfill。

## 组合变量

要合并包含列表或字典的变量，您可以使用以下方法。

### 组合列表变量

您可以使用 set_fact 模块将列表组合到新的 merged_list 变量中，如下所示

```
vars:
  list1:
  - apple
  - banana
  - fig

  list2:
  - peach
  - plum
  - pear

tasks:
- name: Combine list1 and list2 into a merged_list var
  ansible.builtin.set_fact:
    merged_list: "{{ list1 + list2 }}"
```



### 组合字典变量

要合并字典，请使用 combine 过滤器，例如

```
vars:
  dict1:
    name: Leeroy Jenkins
    age: 25
    occupation: Astronaut

  dict2:
    location: Galway
    country: Ireland
    postcode: H71 1234

tasks:
- name: Combine dict1 and dict2 into a merged_dict var
  ansible.builtin.set_fact:
    merged_dict: "{{ dict1 | ansible.builtin.combine(dict2) }}"
```



更多详情，请参阅 ansible.builtin.combine。

### 使用 merge_variables lookup

要合并与给定前缀、后缀或正则表达式匹配的变量，可以使用 community.general.merge_variables lookup，例如

```
merged_variable: "{{ lookup('community.general.merge_variables', '__my_pattern', pattern_type='suffix') }}"
```



更多详情和使用示例，请参考 community.general.merge_variables lookup 文档。



## 注册变量

您可以使用任务关键字 register 从 Ansible 任务的输出创建变量。您可以在 playbook 中的任何后续任务中使用已注册的变量。例如

```
- hosts: web_servers

  tasks:

     - name: Run a shell command and register its output as a variable
       ansible.builtin.shell: /usr/bin/foo
       register: foo_result
       ignore_errors: true

     - name: Run a shell command using output of the previous task
       ansible.builtin.shell: /usr/bin/bar
       when: foo_result.rc == 5
```



有关在后续任务的条件中使用已注册变量的更多示例，请参阅 条件语句。已注册的变量可以是简单变量、列表变量、字典变量或复杂的嵌套数据结构。每个模块的文档都包含一个 RETURN 部分，描述该模块的返回值。要查看特定任务的值，请使用 -v 运行您的 playbook。

已注册的变量存储在内存中。您无法缓存已注册的变量以供将来 playbook 运行使用。已注册的变量仅在当前 playbook 运行的其余时间内对主机有效，包括同一 playbook 运行中的后续任务。

已注册的变量是主机级变量。当您在包含循环的任务中注册变量时，已注册的变量将包含循环中每个项目的数值。在循环期间放入变量中的数据结构将包含一个 results 属性，该属性是模块所有响应的列表。有关此工作原理的更深入示例，请参阅 循环 部分中关于将 register 与循环一起使用的内容。



## 引用嵌套变量

许多注册变量（和facts）是嵌套的 YAML 或 JSON 数据结构。你不能使用简单的{{ foo }}语法访问这些嵌套数据结构中的值。你必须使用方括号表示法或点表示法。例如，要使用方括号表示法从你的 facts 中引用 IP 地址

```
{{ ansible_facts["eth0"]["ipv4"]["address"] }}
```



要使用点表示法从你的 facts 中引用 IP 地址

```
{{ ansible_facts.eth0.ipv4.address }}
```





## 使用 Jinja2 过滤器转换变量

Jinja2 过滤器允许你在模板表达式中转换变量的值。例如，capitalize 过滤器将传递给它的任何值都大写；to_yaml 和 to_json 过滤器会更改变量值的格式。Jinja2 包含许多内置过滤器，Ansible 还提供了更多过滤器。要查找更多过滤器的示例，请参见使用过滤器操作数据。



## 在何处设置变量

你可以在各种地方定义变量，例如在清单、剧本、可重用文件、角色以及命令行中。Ansible 加载它找到的每个可能的变量，然后根据变量优先级规则选择要应用的变量。



### 在清单中定义变量

你可以为每个主机单独定义不同的变量，或者为清单中的主机组设置共享变量。例如，如果[Boston]组中的所有机器都使用“boston.ntp.example.com”作为 NTP 服务器，则可以设置组变量。如何构建你的清单页面详细介绍了如何在清单中设置主机变量和组变量。



### 在一个剧本中定义变量

你可以在剧本中直接定义变量。

```
- hosts: webservers
  vars:
    http_port: 80
```



在剧本中定义变量时，这些变量仅对在该剧本中执行的任务可见。



### 在包含的文件和角色中定义变量

你可以在可重用的变量文件中和/或可重用的角色中定义变量。在可重用的变量文件中定义变量时，敏感变量与剧本分开。这种分离使你能够将你的剧本存储在源代码控制软件中，甚至可以共享剧本，而不会冒暴露密码或其他敏感和个人数据的风险。有关创建可重用文件和角色的信息，请参见重用 Ansible 工件。

此示例显示如何包含在外部文件中定义的变量。

```
---

- hosts: all
  remote_user: root
  vars:
    favcolor: blue
  vars_files:
    - /vars/external_vars.yml

  tasks:

  - name: This is just a placeholder
    ansible.builtin.command: /bin/echo foo
```



每个变量文件的内容都是一个简单的 YAML 字典。例如

```
---
# in the above example, this would be vars/external_vars.yml
somevar: somevalue
password: magic
```





### 在运行时定义变量

你可以在运行剧本时定义变量，方法是使用--extra-vars（或-e）参数在命令行中传递变量。你还可以使用vars_prompt请求用户输入（参见交互式输入：提示）。在命令行中传递变量时，请使用包含一个或多个变量的单引号字符串，格式如下所示。

#### key=value 格式

使用key=value语法传递的值被解释为字符串。如果需要传递非字符串值（例如布尔值、整数、浮点数、列表等），请使用 JSON 格式。

```
ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
```



#### JSON 字符串格式

```
ansible-playbook release.yml --extra-vars '{"version":"1.23.45","other_variable":"foo"}'
ansible-playbook arcade.yml --extra-vars '{"pacman":"mrs","ghosts":["inky","pinky","clyde","sue"]}'
```



使用--extra-vars传递变量时，必须适当地转义引号和其他特殊字符，以适应你的标记（例如 JSON）和 shell。

```
ansible-playbook arcade.yml --extra-vars "{\"name\":\"Conan O\'Brien\"}"
ansible-playbook arcade.yml --extra-vars '{"name":"Conan O'\\\''Brien"}'
ansible-playbook script.yml --extra-vars "{\"dialog\":\"He said \\\"I just can\'t get enough of those single and double-quotes"\!"\\\"\"}"
```



#### 来自 JSON 或 YAML 文件的变量

如果你有很多特殊字符，请使用包含变量定义的 JSON 或 YAML 文件。在 JSON 和 YAML 文件名前都加上@。

```
ansible-playbook release.yml --extra-vars "@some_file.json"
ansible-playbook release.yml --extra-vars "@some_file.yaml"
```





## 变量优先级：我应该在哪里放置变量？

你可以在许多不同的地方设置多个同名变量。当你这样做时，Ansible 会加载它找到的每个可能的变量，然后根据变量优先级选择要应用的变量。换句话说，不同的变量将按照一定的顺序相互覆盖。

就定义变量（在何处定义特定类型的变量）达成指导原则的团队和项目通常可以避免变量优先级问题。我们建议你将每个变量定义在一个地方：弄清楚在哪里定义变量，并保持简单。有关示例，请参见关于在何处设置变量的技巧。

你可以在变量中设置的一些行为参数，也可以在 Ansible 配置中、作为命令行选项以及使用剧本关键字来设置。例如，你可以使用ansible_user在变量中定义 Ansible 用于连接到远程设备的用户，在配置文件中使用DEFAULT_REMOTE_USER，在命令行中使用-u，以及使用剧本关键字remote_user。如果在变量中和通过其他方法定义相同的参数，则变量会覆盖其他设置。这种方法允许特定于主机的设置覆盖更一般的设置。有关这些各种设置的优先级的示例和更多详细信息，请参见控制 Ansible 的行为方式：优先级规则。

### 理解变量优先级

Ansible确实会应用变量优先级，你可能会有用武之地。以下是优先级顺序，从低到高（最后列出的变量会覆盖所有其他变量）：

> 1. 命令行值（例如-u my_user，这些不是变量）
> 2. 角色默认值（如角色目录结构中所定义）1
> 3. 清单文件或脚本组变量2
> 4. 清单 group_vars/all 3
> 5. 剧本 group_vars/all 3
> 6. 清单 group_vars/* 3
> 7. 剧本 group_vars/* 3
> 8. 清单文件或脚本主机变量2
> 9. 清单 host_vars/* 3
> 10. 剧本 host_vars/* 3
> 11. 主机 facts / 缓存的 set_facts 4
> 12. 剧本变量
> 13. 剧本 vars_prompt
> 14. 剧本 vars_files
> 15. 角色变量（如角色目录结构中所定义）
> 16. 块变量（仅限于块中的任务）
> 17. 任务变量（仅限于任务）
> 18. include_vars
> 19. set_facts / 注册变量
> 20. 角色（和 include_role）参数
> 21. include 参数
> 22. 额外变量（例如-e "user=my_user"）（始终具有最高优先级）

一般来说，Ansible 优先考虑最近定义、更主动定义以及具有更明确作用域的变量。角色内部 defaults 文件夹中的变量很容易被覆盖。角色的 vars 目录中的任何内容都会覆盖名称空间中该变量的先前版本。主机和/或清单变量会覆盖角色默认值，但显式包含（例如 vars 目录或include_vars任务）会覆盖清单变量。

Ansible 合并清单中设置的不同变量，以便更具体的设置会覆盖更通用的设置。例如，指定为 group_var 的ansible_ssh_user会被指定为 host_var 的ansible_user覆盖。有关清单中设置的变量优先级的详细信息，请参见变量如何合并。

脚注

[1]

每个角色中的任务都会看到它们自己角色的默认值。在角色之外定义的任务会看到最后一个角色的默认值。

[2](1,2)

在清单文件或动态清单中定义的变量。

[3](1,2,3,4,5,6)

包含由“变量插件”添加的变量，以及由 Ansible 附带的默认变量插件添加的主机变量和组变量。

[4]

使用 set_facts 的可缓存选项创建时，变量在剧本中具有高优先级，但当它们来自缓存时，与主机 facts 的优先级相同。



### 变量作用域

你可以根据所需的值的作用域来决定在哪里设置变量。Ansible 有三个主要作用域：

> - 全局：由配置、环境变量和命令行设置
> - 剧本：每个剧本及其包含的结构、vars 条目（vars；vars_files；vars_prompt）、角色默认值和 vars。
> - 主机：直接与主机关联的变量，例如清单、include_vars、facts 或注册的任务输出

在模板中，您可以自动访问主机范围内所有变量，以及任何已注册的变量、facts 和魔术变量。



### 变量设置位置提示

您应该根据您可能希望对值进行的控制类型来选择定义变量的位置。

在清单中设置与地理位置或行为相关的变量。由于组通常是将角色映射到主机的实体，因此您通常可以在组上设置变量，而不是在角色上定义它们。记住：子组会覆盖父组，主机变量会覆盖组变量。有关设置主机和组变量的详细信息，请参阅在清单中定义变量。

在group_vars/all文件中设置公共默认值。有关如何在清单中组织主机和组变量的详细信息，请参阅组织主机和组变量。组变量通常放置在清单文件旁边，但它们也可以由动态清单返回（请参阅使用动态清单）或在 AWX 或Red Hat Ansible Automation Platform中通过 UI 或 API 定义。

```
---
# file: /etc/ansible/group_vars/all
# this is the site wide default
ntp_server: default-time.example.com
```



在group_vars/my_location文件中设置特定位置的变量。所有组都是all组的子组，因此此处设置的变量会覆盖在group_vars/all中设置的变量。

```
---
# file: /etc/ansible/group_vars/boston
ntp_server: boston-time.example.com
```



如果一台主机使用不同的 NTP 服务器，则可以在 host_vars 文件中设置该服务器，这将覆盖组变量。

```
---
# file: /etc/ansible/host_vars/xyz.boston.example.com
ntp_server: override.example.com
```



在角色中设置默认值以避免未定义变量错误。如果您共享您的角色，其他用户可以依赖您在roles/x/defaults/main.yml文件中添加的合理默认值，或者他们可以轻松地在清单或命令行中覆盖这些值。有关更多信息，请参阅角色。例如

```
---
# file: roles/x/defaults/main.yml
# if no other value is supplied in inventory or as a parameter, this value will be used
http_port: 80
```



在角色中设置变量以确保在该角色中使用某个值，并且不会被清单变量覆盖。如果您不与其他人共享您的角色，则可以以此方式定义特定于应用程序的行为（如端口），方法是在roles/x/vars/main.yml中定义。如果您与其他人共享角色，则在此处放置变量会使它们更难以覆盖，尽管仍然可以通过向角色传递参数或使用-e设置变量来覆盖。

```
---
# file: roles/x/vars/main.yml
# this will absolutely be used in this role
http_port: 80
```



调用角色时将变量作为参数传递，以实现最大的清晰度、灵活性和可见性。此方法会覆盖角色中存在的任何默认值。例如

```
roles:
   - role: apache
     vars:
        http_port: 8080
```



当您阅读此剧本时，很明显您已选择设置变量或覆盖默认值。您还可以传递多个值，这允许您多次运行同一个角色。有关更多详细信息，请参阅在一个剧本中多次运行角色。例如

```
roles:
   - role: app_user
     vars:
        myname: Ian
   - role: app_user
     vars:
       myname: Terry
   - role: app_user
     vars:
       myname: Graham
   - role: app_user
     vars:
       myname: John
```



在一个角色中设置的变量可用于后面的角色。您可以在角色的vars目录（如角色目录结构中所定义）中设置变量，并在您的剧本的其他角色和其他地方使用它们。

```
roles:
   - role: common_settings
   - role: something
     vars:
       foo: 12
   - role: something_else
```



我们建议您不要担心变量优先级，而是在决定变量设置位置时考虑您希望多容易或多频繁地覆盖变量。如果您不确定定义了哪些其他变量，并且您需要特定值，请使用--extra-vars（-e）覆盖所有其他变量。

## 使用高级变量语法

有关用于声明变量并更好地控制 Ansible 使用的 YAML 文件中放置的数据的高级 YAML 语法的详细信息，请参阅[高级剧本语法](https://docs.ansible.org.cn/ansible/latest/playbook_guide/playbooks_advanced_syntax.html#playbooks-advanced-syntax)。
