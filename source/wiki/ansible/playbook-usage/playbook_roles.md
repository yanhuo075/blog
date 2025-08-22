---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.15 剧本角色
order: 14
---

# 角色

角色允许您根据已知的目录结构自动加载相关的变量、文件、任务、处理器和其他 Ansible 工件。将内容分组到角色后，您可以轻松地重用它们并与其他用户共享。

- 角色目录结构
- 存储和查找角色
- 使用角色
  - 在剧本级别使用角色
  - 包含角色：动态重用
  - 导入角色：静态重用
- 角色参数验证
  - 规范格式
  - 示例规范
- 在一个剧本中多次运行角色
  - 传递不同的参数
  - 使用 allow_duplicates: true
- 使用角色依赖项
  - 在一个剧本中多次运行角色依赖项
- 在角色中嵌入模块和插件
- 共享角色：Ansible Galaxy



## 角色目录结构

Ansible 角色具有定义的目录结构，其中包含七个主要的标准目录。每个角色必须至少包含这些目录中的一个。您可以省略角色不使用的任何目录。例如：

```
# playbooks
site.yml
webservers.yml
fooservers.yml
```



```
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```



默认情况下，Ansible 将在大多数角色目录中查找 main.yml 文件以获取相关内容（也包括 main.yaml 和 main）

- tasks/main.yml - 角色提供给剧本以执行的任务列表。
- handlers/main.yml - 导入到父剧本中的处理器，供角色或剧本中的其他角色和任务使用。
- defaults/main.yml - 角色提供的变量的非常低的优先级值（有关更多信息，请参见 使用变量）。角色自身的默认值将优先于其他角色的默认值，但任何/所有其他变量来源都将覆盖此值。
- vars/main.yml - 角色提供给剧本的高优先级变量（有关更多信息，请参见 使用变量）。
- files/stuff.txt - 角色及其子角色可用的一个或多个文件。
- templates/something.j2 - 角色或子角色中使用的模板。
- meta/main.yml - 角色的元数据，包括角色依赖项和可选的 Galaxy 元数据（例如支持的平台）。这是作为独立角色上传到 Galaxy 所必需的，但对于在您的剧本中使用角色则不是必需的。

您可以在某些目录中添加其他 YAML 文件，但默认情况下不会使用它们。它们可以直接包含/导入，或者在使用 include_role/import_role 时指定。例如，您可以将特定于平台的任务放在单独的文件中，并在 tasks/main.yml 文件中引用它们。

```
# roles/example/tasks/main.yml
- name: Install the correct web server for RHEL
  import_tasks: redhat.yml
  when: ansible_facts['os_family']|lower == 'redhat'

- name: Install the correct web server for Debian
  import_tasks: debian.yml
  when: ansible_facts['os_family']|lower == 'debian'

# roles/example/tasks/redhat.yml
- name: Install web server
  ansible.builtin.yum:
    name: "httpd"
    state: present

# roles/example/tasks/debian.yml
- name: Install web server
  ansible.builtin.apt:
    name: "apache2"
    state: present
```



或者在加载角色时直接调用这些任务，这将绕过 main.yml 文件。

```
- name: include apt tasks
  include_role:
      name: package_manager_bootstrap
      tasks_from: apt.yml
  when: ansible_facts['os_family'] == 'Debian'
```



目录 defaults 和 vars 也可能包含 嵌套目录。如果您的变量文件是一个目录，Ansible 将按字母顺序读取该目录内的所有变量文件和目录。如果嵌套目录包含变量文件和目录，Ansible 将首先读取目录。下面是一个 vars/main 目录的示例。

```
roles/
    common/          # this hierarchy represents a "role"
        vars/
            main/    #  <-- variables associated with this role
                first_nested_directory/
                    first_variables_file.yml
                second_nested_directory/
                    second_variables_file.yml
                third_variables_file.yml
```





## 存储和查找角色

默认情况下，Ansible 在以下位置查找角色：

- 如果您正在使用集合，则在集合中。
- 在名为 roles/ 的目录中，相对于剧本文件。
- 在已配置的 roles_path 中。默认搜索路径是 ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles。
- 在剧本文件所在的目录中。

