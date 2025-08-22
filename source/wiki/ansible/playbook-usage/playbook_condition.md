---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.9 条件语句
order: 8
---

# 条件语句

在 playbook 中，您可能希望根据事实（关于远程系统的数据）、变量或先前任务的结果来执行不同的任务或拥有不同的目标。您可能希望某些变量的值取决于其他变量的值。或者，您可能希望根据主机是否与其他条件匹配来创建其他主机组。您可以使用条件语句来完成所有这些操作。

Ansible 在条件语句中使用 Jinja2 测试 和 过滤器。Ansible 支持所有标准测试和过滤器，并添加了一些独特的测试和过滤器。

- 使用 when 的基本条件语句
  - 基于 ansible_facts 的条件语句
  - 基于已注册变量的条件语句
  - 基于变量的条件语句
  - 在循环中使用条件语句
  - 加载自定义事实
  - 带有重用的条件语句
    - 带有导入的条件语句
    - 带有包含的条件语句
    - 带有角色的条件语句
  - 根据事实选择变量、文件或模板
    - 根据事实选择变量文件
    - 根据事实选择文件和模板
- 调试条件语句
- 常用事实
  - [ansible_facts'distribution']
  - [ansible_facts'distribution_major_version']
  - [ansible_facts'os_family']



## 使用 when 的基本条件语句

最简单的条件语句适用于单个任务。创建任务，然后添加一个 when 语句来应用测试。when 子句是一个原始的 Jinja2 表达式，没有双大括号 (参见 引用简单变量)。运行任务或 playbook 时，Ansible 会针对所有主机评估测试。在测试通过（返回 True 值）的任何主机上，Ansible 都会运行该任务。例如，如果您在某些启用了 SELinux 的多台机器上安装 mysql，您可能需要一个任务来配置 SELinux 以允许 mysql 运行。您只希望在启用了 SELinux 的机器上运行该任务。

```
tasks:
  - name: Configure SELinux to start mysql on any port
    ansible.posix.seboolean:
      name: mysql_connect_any
      state: true
      persistent: true
    when: ansible_selinux.status == "enabled"
    # all variables can be used directly in conditionals without double curly braces
```



### 基于 ansible_facts 的条件语句

通常，您希望根据事实来执行或跳过任务。事实是单个主机的属性，包括 IP 地址、操作系统、文件系统的状态等等。使用基于事实的条件语句：

> - 只有当操作系统是特定版本时，才能安装特定软件包。
> - 您可以跳过对具有内部 IP 地址的主机配置防火墙。
> - 只有当文件系统即将满时，才能执行清理任务。

请参见 常用事实，了解在条件语句中经常出现的事实列表。并非所有事实都存在于所有主机中。例如，下面示例中使用的“lsb_major_release”事实仅在目标主机上安装了 lsb_release package 时才存在。要查看系统上可用的事实，请将调试任务添加到您的 playbook 中。

```
- name: Show facts available on the system
  ansible.builtin.debug:
    var: ansible_facts
```



这是一个基于事实的条件语句示例：

```
tasks:
  - name: Shut down Debian flavored systems
    ansible.builtin.command: /sbin/shutdown -t now
    when: ansible_facts['os_family'] == "Debian"
```



如果有多个条件，可以使用括号将它们组合在一起。

```
tasks:
  - name: Shut down CentOS 6 and Debian 7 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
          (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")
```



您可以使用 逻辑运算符 来组合条件。当您有多个需要全部为真的条件（即逻辑 and）时，可以将其指定为列表。

```
tasks:
  - name: Shut down CentOS 6 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"
```



如果事实或变量是字符串，并且您需要对其进行数学比较，请使用过滤器确保 Ansible 将值读取为整数。

```
tasks:
  - ansible.builtin.shell: echo "only on Red Hat 6, derivatives, and later"
    when: ansible_facts['os_family'] == "RedHat" and ansible_facts['lsb']['major_release'] | int >= 6
```



