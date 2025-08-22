---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.7 循环语句
order: 6
---

# 循环语句

Ansible 提供了 loop、with_<lookup> 和 until 关键字来多次执行任务。常用循环的示例包括使用 file 模块更改多个文件和/或目录的所有权，使用 user 模块创建多个用户，以及重复轮询步骤直到达到特定结果。

- 比较循环
- 使用循环
  - 迭代简单列表
  - 迭代哈希列表
  - 迭代字典
  - 使用循环注册变量
  - 重试任务直到满足条件
  - 循环遍历清单
- 确保 loop 的列表输入：使用 query 而不是 lookup
- 向循环添加控制
  - 使用 label 限制循环输出
  - 在循环内暂停
  - 使用 index_var 跟踪循环进度
  - 扩展的循环变量
  - 访问 loop_var 的名称
- 嵌套循环
  - 迭代嵌套列表
  - 通过 include_tasks 堆叠循环
  - Until 和 loop
- 从 with_X 迁移到 loop
  - with_list
  - with_items
  - with_indexed_items
  - with_flattened
  - with_together
  - with_dict
  - with_sequence
  - with_subelements
  - with_nested/with_cartesian
  - with_random_choice

## 比较循环

- until 的正常用例是处理可能失败的任务，而 loop 和 with_<lookup> 用于重复执行略有不同的任务。
- loop 和 with_<lookup> 将对用作输入的列表中的每个项目运行一次任务，而 until 将重新运行任务直到满足条件。对于程序员来说，前者是“for 循环”，后者是“while/until 循环”。
- with_<lookup> 关键字依赖于 查找插件 - 甚至 items 也是一个查找。
- loop 关键字等效于 with_list，是简单循环的最佳选择。
- loop 关键字不会接受字符串作为输入，请参阅 确保 loop 的列表输入：使用 query 而不是 lookup。
- until 关键字接受一个“结束条件”（返回 True 或 False 的表达式），该条件是“隐式模板化”的（不需要 {{ }}），通常基于您为任务 register 的变量。
- loop_control 同时影响 loop 和 with_<lookup>，但不影响 until，后者有自己的配套关键字：retries 和 delay。
- 一般来说，从 with_X 迁移到 loop 中涵盖的任何 with_* 的使用都可以更新为使用 loop。
- 将 with_items 更改为 loop 时要小心，因为 with_items 执行隐式的单级展平。您可能需要将 | flatten(1) 与 loop 一起使用才能匹配确切的结果。例如，要获得与

```
with_items:
  - 1
  - [2,3]
  - 4
```



相同的结果，您需要

```
loop: "{{ [1, [2, 3], 4] | flatten(1) }}"
```



- 任何需要在循环中使用 lookup 的 with_* 语句都不应转换为使用 loop 关键字。例如，与其这样做

```
loop: "{{ lookup('fileglob', '*.txt', wantlist=True) }}"
```



保持这样会更简洁

```
with_fileglob: '*.txt'
```





## 使用循环

### 迭代简单列表

重复的任务可以编写为在简单字符串列表上的标准循环。您可以在任务中直接定义列表。

```
- name: Add several users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
    groups: "wheel"
  loop:
     - testuser1
     - testuser2
```



您可以在变量文件中或 play 的 ‘vars’ 部分中定义列表，然后在任务中引用列表的名称。

```
loop: "{{ somelist }}"
```



这些示例中的任何一个都等效于

```
- name: Add user testuser1
  ansible.builtin.user:
    name: "testuser1"
    state: present
    groups: "wheel"

- name: Add user testuser2
  ansible.builtin.user:
    name: "testuser2"
    state: present
    groups: "wheel"
```



您可以将列表直接传递给某些插件的参数。大多数打包模块，如 yum 和 apt，都具有此功能。如果可用，将列表传递给参数比循环遍历任务更好。例如

```
- name: Optimal yum
  ansible.builtin.yum:
    name: "{{ list_of_packages }}"
    state: present

- name: Non-optimal yum, slower and may cause issues with interdependencies
  ansible.builtin.yum:
    name: "{{ item }}"
    state: present
  loop: "{{ list_of_packages }}"
```