如果您将角色存储在其他位置，请设置 roles_path 配置选项，以便 Ansible 可以找到您的角色。将共享角色检入到单个位置可以更轻松地在多个剧本中使用它们。有关在 ansible.cfg 中管理设置的详细信息，请参见 配置 Ansible。

或者，您可以使用完全限定路径调用角色。

```
---
- hosts: webservers
  roles:
    - role: '/path/to/my/roles/common'
```



## 使用角色

您可以通过以下方式使用角色：

- 在剧本级别使用 roles 选项：这是在剧本中使用角色的经典方法。
- 在任务级别使用 include_role：您可以使用 include_role 在剧本的 tasks 部分中的任何位置动态重用角色。
- 在任务级别使用 import_role：您可以使用 import_role 在剧本的 tasks 部分中的任何位置静态重用角色。
- 作为另一个角色的依赖项（请参见此页面中 meta/main.yml 中的 dependencies 关键字）。



### 在剧本级别使用角色

使用角色的经典（原始）方法是使用给定剧本的 roles 选项。

```
---
- hosts: webservers
  roles:
    - common
    - webservers
```



当您在剧本级别使用 roles 选项时，每个角色“x”都会在以下目录中查找 main.yml（也包括 main.yaml 和 main）：

- roles/x/tasks/
- roles/x/handlers/
- roles/x/vars/
- roles/x/defaults/
- roles/x/meta/
- 任何复制、脚本、模板或包含任务（在角色中）都可以引用 roles/x/{files,templates,tasks/} 中的文件（目录取决于任务），而无需相对或绝对地对其进行路径设置。

在 Play 级别使用 roles 选项时，Ansible 将角色视为静态导入并在剧本解析期间处理它们。Ansible 按以下顺序执行每个 Play：

- Play 中定义的任何 pre_tasks。
- 由 pre_tasks 触发的任何处理器。
- 列在 roles: 中的每个角色，按照列出的顺序。角色的 meta/main.yml 中定义的任何角色依赖项将首先运行，但需遵守标签过滤和条件判断。更多详情，请参见 使用角色依赖项。
- Play 中定义的任何 tasks。
- 由角色或任务触发的任何处理器。
- Play 中定义的任何 post_tasks。
- 由 post_tasks 触发的任何处理器。

您可以将其他关键字传递给 roles 选项。

```
---
- hosts: webservers
  roles:
    - common
    - role: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
    - role: foo_app_instance
      vars:
        dir: '/opt/b'
        app_port: 5001
      tags: typeB
```



当您向 role 选项添加标签时，Ansible 会将该标签应用于角色中的所有任务。

### 包含角色：动态重用

您可以使用 include_role 在 Play 的 tasks 部分的任何位置动态重用角色。在 roles 部分中添加的角色在 Play 中任何其他任务之前运行，而包含的角色则按定义的顺序运行。如果在 include_role 任务之前还有其他任务，则其他任务将首先运行。

要包含角色：

```
---
- hosts: webservers
  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs before the example role"

    - name: Include the example role
      include_role:
        name: example

    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs after the example role"
```



包含角色时，您可以传递其他关键字，包括变量和标签。

```
---
- hosts: webservers
  tasks:
    - name: Include the foo_app_instance role
      include_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
  ...
```



当您向 include_role 任务添加 标签 时，Ansible 将该标签仅应用于 include 本身。这意味着您可以传递 --tags 来仅运行角色中选定的任务（如果这些任务本身具有与 include 语句相同的标签）。有关详细信息，请参见 选择性地运行可重用文件中的带标签的任务。

您可以有条件地包含角色。

```
---
- hosts: webservers
  tasks:
    - name: Include the some_role role
      include_role:
        name: some_role
      when: "ansible_facts['os_family'] == 'RedHat'"
```



### 导入角色：静态重用

您可以使用 import_role 在 Play 的 tasks 部分的任何位置静态重用角色。其行为与使用 roles 关键字相同。例如：

