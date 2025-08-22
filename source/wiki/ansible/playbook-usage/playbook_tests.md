---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.5 语法测试
order: 4
---

# 语法测试

Jinja 中的测试是一种评估模板表达式并返回 True 或 False 的方法。Jinja 自带许多此类测试。请参阅官方 Jinja 模板文档中的内置测试。

测试和过滤器之间的主要区别在于，Jinja 测试用于比较，而过滤器用于数据操作，它们在 Jinja 中具有不同的应用。测试也可以用于列表处理过滤器，例如map()和select()，用于选择列表中的项目。

与所有模板一样，测试始终在 Ansible 控制节点上执行，而不是在任务的目标上执行，因为它们测试的是本地数据。

除了这些 Jinja2 测试之外，Ansible 还提供了一些其他的测试，用户也可以轻松创建自己的测试。

- 测试语法
- 字符串测试
- Vault
- 真值测试
- 版本比较
- 集合论测试
- 测试列表是否包含某个值
- 测试列表值是否为 True
- 路径测试
- 大小格式测试
  - 人类可读
  - 人类可读字节
- 测试任务结果
- 类型测试



## 测试语法

测试语法与过滤器语法(variable | filter)不同。历史上，Ansible 将测试注册为 Jinja 测试和 Jinja 过滤器，允许使用过滤器语法引用它们。

从 Ansible 2.5 开始，将 Jinja 测试用作过滤器将生成弃用警告。从 Ansible 2.9 开始，必须使用 Jinja 测试语法。

使用 Jinja 测试的语法如下：

```
variable is test_name
```



例如：

```
result is failed
```





## 字符串测试

要将字符串与子字符串或正则表达式进行匹配，可以使用match、search或regex测试。

```
vars:
  url: "https://example.com/users/foo/resources/bar"

tasks:
    - debug:
        msg: "matched pattern 1"
      when: url is match("https://example.com/users/.*/resources")

    - debug:
        msg: "matched pattern 2"
      when: url is search("users/.*/resources/.*")

    - debug:
        msg: "matched pattern 3"
      when: url is search("users")

    - debug:
        msg: "matched pattern 4"
      when: url is regex("example\.com/\w+/foo")
```



match如果在字符串开头找到模式，则成功，而search如果在字符串中任意位置找到模式，则成功。regex默认情况下与search类似，但是可以通过传递match_type关键字参数来配置regex以执行其他测试。特别是，match_type确定用于执行搜索的re方法。完整的列表可以在相关的 Python 文档此处找到。

所有字符串测试还都接受可选的ignorecase和multiline参数。这些分别对应于 Python 的re库中的re.I和re.M。



## Vault

2.10 版本新增。

可以使用vault_encrypted测试来测试变量是否为内联单个 Vault 加密值。

```
vars:
  variable: !vault |
    $ANSIBLE_VAULT;1.2;AES256;dev
    61323931353866666336306139373937316366366138656131323863373866376666353364373761
    3539633234313836346435323766306164626134376564330a373530313635343535343133316133
    36643666306434616266376434363239346433643238336464643566386135356334303736353136
    6565633133366366360a326566323363363936613664616364623437336130623133343530333739
    3039

tasks:
  - debug:
      msg: '{{ (variable is vault_encrypted) | ternary("Vault encrypted", "Not vault encrypted") }}'
```





## 真值测试

2.10 版本新增。

从 Ansible 2.10 开始，您现在可以执行类似 Python 的真值和假值检查。

```
- debug:
    msg: "Truthy"
  when: value is truthy
  vars:
    value: "some string"

- debug:
    msg: "Falsy"
  when: value is falsy
  vars:
    value: ""
```



此外，truthy和falsy测试接受一个名为convert_bool的可选参数，该参数将尝试将布尔指示符转换为实际布尔值。

```
- debug:
    msg: "Truthy"
  when: value is truthy(convert_bool=True)
  vars:
    value: "yes"

- debug:
    msg: "Falsy"
  when: value is falsy(convert_bool=True)
  vars:
    value: "off"
```





## 版本比较

1.6 版本新增。

要比较版本号，例如检查ansible_facts['distribution_version']版本是否大于或等于“12.04”，可以使用version测试。

version测试也可以用于评估ansible_facts['distribution_version']。

```
{{ ansible_facts['distribution_version'] is version('12.04', '>=') }}
```



如果ansible_facts['distribution_version']大于或等于 12.04，则此测试返回 True，否则返回 False。

version测试接受以下运算符：

```
<, lt, <=, le, >, gt, >=, ge, ==, =, eq, !=, <>, ne
```