查看 模块文档，以查看是否可以将列表传递给任何特定模块的参数。

### 迭代哈希列表

如果您有一个哈希列表，您可以在循环中引用子键。例如

```
- name: Add several users
  ansible.builtin.user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```



当将 条件 与循环结合使用时，将为每个项目单独处理 when: 语句。有关示例，请参阅 带有 when 的基本条件。

### 迭代字典

要循环遍历字典，请使用 dict2items

```
- name: Using dict2items
  ansible.builtin.debug:
    msg: "{{ item.key }} - {{ item.value }}"
  loop: "{{ tag_data | dict2items }}"
  vars:
    tag_data:
      Environment: dev
      Application: payment
```



在这里，我们正在迭代 tag_data 并从中打印键和值。

### 使用循环注册变量

您可以将循环的输出注册为变量。例如

```
- name: Register loop output as a variable
  ansible.builtin.shell: "echo {{ item }}"
  loop:
    - "one"
    - "two"
  register: echo
```



当你在循环中使用 register 时，放置在变量中的数据结构将包含一个 results 属性，该属性是一个包含模块所有响应的列表。这与不使用循环而使用 register 时返回的数据结构不同。在 results 旁边的 changed/failed/skipped 属性将表示整体状态。如果至少有一个迭代触发了更改/失败，则 changed/failed 将为 true，而只有当所有迭代都被跳过时，skipped 才会为 true。

```
{
    "changed": true,
    "msg": "All items completed",
    "results": [
        {
            "changed": true,
            "cmd": "echo \"one\" ",
            "delta": "0:00:00.003110",
            "end": "2013-12-19 12:00:05.187153",
            "invocation": {
                "module_args": "echo \"one\"",
                "module_name": "shell"
            },
            "item": "one",
            "rc": 0,
            "start": "2013-12-19 12:00:05.184043",
            "stderr": "",
            "stdout": "one"
        },
        {
            "changed": true,
            "cmd": "echo \"two\" ",
            "delta": "0:00:00.002920",
            "end": "2013-12-19 12:00:05.245502",
            "invocation": {
                "module_args": "echo \"two\"",
                "module_name": "shell"
            },
            "item": "two",
            "rc": 0,
            "start": "2013-12-19 12:00:05.242582",
            "stderr": "",
            "stdout": "two"
        }
    ]
}
```



后续对已注册变量进行循环以检查结果可能如下所示

```
- name: Fail if return code is not 0
  ansible.builtin.fail:
    msg: "The command ({{ item.cmd }}) did not have a 0 return code"
  when: item.rc != 0
  loop: "{{ echo.results }}"
```



在迭代期间，当前项的结果将被放置在变量中。

```
- name: Place the result of the current item in the variable
  ansible.builtin.shell: echo "{{ item }}"
  loop:
    - one
    - two
  register: echo
  changed_when: echo.stdout != "one"
```





### 重试任务直到满足条件

1.4版本新增功能。

您可以使用 until 关键字来重试任务，直到满足特定条件。这是一个示例

```
- name: Retry a task until a certain condition is met
  ansible.builtin.shell: /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1
  retries: 5
  delay: 10
```



此任务最多运行 5 次，每次尝试之间延迟 10 秒。如果任何一次尝试的结果在其 stdout 中包含“all systems go”，则任务成功。“retries”的默认值为 3，“delay”的默认值为 5。

要查看单个重试的结果，请使用 -vv 运行 playbook。

当您使用 until 运行任务并将结果注册为变量时，已注册的变量将包含一个名为“attempts”的键，该键记录了任务的重试次数。

如果未指定 until，则任务将重试直到任务成功，但最多重试 retries 次（2.16版本新增）。

您可以将 until 关键字与 loop 或 with_<lookup> 结合使用。循环中每个元素的任务结果都注册在变量中，并可在 until 条件中使用。这是一个示例

