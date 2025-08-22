---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.2 操作数据
order: 1
---

# 操作数据

在许多情况下，您需要对变量执行复杂的操作。虽然不建议将 Ansible 用作数据处理/操作工具，但您可以结合使用现有的 Jinja2 模板和许多添加的 Ansible 过滤器、查找和测试来执行一些非常复杂的转换。

- 让我们从每个插件类型的快速定义开始

  查找（lookups）：主要用于查询“外部数据”，在 Ansible 中，它们是使用 with_<lookup> 结构进行循环的主要部分，但它们可以独立使用以返回数据进行处理。由于它们在循环中的主要功能（如前所述），它们通常返回一个列表。与 lookup 或 query Jinja2 操作符一起使用。过滤器（filters）：用于更改/转换数据，与 | Jinja2 操作符一起使用。测试（tests）：用于验证数据，与 is Jinja2 操作符一起使用。



## 循环和列表推导

大多数编程语言都有循环（for、while 等）和列表推导，以便对列表（包括对象列表）进行转换。Jinja2 有一些过滤器可以提供此功能：map、select、reject、selectattr、rejectattr。

- map：这是一个基本的 for 循环，它只允许您更改列表中的每个项目，使用 'attribute' 关键字，您可以根据列表元素的属性进行转换。
- select/reject：这是一个带有条件的 for 循环，它允许您创建一个基于条件结果匹配（或不匹配）的列表子集。
- selectattr/rejectattr：与上述非常相似，但它使用列表元素的特定属性进行条件语句。

使用循环创建指数退避。

```
- name: try wait_for_connection up to 10 times with exponential delay
  ansible.builtin.wait_for_connection:
    delay: '{{ item | int }}'
    timeout: 1
  loop: '{{ range(1, 11) | map("pow", 2) }}'
  loop_control:
    extended: true
  ignore_errors: "{{ not ansible_loop.last }}"
  register: result
  when: result is not defined or result is failed
```





### 从列表中匹配元素，并从字典中提取键

Python 等效代码将是

```
chains = [1, 2]
for chain in chains:
    for config in chains_config[chain]['configs']:
        print(config['type'])
```



在 Ansible 中有几种方法可以做到这一点，这只是一个例子

从字典列表中提取匹配键的方法

```
 tasks:
   - name: Show extracted list of keys from a list of dictionaries
     ansible.builtin.debug:
       msg: "{{ chains | map('extract', chains_config) | map(attribute='configs') | flatten | map(attribute='type') | flatten }}"
     vars:
       chains: [1, 2]
       chains_config:
           1:
               foo: bar
               configs:
                   - type: routed
                     version: 0.1
                   - type: bridged
                     version: 0.2
           2:
               foo: baz
               configs:
                   - type: routed
                     version: 1.0
                   - type: bridged
                     version: 1.1
```



调试任务的结果，一个包含提取键的列表

```
   ok: [localhost] => {
       "msg": [
           "routed",
           "bridged",
           "routed",
           "bridged"
       ]
   }
```



获取每个主机上不同的变量的唯一值列表

```
   vars:
       unique_value_list: "{{ groups['all'] | map ('extract', hostvars, 'varname') | list | unique}}"
```





### 查找挂载点

在这种情况下，我们希望找到机器上给定路径的挂载点，因为我们已经收集了挂载事实，我们可以使用以下方法

使用 selectattr 将挂载过滤到列表中，然后我可以排序并从中选择最后一个

```
  - hosts: all
    gather_facts: True
    vars:
       path: /var/lib/cache
    tasks:
    - name: The mount point for {{path}}, found using the Ansible mount facts, [-1] is the same as the 'last' filter
      ansible.builtin.debug:
       msg: "{{(ansible_facts.mounts | selectattr('mount', 'in', path) | list | sort(attribute='mount'))[-1]['mount']}}"
```





### 从列表中省略元素

特殊的 omit 变量仅适用于模块选项，但我们仍然可以将其用作标识符来定制元素列表

在输入模块选项时进行内联列表过滤

```
   - name: Enable a list of Windows features, by name
     ansible.builtin.set_fact:
       win_feature_list: "{{ namestuff | reject('equalto', omit) | list }}"
     vars:
       namestuff:
         - "{{ (fs_installed_smb_v1 | default(False)) | ternary(omit, 'FS-SMB1') }}"
         - "foo"
         - "bar"
```



另一种方法是避免首先向列表中添加元素，这样您就可以直接使用它

使用 set_fact 在循环中以条件方式递增列表

```
   - name: Build unique list with some items conditionally omitted
     ansible.builtin.set_fact:
        namestuff: ' {{ (namestuff | default([])) | union([item]) }}'
     when: item != omit
     loop:
         - "{{ (fs_installed_smb_v1 | default(False)) | ternary(omit, 'FS-SMB1') }}"
         - "foo"
         - "bar"
```