此测试还接受第三个参数strict，它定义是否应使用ansible.module_utils.compat.version.StrictVersion定义的严格版本解析。默认值为False（使用ansible.module_utils.compat.version.LooseVersion），True启用严格版本解析。

```
{{ sample_version_var is version('1.0', operator='lt', strict=True) }}
```



从 Ansible 2.11 开始，version测试接受一个version_type参数，该参数与strict互斥，并接受以下值：

```
loose, strict, semver, semantic, pep440
```



- 宽松

  此类型对应于 Python distutils.version.LooseVersion类。所有版本格式对此类型都有效。比较规则简单且可预测，但可能并不总是给出预期的结果。

- 严格

  此类型对应于 Python distutils.version.StrictVersion类。版本号由两个或三个点分隔的数字组件组成，末尾可选“预发布”标签。预发布标签由单个字母“a”或“b”后跟一个数字组成。如果两个版本号的数字组件相等，则带有预发布标签的版本号将始终被认为早于（小于）没有预发布标签的版本号。

- semver/semantic

  此类型实现了语义版本方案进行版本比较。

- pep440

  此类型实现 Python PEP-440 版本规则进行版本比较。在 2.14 版本中添加。

使用 version_type 比较语义版本，方法如下所示

```
{{ sample_semver_var is version('2.0.0-rc.1+build.123', 'lt', version_type='semver') }}
```



在 Ansible 2.14 中，添加了 version_type 的 pep440 选项，此类型的规则在 PEP-440 中定义。以下示例展示了此类型如何区分预发布版本小于正式版本。

```
{{ '2.14.0rc1' is version('2.14.0', 'lt', version_type='pep440') }}
```



在剧本或角色中使用 version 时，请勿使用 {{ }}，如 FAQ 中所述。

```
vars:
    my_version: 1.2.3

tasks:
    - debug:
        msg: "my_version is higher than 1.0.0"
      when: my_version is version('1.0.0', '>')
```





## 集合论测试

2.1 版本新增。

要查看列表是否包含或被另一个列表包含，可以使用“subset”和“superset”。

```
vars:
    a: [1,2,3,4,5]
    b: [2,3]
tasks:
    - debug:
        msg: "A includes B"
      when: a is superset(b)

    - debug:
        msg: "B is included in A"
      when: b is subset(a)
```





## 测试列表是否包含值

2.8 版本新增。

Ansible 包含一个 contains 测试，其操作方式类似，但与 Jinja2 提供的 in 测试相反。contains 测试旨在与 select、reject、selectattr 和 rejectattr 过滤器一起使用。

```
vars:
  lacp_groups:
    - master: lacp0
      network: 10.65.100.0/24
      gateway: 10.65.100.1
      dns4:
        - 10.65.100.10
        - 10.65.100.11
      interfaces:
        - em1
        - em2

    - master: lacp1
      network: 10.65.120.0/24
      gateway: 10.65.120.1
      dns4:
        - 10.65.100.10
        - 10.65.100.11
      interfaces:
          - em3
          - em4

tasks:
  - debug:
      msg: "{{ (lacp_groups|selectattr('interfaces', 'contains', 'em1')|first).master }}"
```



## 测试列表值是否为 True

2.4 版本新增。

可以使用 any 和 all 检查列表中任何或所有元素是否为真。

```
vars:
  mylist:
      - 1
      - "{{ 3 == 3 }}"
      - True
  myotherlist:
      - False
      - True
tasks:

  - debug:
      msg: "all are true!"
    when: mylist is all

  - debug:
      msg: "at least one is true"
    when: myotherlist is any
```





## 测试路径

以下测试可以提供有关控制节点上路径的信息。

```
- debug:
    msg: "path is a directory"
  when: mypath is directory

- debug:
    msg: "path is a file"
  when: mypath is file

- debug:
    msg: "path is a symlink"
  when: mypath is link

- debug:
    msg: "path already exists"
  when: mypath is exists

- debug:
    msg: "path is {{ (mypath is abs)|ternary('absolute','relative')}}"

- debug:
    msg: "path is the same file as path2"
  when: mypath is same_file(path2)

- debug:
    msg: "path is a mount"
  when: mypath is mount

- debug:
    msg: "path is a directory"
  when: mypath is directory
  vars:
     mypath: /my/path

- debug:
    msg: "path is a file"
  when: "'/my/path' is file"
```



## 测试大小格式

human_readable 和 human_to_bytes 函数允许您测试您的剧本，以确保您在任务中使用正确的尺寸格式，并向计算机提供字节格式，向人们提供人类可读的格式。