```
---
- hosts: webservers
  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "before we run our role"

    - name: Import the example role
      import_role:
        name: example

    - name: Print a message
      ansible.builtin.debug:
        msg: "after we ran our role"
```



导入角色时，您可以传递其他关键字，包括变量和标签。

```
---
- hosts: webservers
  tasks:
    - name: Import the foo_app_instance role
      import_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
  ...
```



当您向 import_role 语句添加标签时，Ansible 会将该标签应用于角色中的所有任务。有关详细信息，请参见 标签继承：向多个任务添加标签。



## 角色参数验证

从 2.11 版本开始，您可以选择根据参数规范启用角色参数验证。此规范在 meta/argument_specs.yml 文件（或使用 .yaml 文件扩展名）中定义。当定义此参数规范时，会在角色执行的开头插入一个新任务，该任务会根据规范验证为角色提供的参数。如果参数验证失败，则角色将执行失败。

### 规范格式

角色参数规范必须在角色 meta/argument_specs.yml 文件中的顶级 argument_specs 块中定义。所有字段都小写。

- entry-point-name：

  角色入口点的名称。如果未指定入口点，则应为 main。这将是待执行的任务文件的基名称，没有 .yml 或 .yaml 文件扩展名。short_description：入口点的简短单行描述。理想情况下，它是一个短语而不是句子。short_description 由 ansible-doc -t role -l 显示。它也成为文档中角色页面标题的一部分。简短描述应始终为字符串，切勿为列表，也不应以句点结尾。description：可以包含多行的较长描述。这可以是单个字符串或字符串列表。如果这是一个字符串列表，则每个列表元素都是一个新段落。version_added：添加入口点时角色的版本。这是一个字符串，而不是浮点数，例如 version_added: '2.1'。在集合中，这必须是添加入口点的集合版本。例如，version_added: 1.0.0。author：入口点作者的姓名。这可以是单个字符串或字符串列表。每个作者使用一个列表条目。如果只有一个作者，则使用字符串或单元素列表。options：选项通常称为“参数”。本节定义这些选项。对于每个角色选项（参数），您可以包含：option-name：选项/参数的名称。description：此选项作用的详细说明。它应该用完整的句子写成。这可以是单个字符串或字符串列表。如果这是一个字符串列表，则每个列表元素都是一个新段落。version_added：仅当此选项在初始角色/入口点发布后添加时才需要。换句话说，它大于顶级 version_added 字段。这是一个字符串，而不是浮点数，例如 version_added: '2.1'。在集合中，这必须是添加选项的集合版本。例如，version_added: 1.0.0。type：选项的数据类型。有关 type 的允许值，请参见 参数规范。默认为 str。如果选项的类型为 list，则应指定 elements。required：仅当 true 时才需要。如果缺失，则该选项不是必需的。default：如果 required 为 false/缺失，则可以指定 default（如果缺失，则假定为 null）。确保文档中的默认值与代码中的默认值匹配。角色变量的实际默认值将始终来自角色默认值（如 角色目录结构 中所定义）。除非默认值需要其他信息或条件，否则默认字段不得列为描述的一部分。如果选项是布尔值，为了兼容ansible-lint，你应该使用true/false。choices:选项值的列表。如果为空，则应省略。elements:当类型为list时，指定列表元素的数据类型。options：如果此选项接受字典或字典列表，你可以在此处定义其结构。

### 示例规范