### 合并来自同一字典列表的值

结合上述示例中的肯定和否定过滤器，您可以获得“存在时的值”，以及不存在时的“回退”。

根据需要使用 selectattr 和 rejectattr 获取 ansible_host 或 inventory_hostname

```
   - hosts: localhost
     tasks:
       - name: Check hosts in inventory that respond to ssh port
         wait_for:
           host: "{{ item }}"
           port: 22
         loop: '{{ has_ah + no_ah }}'
         vars:
           has_ah: '{{ hostvars|dictsort|selectattr("1.ansible_host", "defined")|map(attribute="1.ansible_host")|list }}'
           no_ah: '{{ hostvars|dictsort|rejectattr("1.ansible_host", "defined")|map(attribute="0")|list }}'
```





### 基于变量的自定义 Fileglob

此示例使用 Python 参数列表解包来创建基于变量的自定义 fileglob 列表。

将 fileglob 与基于变量的列表一起使用。

```
  - hosts: all
    vars:
      mygroups:
        - prod
        - web
    tasks:
      - name: Copy a glob of files based on a list of groups
        copy:
          src: "{{ item }}"
          dest: "/tmp/{{ item }}"
        loop: '{{ q("fileglob", *globlist) }}'
        vars:
          globlist: '{{ mygroups | map("regex_replace", "^(.*)$", "files/\1/*.conf") | list }}'
```





## 复杂类型转换

Jinja 提供了用于简单数据类型转换的过滤器（int、bool 等），但是当您想要转换数据结构时，事情就没有那么容易了。您可以如上所示使用循环和列表推导来提供帮助，还可以链接和使用其他过滤器和查找来实现更复杂的转换。



### 从列表创建字典

在大多数语言中，从成对列表创建字典（也称为 map/关联数组/hash 等）很容易。在 Ansible 中，有几种方法可以做到这一点，哪种方法最适合您可能取决于您的数据来源。

这些示例产生 {"a": "b", "c": "d"}

假设列表为 [key, value , key, value, …]，则将简单列表转换为字典

```
 vars:
     single_list: [ 'a', 'b', 'c', 'd' ]
     mydict: "{{ dict(single_list[::2] | zip_longest(single_list[1::2])) }}"
```



当我们有一个成对的列表时，它更简单：

```
 vars:
     list_of_pairs: [ ['a', 'b'], ['c', 'd'] ]
     mydict: "{{ dict(list_of_pairs) }}"
```



两者最终都是相同的，其中 zip_longest 将 single_list 转换为 list_of_pairs 生成器。

稍微复杂一点，使用 set_fact 和 loop 从 2 个列表创建/更新具有键值对的字典

使用 set_fact 从一组列表创建字典

```
    - name: Uses 'combine' to update the dictionary and 'zip' to make pairs of both lists
      ansible.builtin.set_fact:
        mydict: "{{ mydict | default({}) | combine({item[0]: item[1]}) }}"
      loop: "{{ (keys | zip(values)) | list }}"
      vars:
        keys:
          - foo
          - var
          - bar
        values:
          - a
          - b
          - c
```



这会产生 {"foo": "a", "var": "b", "bar": "c"}。

您甚至可以将这些简单示例与其他过滤器和查找结合起来，通过将模式与变量名称匹配来动态创建字典

使用 'vars' 从一组列表定义字典，而无需任务

```
   vars:
       xyz_stuff: 1234
       xyz_morestuff: 567
       myvarnames: "{{ q('varnames', '^xyz_') }}"
       mydict: "{{ dict(myvarnames|map('regex_replace', '^xyz_', '')|list | zip(q('vars', *myvarnames))) }}"
```



快速解释一下，因为这两行代码有很多需要解释的地方

> - varnames 查找返回与“以 xyz_ 开头”匹配的变量列表。
> - 然后将上一步中的列表馈送到 vars 查找以获取值列表。* 用于“解引用列表”（Jinja 中可用的 pythonism），否则它将把列表作为单个参数。
> - 两个列表都传递给 zip 过滤器，以将它们配对到一个统一的列表中（key，value，key2，value2，…）。
> - 然后，dict 函数采用此“成对列表”来创建字典。

如何使用事实查找满足条件 X 的主机数据的示例

```
vars:
  uptime_of_host_most_recently_rebooted: "{{ansible_play_hosts_all | map('extract', hostvars, 'ansible_uptime_seconds') | sort | first}}"
```



显示主机正常运行时间（以天/小时/分钟/秒为单位）的示例（假设已收集事实）。

```
- name: Show the uptime in days/hours/minutes/seconds
  ansible.builtin.debug:
   msg: Uptime {{ now().replace(microsecond=0) - now().fromtimestamp(now(fmt='%s') | int - ansible_uptime_seconds) }}
```