```
- name: Retry combined with a loop
  uri:
    url: "https://{{ item }}.ansible.com"
    method: GET
  register: uri_output
  with_items:
  - "galaxy"
  - "docs"
  - "forum"
  - "www"
  retries: 2
  delay: 1
  until: "uri_output.status == 200"
```





### 循环遍历清单

通常，playbook 本身就是对您的清单的循环，但有时您需要一个任务对不同的主机集执行相同的操作。要循环遍历您的清单，或者只是其中的一个子集，您可以使用带有 ansible_play_batch 或 groups 变量的常规 loop。

```
- name: Show all the hosts in the inventory
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ groups['all'] }}"

- name: Show all the hosts in the current play
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ ansible_play_batch }}"
```



还有一个特定的查找插件 inventory_hostnames，可以像这样使用

```
- name: Show all the hosts in the inventory
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ query('inventory_hostnames', 'all') }}"

- name: Show all the hosts matching the pattern, ie all but the group www
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ query('inventory_hostnames', 'all:!www') }}"
```



有关模式的更多信息，请参见 模式：目标主机和组。



## 确保 loop 的列表输入：使用 query 而不是 lookup

loop 关键字需要一个列表作为输入，但 lookup 关键字默认返回一个逗号分隔的值字符串。Ansible 2.5 引入了一个名为 query 的新 Jinja2 函数，该函数始终返回一个列表，从而在使用 loop 关键字时提供了更简单的接口和来自查找插件的更可预测的输出。

您可以使用 wantlist=True 强制 lookup 向 loop 返回列表，或者您可以改用 query。

以下两个示例执行相同的操作。

```
loop: "{{ query('inventory_hostnames', 'all') }}"

loop: "{{ lookup('inventory_hostnames', 'all', wantlist=True) }}"
```





## 向循环添加控制

2.1版本新增功能。

loop_control 关键字允许您以有用的方式管理循环。

### 使用 label 限制循环输出

2.2版本新增功能。

当循环遍历复杂的数据结构时，任务的控制台输出可能非常庞大。要限制显示的输出，请将 label 指令与 loop_control 一起使用。

```
- name: Create servers
  digital_ocean:
    name: "{{ item.name }}"
    state: present
  loop:
    - name: server1
      disks: 3gb
      ram: 15Gb
      network:
        nic01: 100Gb
        nic02: 10Gb
        ...
  loop_control:
    label: "{{ item.name }}"
```



此任务的输出将仅显示每个 item 的 name 字段，而不是多行 {{ item }} 变量的全部内容。

### 在循环中暂停

2.2版本新增功能。

要控制任务循环中每个项目执行之间的时间（以秒为单位），请将 pause 指令与 loop_control 一起使用。

```
# main.yml
- name: Create servers, pause 3s before creating next
  community.digitalocean.digital_ocean:
    name: "{{ item }}"
    state: present
  loop:
    - server1
    - server2
  loop_control:
    pause: 3
```



### 使用 index_var 跟踪循环进度

2.5版本新增功能。

要跟踪您在循环中的位置，请将 index_var 指令与 loop_control 一起使用。此指令指定一个变量名以包含当前循环索引。

```
- name: Count our fruit
  ansible.builtin.debug:
    msg: "{{ item }} with index {{ my_idx }}"
  loop:
    - apple
    - banana
    - pear
  loop_control:
    index_var: my_idx
```



### 扩展循环变量

2.8版本新增功能。

从 Ansible 2.8 开始，您可以使用循环控制的 extended 选项获取扩展的循环信息。此选项将公开以下信息。

