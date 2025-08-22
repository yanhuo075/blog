---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.3 模板引擎 (Jinja2)
order: 2
---

# 模板引擎 (Jinja2)

Ansible 使用 Jinja2 模板引擎实现动态表达式，并访问变量和事实。您可以使用template 模块进行模板化。例如，您可以为配置文件创建模板，然后将该配置文件部署到多个环境，并为每个环境提供正确的数据（IP 地址、主机名、版本）。您也可以直接在 playbook 中使用模板，例如对任务名称进行模板化等等。您可以使用Jinja2中包含的所有标准过滤器和测试。Ansible 包含其他专门用于选择和转换数据的过滤器、用于评估模板表达式的测试以及查找插件，用于从外部来源（例如文件、API 和数据库）检索数据以用于模板化。

所有模板化操作都在 Ansible 控制节点上完成，在任务发送到目标机器并执行之前。这种方法最大限度地减少了对目标机器的包要求（只有控制节点需要 jinja2）。它还限制了 Ansible 传递到目标机器的数据量。Ansible 在控制节点上解析模板，并且只将每个任务所需的信息传递到目标机器，而不是将控制节点上的所有数据都传递过去并在目标机器上进行解析。

## Jinja2 示例

在这个例子中，我们想将服务器主机名写入其 /tmp/hostname。

我们的目录如下所示

```
├── hostname.yml
├── templates
    └── test.j2
```



我们的 hostname.yml

```
---
- name: Write hostname
  hosts: all
  tasks:
  - name: write hostname using jinja2
    ansible.builtin.template:
       src: templates/test.j2
       dest: /tmp/hostname
```



我们的 test.j2

```
My name is {{ ansible_facts['hostname'] }}
```



## 模板中的 Python3

Ansible 使用 Jinja2 在模板和变量中利用 Python 数据类型和标准函数。您可以使用这些数据类型和标准函数对数据执行丰富的操作。但是，如果您使用模板，则必须了解 Python 版本之间的差异。

这些主题可帮助您设计可在 Python2 和 Python3 上运行的模板。如果您从 Python2 升级到 Python3，它们也可能有所帮助。在 Python2 或 Python3 中升级通常不会引入影响 Jinja2 模板的更改。



### 字典视图

在 Python2 中，dict.keys()、dict.values() 和 dict.items() 方法返回一个列表。Jinja2 使用 Ansible 可以转换回列表的字符串表示形式将该列表返回给 Ansible。

在 Python3 中，这些方法返回一个字典视图 对象。Jinja2 返回的字典视图的字符串表示形式无法被 Ansible 解析回列表。但是，通过在使用 dict.keys()、dict.values() 或 dict.items() 时使用 list 过滤器，可以轻松实现其可移植性。

```
vars:
  hosts:
    testhost1: 127.0.0.2
    testhost2: 127.0.0.3
tasks:
  - debug:
      msg: '{{ item }}'
    # Only works with Python 2
    #loop: "{{ hosts.keys() }}"
    # Works with both Python 2 and Python 3
    loop: "{{ hosts.keys() | list }}"
```



### dict.iteritems()

Python2 字典具有 iterkeys()、itervalues() 和 iteritems() 方法。

Python3 字典没有这些方法。使用 dict.keys()、dict.values() 和 dict.items() 使您的 playbook 和模板与 Python2 和 Python3 都兼容。

```
vars:
  hosts:
    testhost1: 127.0.0.2
    testhost2: 127.0.0.3
tasks:
  - debug:
      msg: '{{ item }}'
    # Only works with Python 2
    #loop: "{{ hosts.iteritems() }}"
    # Works with both Python 2 and Python 3
    loop: "{{ hosts.items() | list }}"
```



## now 函数：获取当前时间

2.8 版本新增。

now() Jinja2 函数检索 Python datetime 对象或当前时间的字符串表示形式。

now() 函数支持两个参数

- utc

  指定 True 以获取 UTC 当前时间。默认为 False。

- fmt

  接受一个 strftime 字符串，该字符串返回格式化的日期时间字符串。

例如：dtg: "Current time (UTC): {{ now(utc=true,fmt='%Y-%m-%d %H:%M:%S') }}"



## undef 函数：为未定义变量添加提示

2.12 版本新增。

Jinja2 的 undef() 函数返回一个 Python AnsibleUndefined 对象，该对象派生自 jinja2.StrictUndefined。使用 undef() 来取消定义优先级较低的变量。例如，可以为一个任务块覆盖主机变量

```
---
- hosts: localhost
  gather_facts: no
  module_defaults:
    group/ns.col.auth: "{{ vaulted_credentials | default({}) }}"
  tasks:
    - ns.col.module1:
    - ns.col.module2:

    - name: override host variable
      vars:
        vaulted_credentials: "{{ undef() }}"
      block:
        - ns.col.module1:
```



undef 函数接受一个可选参数

- hint

  如果 DEFAULT_UNDEFINED_VAR_BEHAVIOR 配置为给出错误，则提供有关未定义变量的自定义提示。