### 人类可读

断言给定字符串是否为人可读。

例如

```
- name: "Human Readable"
  assert:
    that:
      - '"1.00 Bytes" == 1|human_readable'
      - '"1.00 bits" == 1|human_readable(isbits=True)'
      - '"10.00 KB" == 10240|human_readable'
      - '"97.66 MB" == 102400000|human_readable'
      - '"0.10 GB" == 102400000|human_readable(unit="G")'
      - '"0.10 Gb" == 102400000|human_readable(isbits=True, unit="G")'
```



这将导致

```
{ "changed": false, "msg": "All assertions passed" }
```



### 人类可读转换为字节

以字节格式返回给定字符串。

例如

```
- name: "Human to Bytes"
  assert:
    that:
      - "{{'0'|human_to_bytes}}        == 0"
      - "{{'0.1'|human_to_bytes}}      == 0"
      - "{{'0.9'|human_to_bytes}}      == 1"
      - "{{'1'|human_to_bytes}}        == 1"
      - "{{'10.00 KB'|human_to_bytes}} == 10240"
      - "{{   '11 MB'|human_to_bytes}} == 11534336"
      - "{{  '1.1 GB'|human_to_bytes}} == 1181116006"
      - "{{'10.00 Kb'|human_to_bytes(isbits=True)}} == 10240"
```



这将导致

```
{ "changed": false, "msg": "All assertions passed" }
```





## 测试任务结果

以下任务说明了用于检查任务状态的测试。

```
tasks:

  - shell: /usr/bin/foo
    register: result
    ignore_errors: True

  - debug:
      msg: "it failed"
    when: result is failed

  # in most cases you'll want a handler, but if you want to do something right now, this is nice
  - debug:
      msg: "it changed"
    when: result is changed

  - debug:
      msg: "it succeeded in Ansible >= 2.1"
    when: result is succeeded

  - debug:
      msg: "it succeeded"
    when: result is success

  - debug:
      msg: "it was skipped"
    when: result is skipped
```





## 类型测试

当需要确定类型时，可能很想使用 type_debug 过滤器并将其与该类型的字符串名称进行比较，但是，您应该改用类型测试比较，例如

```
tasks:
  - name: "String interpretation"
    vars:
      a_string: "A string"
      a_dictionary: {"a": "dictionary"}
      a_list: ["a", "list"]
    assert:
      that:
      # Note that a string is classed as also being "iterable" and "sequence", but not "mapping"
      - a_string is string and a_string is iterable and a_string is sequence and a_string is not mapping

      # Note that a dictionary is classed as not being a "string", but is "iterable", "sequence" and "mapping"
      - a_dictionary is not string and a_dictionary is iterable and a_dictionary is mapping

      # Note that a list is classed as not being a "string" or "mapping" but is "iterable" and "sequence"
      - a_list is not string and a_list is not mapping and a_list is iterable

  - name: "Number interpretation"
    vars:
      a_float: 1.01
      a_float_as_string: "1.01"
      an_integer: 1
      an_integer_as_string: "1"
    assert:
      that:
      # Both a_float and an_integer are "number", but each has their own type as well
      - a_float is number and a_float is float
      - an_integer is number and an_integer is integer

      # Both a_float_as_string and an_integer_as_string are not numbers
      - a_float_as_string is not number and a_float_as_string is string
      - an_integer_as_string is not number and a_float_as_string is string

      # a_float or a_float_as_string when cast to a float and then to a string should match the same value cast only to a string
      - a_float | float | string == a_float | string
      - a_float_as_string | float | string == a_float_as_string | string

      # Likewise an_integer and an_integer_as_string when cast to an integer and then to a string should match the same value cast only to an integer
      - an_integer | int | string == an_integer | string
      - an_integer_as_string | int | string == an_integer_as_string | string

      # However, a_float or a_float_as_string cast as an integer and then a string does not match the same value cast to a string
      - a_float | int | string != a_float | string
      - a_float_as_string | int | string != a_float_as_string | string

      # Again, Likewise an_integer and an_integer_as_string cast as a float and then a string does not match the same value cast to a string
      - an_integer | float | string != an_integer | string
      - an_integer_as_string | float | string != an_integer_as_string | string

  - name: "Native Boolean interpretation"
    loop:
    - yes
    - true
    - True
    - TRUE
    - no
    - No
    - NO
    - false
    - False
    - FALSE
    assert:
      that:
      # Note that while other values may be cast to boolean values, these are the only ones that are natively considered boolean
      # Note also that `yes` is the only case-sensitive variant of these values.
      - item is boolean
```