| 变量                   | 描述                                                 |
| ---------------------- | ---------------------------------------------------- |
| ansible_loop.allitems  | 循环中所有项的列表                                   |
| ansible_loop.index     | 循环的当前迭代。（1 索引）                           |
| ansible_loop.index0    | 循环的当前迭代。（0 索引）                           |
| ansible_loop.revindex  | 从循环末尾开始的迭代次数（1 索引）                   |
| ansible_loop.revindex0 | 从循环末尾开始的迭代次数（0 索引）                   |
| ansible_loop.first     | 如果为第一次迭代，则为 True                          |
| ansible_loop.last      | 如果为最后一次迭代，则为 True                        |
| ansible_loop.length    | 循环中的项目数                                       |
| ansible_loop.previtem  | 来自循环上一次迭代的项目。在第一次迭代期间未定义。   |
| ansible_loop.nextitem  | 来自循环下一次迭代的项目。在最后一次迭代期间未定义。 |

```
loop_control:
  extended: true
```



2.14版本新增功能。

要禁用 ansible_loop.allitems 项以减少内存消耗，请设置 loop_control.extended_allitems: false。

```
loop_control:
  extended: true
  extended_allitems: false
```



### 访问您的 loop_var 的名称

2.8版本新增功能。

从 Ansible 2.8 开始，您可以使用 ansible_loop_var 变量获取提供给 loop_control.loop_var 的值的名称

对于角色作者，编写允许循环的角色，而不是指定所需的 loop_var 值，您可以通过以下方式收集值

```
"{{ lookup('vars', ansible_loop_var) }}"
```





## 嵌套循环

虽然我们在这些示例中使用 loop，但这同样适用于 with_<lookup>。

### 迭代嵌套列表

“嵌套”循环最简单的方法是避免嵌套循环，只需格式化数据以实现相同的结果。您可以使用 Jinja2 表达式来迭代复杂的列表。例如，一个循环可以组合嵌套列表，从而模拟嵌套循环。

```
- name: Give users access to multiple databases
  community.mysql.mysql_user:
    name: "{{ item[0] }}"
    priv: "{{ item[1] }}.*:ALL"
    append_privs: true
    password: "foo"
  loop: "{{ ['alice', 'bob'] | product(['clientdb', 'employeedb', 'providerdb']) | list }}"
```



### 通过 include_tasks 堆叠循环

2.1版本新增功能。

您可以使用 include_tasks 嵌套两个循环任务。但是，默认情况下，Ansible 会为每个循环设置循环变量 item。这意味着内部的嵌套循环会覆盖外部循环的 item 值。为了避免这种情况，您可以使用 loop_control 指定每个循环的变量名称，即 loop_var。

```
# main.yml
- include_tasks: inner.yml
  loop:
    - 1
    - 2
    - 3
  loop_control:
    loop_var: outer_item

# inner.yml
- name: Print outer and inner items
  ansible.builtin.debug:
    msg: "outer item={{ outer_item }} inner item={{ item }}"
  loop:
    - a
    - b
    - c
```



### Until 和 loop

until 条件将应用于 loop 的每个 item。

```
- debug: msg={{item}}
  loop:
    - 1
    - 2
    - 3
  retries: 2
  until: item > 2
```



这将使 Ansible 重试前两个项目两次，然后在第三次尝试时使项目失败，然后在第三个项目的第一次尝试时成功，最终导致整个任务失败。

```
[started TASK: debug on localhost]
FAILED - RETRYING: [localhost]: debug (2 retries left).Result was: {
    "attempts": 1,
    "changed": false,
    "msg": 1,
    "retries": 3
}
FAILED - RETRYING: [localhost]: debug (1 retries left).Result was: {
    "attempts": 2,
    "changed": false,
    "msg": 1,
    "retries": 3
}
failed: [localhost] (item=1) => {
    "msg": 1
}
FAILED - RETRYING: [localhost]: debug (2 retries left).Result was: {
    "attempts": 1,
    "changed": false,
    "msg": 2,
    "retries": 3
}
FAILED - RETRYING: [localhost]: debug (1 retries left).Result was: {
    "attempts": 2,
    "changed": false,
    "msg": 2,
    "retries": 3
}
failed: [localhost] (item=2) => {
    "msg": 2
}
ok: [localhost] => (item=3) => {
    "msg": 3
}
fatal: [localhost]: FAILED! => {"msg": "One or more items failed"}
```