```
# roles/myapp/meta/argument_specs.yml
---
argument_specs:
  # roles/myapp/tasks/main.yml entry point
  main:
    short_description: Main entry point for the myapp role
    description:
      - This is the main entrypoint for the C(myapp) role.
      - Here we can describe what this entrypoint does in lengthy words.
      - Every new list item is a new paragraph. You can have multiple sentences
        per paragraph.
    author:
      - Daniel Ziegenberg
    options:
      myapp_int:
        type: "int"
        required: false
        default: 42
        description:
          - "The integer value, defaulting to 42."
          - "This is a second paragraph."

      myapp_str:
        type: "str"
        required: true
        description: "The string value"

      myapp_list:
        type: "list"
        elements: "str"
        required: true
        description: "A list of string values."
        version_added: 1.3.0

      myapp_list_with_dicts:
        type: "list"
        elements: "dict"
        required: false
        default:
          - myapp_food_kind: "meat"
            myapp_food_boiling_required: true
            myapp_food_preparation_time: 60
          - myapp_food_kind: "fruits"
            myapp_food_preparation_time: 5
        description: "A list of dicts with a defined structure and with default a value."
        options:
          myapp_food_kind:
            type: "str"
            choices:
              - "vegetables"
              - "fruits"
              - "grains"
              - "meat"
            required: false
            description: "A string value with a limited list of allowed choices."

          myapp_food_boiling_required:
            type: "bool"
            required: false
            default: false
            description: "Whether the kind of food requires boiling before consumption."

          myapp_food_preparation_time:
            type: int
            required: true
            description: "Time to prepare a dish in minutes."

      myapp_dict_with_suboptions:
        type: "dict"
        required: false
        default:
          myapp_host: "bar.foo"
          myapp_exclude_host: true
          myapp_path: "/etc/myapp"
        description: "A dict with a defined structure and default values."
        options:
          myapp_host:
            type: "str"
            choices:
              - "foo.bar"
              - "bar.foo"
              - "ansible.foo.bar"
            required: true
            description: "A string value with a limited list of allowed choices."

          myapp_exclude_host:
            type: "bool"
            required: true
            description: "A boolean value."

          myapp_path:
            type: "path"
            required: true
            description: "A path value."

          original_name:
            type: list
            elements: "str"
            required: false
            description: "An optional list of string values."

  # roles/myapp/tasks/alternate.yml entry point
  alternate:
    short_description: Alternate entry point for the myapp role
    description:
      - This is the alternate entrypoint for the C(myapp) role.
    version_added: 1.2.0
    options:
      myapp_int:
        type: "int"
        required: false
        default: 1024
        description: "The integer value, defaulting to 1024."
```





## 在一个剧本中多次运行一个角色

Ansible在一个剧本中只执行每个角色一次，即使你多次定义它，除非为每个定义定义了不同的参数。例如，Ansible只在像这样的剧本中运行一次角色foo

```
---
- hosts: webservers
  roles:
    - foo
    - bar
    - foo
```



你有两种方法可以强制 Ansible 多次运行一个角色。

### 传递不同的参数

如果你在每个角色定义中传递不同的参数，Ansible 将多次运行该角色。提供不同的变量值与传递不同的角色参数不同。你必须为此行为使用roles关键字，因为import_role和include_role不接受角色参数。

此剧本将foo角色运行两次

```
---
- hosts: webservers
  roles:
    - { role: foo, message: "first" }
    - { role: foo, message: "second" }
```



此语法也会将foo角色运行两次；

```
---
- hosts: webservers
  roles:
    - role: foo
      message: "first"
    - role: foo
      message: "second"
```



在这些示例中，Ansible运行foo两次，因为每个角色定义都有不同的参数。

### 使用allow_duplicates: true

将allow_duplicates: true添加到角色的meta/main.yml文件中

```
# playbook.yml
---
- hosts: webservers
  roles:
    - foo
    - foo

# roles/foo/meta/main.yml
---
allow_duplicates: true
```



在这个例子中，Ansible运行foo两次，因为我们已经明确地启用了它。



## 使用角色依赖

角色依赖允许你在使用角色时自动引入其他角色。

角色依赖是先决条件，而不是真正的依赖关系。角色之间没有父子关系。Ansible加载所有列出的角色，首先运行在dependencies下列出的角色，然后运行列出它们的的。剧本对象是所有角色的父对象，包括由dependencies列表调用的角色。

角色依赖存储在角色目录中的meta/main.yml文件中。此文件应包含角色列表以及在指定角色之前插入的参数。例如

```
# roles/myapp/meta/main.yml
---
dependencies:
  - role: common
    vars:
      some_parameter: 3
  - role: apache
    vars:
      apache_port: 80
  - role: postgres
    vars:
      dbname: blarg
      other_parameter: 12
```