您可以将 Ansible 事实存储为变量，以用于条件逻辑，如下例所示：

```
tasks:
    - name: Get the CPU temperature
      set_fact:
        temperature: "{{ ansible_facts['cpu_temperature'] }}"

    - name: Restart the system if the temperature is too high
      when: temperature | float > 90
      shell: "reboot"
```





### 基于已注册变量的条件语句

通常，在 playbook 中，您希望根据先前任务的结果来执行或跳过任务。例如，您可能希望在先前的任务升级服务后配置服务。要创建基于已注册变量的条件语句：

> 1. 将先前任务的结果注册为变量。
> 2. 创建基于已注册变量的条件测试。

您可以使用 register 关键字创建已注册变量的名称。已注册变量始终包含创建它的任务的状态以及任务生成的任何输出。您可以在模板和操作行以及条件 when 语句中使用已注册变量。您可以使用 variable.stdout 访问已注册变量的字符串内容。例如：

```
- name: Test play
  hosts: all

  tasks:

      - name: Register a variable
        ansible.builtin.shell: cat /etc/motd
        register: motd_contents

      - name: Use the variable in conditional statement
        ansible.builtin.shell: echo "motd contains the word hi"
        when: motd_contents.stdout.find('hi') != -1
```



如果变量是列表，则可以在任务循环中使用已注册的结果。如果变量不是列表，则可以使用 stdout_lines 或 variable.stdout.split() 将其转换为列表。您也可以按其他字段拆分行。

```
- name: Registered variable usage as a loop list
  hosts: all
  tasks:

    - name: Retrieve the list of home directories
      ansible.builtin.command: ls /home
      register: home_dirs

    - name: Add home dirs to the backup spooler
      ansible.builtin.file:
        path: /mnt/bkspool/{{ item }}
        src: /home/{{ item }}
        state: link
      loop: "{{ home_dirs.stdout_lines }}"
      # same as loop: "{{ home_dirs.stdout.split() }}"
```



已注册变量的字符串内容可能是空的。如果您只想在已注册变量的标准输出为空的主机上运行另一个任务，请检查已注册变量的字符串内容是否为空。

```
- name: check registered variable for emptiness
  hosts: all

  tasks:

      - name: List contents of directory
        ansible.builtin.command: ls mydir
        register: contents

      - name: Check contents for emptiness
        ansible.builtin.debug:
          msg: "Directory is empty"
        when: contents.stdout == ""
```



Ansible 始终为每个主机在已注册变量中注册某些内容，即使在任务失败或 Ansible 由于条件未满足而跳过任务的主机上也是如此。要在这些主机上运行后续任务，请查询已注册变量的 is skipped（而不是“undefined”或“default”）。有关详细信息，请参见 注册变量。以下是基于任务成功或失败的条件语句示例。如果希望 Ansible 在发生错误时继续在主机上执行，请记住忽略错误。

```
tasks:
  - name: Register a variable, ignore errors and continue
    ansible.builtin.command: /bin/false
    register: result
    ignore_errors: true

  - name: Run only if the task that registered the "result" variable fails
    ansible.builtin.command: /bin/something
    when: result is failed

  - name: Run only if the task that registered the "result" variable succeeds
    ansible.builtin.command: /bin/something_else
    when: result is succeeded

  - name: Run only if the task that registered the "result" variable is skipped
    ansible.builtin.command: /bin/still/something_else
    when: result is skipped

  - name: Run only if the task that registered the "result" variable changed something.
    ansible.builtin.command: /bin/still/something_else
    when: result is changed
```



### 基于变量的条件语句

您还可以根据在 playbook 或清单中定义的变量创建条件语句。因为条件语句需要布尔输入（测试必须评估为 True 才能触发条件），所以您必须将 | bool 过滤器应用于非布尔变量，例如包含“yes”、“on”、“1”或“true”之类的内容的字符串变量。您可以像这样定义变量：