## 从 with_X 迁移到 loop

在大多数情况下，循环最好使用 loop 关键字而不是 with_X 样式的循环。loop 语法通常最好使用过滤器来表达，而不是更复杂地使用 query 或 lookup。

这些示例展示了如何将许多常见的 with_ 样式循环转换为 loop 和过滤器。

### with_list

with_list 直接被 loop 替换。

```
- name: with_list
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_list:
    - one
    - two

- name: with_list -> loop
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop:
    - one
    - two
```



### with_items

with_items 被 loop 和 flatten 过滤器替换。

```
- name: with_items
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_items: "{{ items }}"

- name: with_items -> loop
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ items|flatten(levels=1) }}"
```



### with_indexed_items

with_indexed_items 被 loop、flatten 过滤器和 loop_control.index_var 替换。

```
- name: with_indexed_items
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_indexed_items: "{{ items }}"

- name: with_indexed_items -> loop
  ansible.builtin.debug:
    msg: "{{ index }} - {{ item }}"
  loop: "{{ items|flatten(levels=1) }}"
  loop_control:
    index_var: index
```



### with_flattened

with_flattened 被 loop 和 flatten 过滤器替换。

```
- name: with_flattened
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_flattened: "{{ items }}"

- name: with_flattened -> loop
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop: "{{ items|flatten }}"
```



### with_together

with_together 被 loop 和 zip 过滤器替换。

```
- name: with_together
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_together:
    - "{{ list_one }}"
    - "{{ list_two }}"

- name: with_together -> loop
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ list_one|zip(list_two)|list }}"
```



另一个包含复杂数据的示例

```
- name: with_together -> loop
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }} - {{ item.2 }}"
  loop: "{{ data[0]|zip(*data[1:])|list }}"
  vars:
    data:
      - ['a', 'b', 'c']
      - ['d', 'e', 'f']
      - ['g', 'h', 'i']
```



### with_dict

with_dict 可以用 loop 和 dictsort 或 dict2items 过滤器替换。

```
- name: with_dict
  ansible.builtin.debug:
    msg: "{{ item.key }} - {{ item.value }}"
  with_dict: "{{ dictionary }}"

- name: with_dict -> loop (option 1)
  ansible.builtin.debug:
    msg: "{{ item.key }} - {{ item.value }}"
  loop: "{{ dictionary|dict2items }}"

- name: with_dict -> loop (option 2)
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ dictionary|dictsort }}"
```



### with_sequence

with_sequence 被 loop 和 range 函数以及可能的 format 过滤器替换。

```
- name: with_sequence
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_sequence: start=0 end=4 stride=2 format=testuser%02x

- name: with_sequence -> loop
  ansible.builtin.debug:
    msg: "{{ 'testuser%02x' | format(item) }}"
  loop: "{{ range(0, 4 + 1, 2)|list }}"
```



循环的范围不包括终点。

### with_subelements

with_subelements 被 loop 和 subelements 过滤器替换。

```
- name: with_subelements
  ansible.builtin.debug:
    msg: "{{ item.0.name }} - {{ item.1 }}"
  with_subelements:
    - "{{ users }}"
    - mysql.hosts

- name: with_subelements -> loop
  ansible.builtin.debug:
    msg: "{{ item.0.name }} - {{ item.1 }}"
  loop: "{{ users|subelements('mysql.hosts') }}"
```



### with_nested/with_cartesian

with_nested 和 with_cartesian 被 loop 和 product 过滤器替换。

```
- name: with_nested
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_nested:
    - "{{ list_one }}"
    - "{{ list_two }}"

- name: with_nested -> loop
  ansible.builtin.debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ list_one|product(list_two)|list }}"
```



### with_random_choice

with_random_choice 被 random 过滤器的简单用法替换，无需使用 loop。

```
- name: with_random_choice
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_random_choice: "{{ my_list }}"

- name: with_random_choice -> loop (No loop is needed here)
  ansible.builtin.debug:
    msg: "{{ my_list|random }}"
  tags: random
```