Ansible总是先执行在dependencies中列出的角色，然后再执行列出它们的那些角色。当你使用roles关键字时，Ansible会递归地执行此模式。例如，如果你在roles:下列出角色foo，角色foo在其meta/main.yml文件中在dependencies下列出角色bar，而角色bar在其meta/main.yml文件中在dependencies下列出角色baz，则Ansible将按顺序执行baz、bar和foo。

### 在一个剧本中多次运行角色依赖

Ansible 将重复的角色依赖视为在roles:下列出的重复角色：Ansible 只执行一次角色依赖，即使多次定义也是如此，除非角色上定义的参数、标签或 when 子句对于每个定义都不同。如果一个剧本中的两个角色都将第三个角色列为依赖项，Ansible 只运行一次该角色依赖项，除非你传递不同的参数、标签、when 子句，或在要多次运行的角色中使用allow_duplicates: true。有关更多详细信息，请参阅Galaxy 角色依赖项。

例如，名为car的角色依赖于名为wheel的角色，如下所示

```
---
dependencies:
  - role: wheel
    n: 1
  - role: wheel
    n: 2
  - role: wheel
    n: 3
  - role: wheel
    n: 4
```



而wheel角色依赖于两个角色：tire和brake。wheel的meta/main.yml将包含以下内容

```
---
dependencies:
  - role: tire
  - role: brake
```



而tire和brake的meta/main.yml将包含以下内容

```
---
allow_duplicates: true
```



最终的执行顺序如下所示

```
tire(n=1)
brake(n=1)
wheel(n=1)
tire(n=2)
brake(n=2)
wheel(n=2)
...
car
```



要在角色依赖项中使用allow_duplicates: true，必须为在dependencies下列出的角色指定它，而不是为列出它的角色指定它。在上面的示例中，allow_duplicates: true出现在tire和brake角色的meta/main.yml中。wheel角色不需要allow_duplicates: true，因为car定义的每个实例都使用不同的参数值。



## 在角色中嵌入模块和插件

如果你编写自定义模块（请参阅你应该开发模块吗？）或插件（请参阅开发插件），你可能希望将其作为角色的一部分进行分发。例如，如果你编写了一个模块来帮助配置你公司的内部软件，并且你希望组织中的其他人使用此模块，但又不想告诉每个人如何配置他们的 Ansible 库路径，你可以将该模块包含在你的internal_config角色中。

要将模块或插件添加到角色：在角色的“tasks”和“handlers”结构旁边，添加一个名为“library”的目录，然后将模块直接包含在“library”目录内。

假设你有这个

```
roles/
    my_custom_modules/
        library/
            module1
            module2
```



该模块可以在角色本身中使用，也可以在之后调用的任何角色中使用，如下所示

```
---
- hosts: webservers
  roles:
    - my_custom_modules
    - some_other_role_using_my_custom_modules
    - yet_another_role_using_my_custom_modules
```



如有必要，你还可以将模块嵌入到角色中以修改 Ansible 核心发行版中的模块。例如，你可以在生产版本发布之前使用特定模块的开发版本，方法是复制该模块并将副本嵌入到角色中。谨慎使用此方法，因为核心组件的 API 签名可能会发生更改，并且此解决方法不能保证有效。

可以使用相同的机制将插件嵌入到角色中并进行分发，使用相同的模式。例如，对于过滤器插件

```
roles/
    my_custom_filter/
        filter_plugins
            filter1
            filter2
```



然后可以在“my_custom_filter”之后调用的任何角色中的 Jinja 模板中使用这些过滤器。

## 共享角色：Ansible Galaxy

Ansible Galaxy 是一个免费网站，用于查找、下载、评价和审查各种社区开发的 Ansible 角色，可以帮助你快速启动自动化项目。

客户端ansible-galaxy包含在 Ansible 中。Galaxy 客户端允许你从 Ansible Galaxy 下载角色，并为创建你自己的角色提供了一个优秀的默认框架。

阅读[Ansible Galaxy 文档](https://readthedocs.ansible.org.cn/projects/galaxy-ng/en/latest/)页面以了解更多信息。