```
vars:
  epic: true
  monumental: "yes"
```



使用上面的变量，Ansible 将运行其中一个任务并跳过另一个任务。

```
tasks:
    - name: Run the command if "epic" or "monumental" is true
      ansible.builtin.shell: echo "This certainly is epic!"
      when: epic or monumental | bool

    - name: Run the command if "epic" is false
      ansible.builtin.shell: echo "This certainly isn't epic!"
      when: not epic
```



如果未设置所需的变量，您可以使用 Jinja2 的 defined 测试来跳过或失败。例如：

```
tasks:
    - name: Run the command if "foo" is defined
      ansible.builtin.shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
      when: foo is defined

    - name: Fail if "bar" is undefined
      ansible.builtin.fail: msg="Bailing out. This play requires 'bar'"
      when: bar is undefined
```



这与 vars 文件的条件导入（见下文）结合使用特别有用。如示例所示，您无需使用 {{ }} 来在条件语句中使用变量，因为这些变量已经隐含。



### 在循环中使用条件语句

如果将 when 语句与 循环 结合使用，Ansible 会分别处理每个项目的条件。这是设计使然，因此您可以对循环中的某些项目执行任务，而跳过其他项目。例如：

```
tasks:
    - name: Run with items greater than 5
      ansible.builtin.command: echo {{ item }}
      loop: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5
```



如果需要在循环变量未定义时跳过整个任务，请使用 |default 过滤器提供一个空的迭代器。例如，在循环遍历列表时：

```
- name: Skip the whole task when a loop variable is undefined
  ansible.builtin.command: echo {{ item }}
  loop: "{{ mylist|default([]) }}"
  when: item > 5
```



在循环遍历字典时，也可以执行相同的操作：

```
- name: The same as above using a dict
  ansible.builtin.command: echo {{ item.key }}
  loop: "{{ query('dict', mydict|default({})) }}"
  when: item.value > 5
```





### 加载自定义事实

您可以提供自己的事实，如 您是否应该开发模块？ 中所述。要运行它们，只需在任务列表的顶部调用您自己的自定义事实收集模块，然后可以访问此处返回的变量以用于将来的任务。

```
tasks:
    - name: Gather site specific fact data
      action: site_facts

    - name: Use a custom fact
      ansible.builtin.command: /usr/bin/thingy
      when: my_custom_fact_just_retrieved_from_the_remote_system == '1234'
```





### 带有重用的条件语句

您可以在可重用的任务文件、剧本或角色中使用条件语句。Ansible 对动态重用（includes）和静态重用（imports）的条件语句执行方式不同。有关 Ansible 中重用的更多信息，请参阅重用 Ansible 工件。



#### 使用 import 的条件语句

当您向 import 语句添加条件语句时，Ansible 会将此条件应用于导入文件中所有任务。此行为等效于标签继承：向多个任务添加标签。Ansible 将条件应用于每个任务并分别评估每个任务。例如，如果您想定义然后显示一个以前未定义的变量，您可能有一个名为main.yml的剧本和一个名为other_tasks.yml的任务文件。

```
# all tasks within an imported file inherit the condition from the import statement
# main.yml
- hosts: all
  tasks:
  - import_tasks: other_tasks.yml # note "import"
    when: x is not defined

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo

- name: Print a variable
  ansible.builtin.debug:
    var: x
```



Ansible 在执行时将其扩展为等效于以下内容：

```
- name: Set a variable if not defined
  ansible.builtin.set_fact:
    x: foo
  when: x is not defined
  # this task sets a value for x

- name: Do the task if "x" is not defined
  ansible.builtin.debug:
    var: x
  when: x is not defined
  # Ansible skips this task, because x is now defined
```



如果x最初已定义，则两个任务都将按预期跳过。但如果x最初未定义，则调试任务将被跳过，因为条件是针对每个导入的任务进行评估的。对于set_fact任务，条件将评估为true，这将定义变量并导致debug条件评估为false。

如果这不是您想要的行为，请使用include_*语句仅将条件应用于该语句本身。

```
# using a conditional on include_* only applies to the include task itself
# main.yml
- hosts: all
  tasks:
  - include_tasks: other_tasks.yml # note "include"
    when: x is not defined
```



现在，如果x最初未定义，则调试任务不会被跳过，因为条件是在 include 时进行评估的，并且不适用于各个任务。

您可以将条件应用于import_playbook以及其他import_*语句。当您使用此方法时，Ansible 会针对每个与条件不匹配的主机上的每个任务返回“已跳过”消息，从而产生重复的输出。在许多情况下，group_by 模块 可以成为实现相同目标的更简化方法；请参阅处理操作系统和发行版差异。



#### 使用 include 的条件语句

当您在include_*语句上使用条件语句时，该条件仅应用于 include 任务本身，而不应用于包含文件中的任何其他任务。为了与上面关于 import 条件语句的示例形成对比，请查看相同的剧本和任务文件，但使用 include 而不是 import。

```
# Includes let you reuse a file to define a variable when it is not already defined

# main.yml
- include_tasks: other_tasks.yml
  when: x is not defined

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo

- name: Print a variable
  ansible.builtin.debug:
    var: x
```



Ansible 在执行时将其扩展为等效于以下内容：

```
# main.yml
- include_tasks: other_tasks.yml
  when: x is not defined
  # if condition is met, Ansible includes other_tasks.yml

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo
  # no condition applied to this task, Ansible sets the value of x to foo

- name: Print a variable
  ansible.builtin.debug:
    var: x
  # no condition applied to this task, Ansible prints the debug statement
```



通过使用include_tasks而不是import_tasks，将按预期执行other_tasks.yml中的两个任务。有关include与import之间差异的更多信息，请参阅重用 Ansible 工件。

#### 使用角色的条件语句

有三种方法可以将条件应用于角色

> - 通过将您的when语句放在roles关键字下，将相同的条件应用于角色中的所有任务。请参阅本节中的示例。
> - 通过将您的when语句放在剧本中的静态import_role上，将相同的条件应用于角色中的所有任务。
> - 向角色本身内的各个任务或块添加条件。这是唯一允许您根据when语句选择或跳过角色中某些任务的方法。要选择或跳过角色中的任务，您必须在各个任务或块上设置条件，在您的剧本中使用动态include_role，并将条件添加到 include。当您使用此方法时，Ansible 会将条件应用于 include 本身以及角色中也具有该when语句的任何任务。

当您使用roles关键字在剧本中静态地包含角色时，Ansible 会将您定义的条件添加到角色中的所有任务。例如：

```
- hosts: webservers
  roles:
     - role: debian_stock_config
       when: ansible_facts['os_family'] == 'Debian'
```





### 根据事实选择变量、文件或模板

有时，主机的事实决定了您想要用于某些变量的值，甚至是您想要为该主机选择的变量或模板。例如，CentOS 和 Debian 上的软件包名称不同。常用服务的配置文件在不同的操作系统版本和类型上也各不相同。要根据主机的事实加载不同的变量文件、模板或其他文件：

> 1. 将您的变量文件、模板或文件命名为与区分它们的 Ansible 事实相匹配。
> 2. 使用基于该 Ansible 事实的变量为每个主机选择正确的变量文件、模板或文件。

Ansible 将变量与任务分开，防止您的剧本变成包含嵌套条件语句的任意代码。这种方法会产生更简洁且更易于审核的配置规则，因为需要跟踪的决策点更少。

#### 根据事实选择变量文件

您可以创建一个在多个平台和操作系统版本上工作的剧本，只需最少的语法即可将您的变量值放在变量文件中并有条件地导入它们。如果您想在一些 CentOS 和一些 Debian 服务器上安装 Apache，请创建具有 YAML 密钥和值的变量文件。例如：

```
---
# for vars/RedHat.yml
apache: httpd
somethingelse: 42
```



然后根据您在剧本中收集的主机事实导入这些变量文件。

```
---
- hosts: webservers
  remote_user: root
  vars_files:
    - "vars/common.yml"
    - [ "vars/{{ ansible_facts['os_family'] }}.yml", "vars/os_defaults.yml" ]
  tasks:
  - name: Make sure apache is started
    ansible.builtin.service:
      name: '{{ apache }}'
      state: started
```



Ansible 收集 webservers 组中主机的相关事实，然后将变量“ansible_facts['os_family']”插入到文件名列表中。如果您有使用 Red Hat 操作系统（例如 CentOS）的主机，Ansible 将查找“vars/RedHat.yml”。如果该文件不存在，Ansible 将尝试加载“vars/os_defaults.yml”。对于 Debian 主机，Ansible 将首先查找“vars/Debian.yml”，然后再回退到“vars/os_defaults.yml”。如果找不到列表中的任何文件，Ansible 将引发错误。

#### 根据事实选择文件和模板

当不同的操作系统版本或类型需要不同的配置文件或模板时，您可以使用相同的方法。根据分配给每个主机的变量选择相应的文件或模板。这种方法通常比在单个模板中放置大量条件语句来涵盖多个操作系统或软件包版本要简洁得多。

例如，您可以为 CentOS 和 Debian 之间的配置文件创建不同的模板。

```
- name: Template a file
  ansible.builtin.template:
    src: "{{ item }}"
    dest: /etc/myapp/foo.conf
  loop: "{{ query('first_found', { 'files': myfiles, 'paths': mypaths}) }}"
  vars:
    myfiles:
      - "{{ ansible_facts['distribution'] }}.conf"
      -  default.conf
    mypaths: ['search_location_one/somedir/', '/opt/other_location/somedir/']
```





## 调试条件语句

如果您的条件语句when的行为与您预期不符，您可以添加debug语句来确定条件是否评估为true或false。条件语句中意外行为的一个常见原因是将整数作为字符串或将字符串作为整数进行测试。要调试条件语句，请将整个语句作为debug任务中的var:值添加。然后，Ansible 将显示测试以及语句的评估方式。例如，这是一组任务和示例输出：

```
- name: check value of return code
  ansible.builtin.debug:
    var: bar_status.rc

- name: check test for rc value as string
  ansible.builtin.debug:
    var: bar_status.rc == "127"

- name: check test for rc value as integer
  ansible.builtin.debug:
    var: bar_status.rc == 127
```



```
TASK [check value of return code] *********************************************************************************
ok: [foo-1] => {
    "bar_status.rc": "127"
}

TASK [check test for rc value as string] **************************************************************************
ok: [foo-1] => {
    "bar_status.rc == \"127\"": false
}

TASK [check test for rc value as integer] *************************************************************************
ok: [foo-1] => {
    "bar_status.rc == 127": true
}
```





## 常用事实

以下 Ansible 事实经常用于条件语句中。



### [ansible_facts'distribution']

可能的值（示例，并非完整列表）：

```
Alpine
Altlinux
Amazon
Archlinux
ClearLinux
Coreos
CentOS
Debian
Fedora
Gentoo
Mandriva
NA
OpenWrt
OracleLinux
RedHat
Slackware
SLES
SMGL
SUSE
Ubuntu
VMwareESX
```





### [ansible_facts'distribution_major_version']

操作系统的 major 版本。例如，Ubuntu 16.04 的值为16。



### [ansible_facts'os_family']

可能的值（示例，并非完整列表）：

```
AIX
Alpine
Altlinux
Archlinux
Darwin
Debian
FreeBSD
Gentoo
HP-UX
Mandrake
RedHat
SMGL
Slackware
Solaris
Suse
Windows
```
