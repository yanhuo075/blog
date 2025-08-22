---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.4 使用过滤器处理数据
order: 3
---

# 使用过滤器处理数据

过滤器允许您将JSON数据转换为YAML数据，分割URL以提取主机名，获取字符串的SHA1哈希值，添加或乘以整数等等。您可以使用此处记录的Ansible特定过滤器来处理您的数据，或者使用Jinja2附带的任何标准过滤器 - 请参阅官方Jinja2模板文档中内置过滤器列表。您还可以使用Python方法来转换数据。您可以创建自定义Ansible过滤器作为插件，尽管我们通常欢迎将新的过滤器添加到ansible-core存储库中，以便每个人都可以使用它们。

因为模板化发生在Ansible控制节点上，而不是目标主机上，所以过滤器在控制节点上执行并在本地转换数据。

- 处理未定义变量
  - 提供默认值
  - 使变量可选
  - 定义必填值
- 定义true/false/null的不同值（三元运算）
- 管理数据类型
  - 发现数据类型
  - 将字符串转换为列表
  - 将字典转换为列表
  - 将列表转换为字典
  - 强制数据类型
- 数据格式化：YAML和JSON
  - 过滤器to_json和Unicode支持
- 组合和选择数据
  - 组合来自多个列表的项目：zip和zip_longest
  - 组合对象和子元素
  - 组合哈希表/字典
  - 从数组或哈希表中选择值
  - 组合列表
    - 排列
    - 组合
    - 乘积
  - 选择JSON数据：JSON查询
- 随机化数据
  - 随机MAC地址
  - 随机项目或数字
  - 洗牌列表
- 管理列表变量
- 从集合或列表中选择（集合论）
- 计算数字（数学）
- 管理网络交互
  - IP地址过滤器
  - 网络CLI过滤器
  - 网络XML过滤器
  - 网络VLAN过滤器
- 散列和加密字符串和密码
- 处理文本
  - 向文件添加注释
  - URLEncode变量
  - 分割URL
  - 使用正则表达式搜索字符串
  - 管理文件名和路径名
- 处理字符串
- 管理UUID
- 处理日期和时间
- 获取Kubernetes资源名称

## 处理未定义变量

过滤器可以通过提供默认值或使某些变量可选来帮助您管理丢失或未定义的变量。如果您将Ansible配置为忽略大多数未定义变量，则可以使用mandatory过滤器将某些变量标记为需要值。



### 提供默认值

您可以使用Jinja2的“default”过滤器直接在模板中为变量提供默认值。这通常比在未定义变量时失败更好的方法。

```
{{ some_variable | default(5) }}
```



在上面的示例中，如果变量“some_variable”未定义，Ansible将使用默认值5，而不是引发“未定义变量”错误并失败。如果您在角色中工作，还可以添加角色默认值来定义角色中变量的默认值。要了解有关角色默认值的更多信息，请参阅角色目录结构。

从2.8版本开始，尝试访问Jinja中Undefined值的属性将返回另一个Undefined值，而不是立即抛出错误。这意味着您现在可以简单地在嵌套数据结构中使用带有值的默认值（换句话说，{{ foo.bar.baz | default('DEFAULT') }}），而无需知道中间值是否已定义。

如果要在变量计算结果为false或空字符串时使用默认值，则必须将第二个参数设置为true

```
{{ lookup('env', 'MY_USER') | default('admin', true) }}
```





### 使变量可选

默认情况下，Ansible要求模板表达式中的所有变量都具有值。但是，您可以使特定模块变量可选。例如，您可能希望使用系统默认值来处理一些项目并控制其他项目的值。要使模块变量可选，请将默认值设置为特殊变量omit

```
- name: Touch files with an optional mode
  ansible.builtin.file:
    dest: "{{ item.path }}"
    state: touch
    mode: "{{ item.mode | default(omit) }}"
  loop:
    - path: /tmp/foo
    - path: /tmp/bar
    - path: /tmp/baz
      mode: "0444"
```



在此示例中，文件/tmp/foo和/tmp/bar的默认模式由系统的umask决定。Ansible不会为mode发送值。只有第三个文件/tmp/baz收到mode=0444选项。



### 定义必填值

如果您将Ansible配置为忽略未定义的变量，则可能需要将某些值定义为必填值。默认情况下，如果您的playbook或命令中的变量未定义，Ansible将失败。您可以通过将DEFAULT_UNDEFINED_VAR_BEHAVIOR设置为false来配置Ansible以允许未定义的变量。在这种情况下，您可能希望要求定义某些变量。您可以使用

```
{{ variable | mandatory }}
```



变量值将按原样使用，但如果变量未定义，则模板评估将引发错误。

一种方便的需要覆盖变量的方法是使用undef()函数为其提供未定义的值。

```
galaxy_url: "https://galaxy.ansible.com"
galaxy_api_key: "{{ undef(hint='You must specify your Galaxy API key') }}"
```



## 定义true/false/null的不同值（三元运算）

您可以创建一个测试，然后定义一个值在测试返回true时使用，另一个值在测试返回false时使用（1.9版新增）。

```
{{ (status == 'needs_restart') | ternary('restart', 'continue') }}
```



此外，您可以定义一个值用于true，一个值用于false，以及一个值用于null（2.8版新增）。

```
{{ enabled | ternary('no shutdown', 'shutdown', omit) }}
```



## 管理数据类型

您可能需要了解、更改或设置变量的数据类型。例如，已注册的变量在您的下一个任务需要列表时可能包含字典，或者用户提示在您的剧本需要布尔值时可能返回字符串。使用ansible.builtin.type_debug、ansible.builtin.dict2items和ansible.builtin.items2dict过滤器来管理数据类型。您也可以使用数据类型本身将值强制转换为特定数据类型。

### 发现数据类型

2.3 版本新增。

如果您不确定变量的底层 Python 类型，可以使用ansible.builtin.type_debug过滤器来显示它。这在需要特定类型变量的调试时非常有用。

```
{{ myvar | type_debug }}
```



您应该注意，虽然这看起来像是一个用于检查变量中是否具有正确数据类型的有用过滤器，但您通常应该更倾向于使用类型测试，它允许您测试特定数据类型。

### 将字符串转换为列表

使用ansible.builtin.split过滤器将字符/字符串分隔的字符串转换为适合循环的项目列表。例如，如果您想用逗号分割字符串变量fruits，您可以使用：

```
{{ fruits | split(',') }}
```



字符串数据（应用ansible.builtin.split过滤器之前）

```
fruits: apple,banana,orange
```



列表数据（应用ansible.builtin.split过滤器之后）

```
- apple
- banana
- orange
```





### 将字典转换为列表

2.6 版本新增。

使用ansible.builtin.dict2items过滤器将字典转换为适合循环的项目列表。

```
{{ dict | dict2items }}
```



字典数据（应用ansible.builtin.dict2items过滤器之前）

```
tags:
  Application: payment
  Environment: dev
```



列表数据（应用ansible.builtin.dict2items过滤器之后）

```
- key: Application
  value: payment
- key: Environment
  value: dev
```



2.8 版本新增。

ansible.builtin.dict2items过滤器是ansible.builtin.items2dict过滤器的反向操作。

如果您想配置键的名称，ansible.builtin.dict2items过滤器接受2个关键字参数。传递**[key_name](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/dict2items_filter.html#ansible-collections-ansible-builtin-dict2items-filter-parameter-key-name)**和**[value_name](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/dict2items_filter.html#ansible-collections-ansible-builtin-dict2items-filter-parameter-value-name)**参数来配置列表输出中键的名称。

```
{{ files | dict2items(key_name='file', value_name='path') }}
```



字典数据（应用ansible.builtin.dict2items过滤器之前）

```
files:
  users: /etc/passwd
  groups: /etc/group
```



列表数据（应用ansible.builtin.dict2items过滤器之后）

```
- file: users
  path: /etc/passwd
- file: groups
  path: /etc/group
```



### 将列表转换为字典

2.7 版本新增。

使用ansible.builtin.items2dict过滤器将列表转换为字典，并将内容映射到key: value对。

```
{{ tags | items2dict }}
```



列表数据（应用ansible.builtin.items2dict过滤器之前）

```
tags:
  - key: Application
    value: payment
  - key: Environment
    value: dev
```



字典数据（应用ansible.builtin.items2dict过滤器之后）

```
Application: payment
Environment: dev
```



ansible.builtin.items2dict过滤器是ansible.builtin.dict2items过滤器的反向操作。

并非所有列表都使用key来指定键和value来指定值。例如：

```
fruits:
  - fruit: apple
    color: red
  - fruit: pear
    color: yellow
  - fruit: grapefruit
    color: yellow
```



在此示例中，您必须传递**[key_name](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/items2dict_filter.html#ansible-collections-ansible-builtin-items2dict-filter-parameter-key-name)**和**[value_name](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/items2dict_filter.html#ansible-collections-ansible-builtin-items2dict-filter-parameter-value-name)**参数来配置转换。例如：

```
{{ fruits | items2dict(key_name='fruit', value_name='color') }}
```



如果您不传递这些参数，或者没有为您的列表传递正确的值，您将看到KeyError: key或KeyError: my_typo。

### 强制数据类型

您可以将值强制转换为某些类型。例如，如果您期望从vars_prompt输入“True”，并且您希望 Ansible 将其识别为布尔值而不是字符串：

```
- ansible.builtin.debug:
     msg: test
  when: some_string_value | bool
```



如果您想对事实进行数学比较，并且您希望 Ansible 将其识别为整数而不是字符串：

```
- shell: echo "only on Red Hat 6, derivatives, and later"
  when: ansible_facts['os_family'] == "RedHat" and ansible_facts['lsb']['major_release'] | int >= 6
```



1.6 版本新增。



## 数据格式：YAML 和 JSON

您可以使用模板将数据结构在 JSON 或 YAML 格式之间切换，并提供格式化、缩进和加载数据的选项。这些基本过滤器偶尔可用于调试。

```
{{ some_variable | to_json }}
{{ some_variable | to_yaml }}
```



有关这些过滤器的文档，请参阅ansible.builtin.to_json和ansible.builtin.to_yaml。

对于易于阅读的输出，您可以使用：

```
{{ some_variable | to_nice_json }}
{{ some_variable | to_nice_yaml }}
```



有关这些过滤器的文档，请参阅ansible.builtin.to_nice_json和ansible.builtin.to_nice_yaml。

您可以更改任一格式的缩进：

```
{{ some_variable | to_nice_json(indent=2) }}
{{ some_variable | to_nice_yaml(indent=8) }}
```



ansible.builtin.to_yaml和ansible.builtin.to_nice_yaml过滤器使用PyYAML 库，该库具有默认的 80 个字符字符串长度限制。这会导致在第 80 个字符后出现意外换行符（如果第 80 个字符后有空格）。为了避免这种行为并生成长行，请使用**width**选项。您必须使用硬编码数字来定义宽度，而不是像float("inf")这样的结构，因为过滤器不支持代理 Python 函数。例如：

```
{{ some_variable | to_yaml(indent=8, width=1337) }}
{{ some_variable | to_nice_yaml(indent=8, width=1337) }}
```



该过滤器支持传递其他 YAML 参数。有关完整列表，请参阅PyYAML 文档中的dump()。

如果您正在读取一些已格式化的数据：

```
{{ some_variable | from_json }}
{{ some_variable | from_yaml }}
```



例如：

```
tasks:
  - name: Register JSON output as a variable
    ansible.builtin.shell: cat /some/path/to/file.json
    register: result

  - name: Set a variable
    ansible.builtin.set_fact:
      myvar: "{{ result.stdout | from_json }}"
```



### 过滤器to_json和Unicode支持

默认情况下，ansible.builtin.to_json和ansible.builtin.to_nice_json会将接收到的数据转换为 ASCII，因此：

```
{{ 'München'| to_json }}
```



将返回：

```
'M\u00fcnchen'
```



要保留 Unicode 字符，请将参数ensure_ascii=False传递给过滤器。

```
{{ 'München'| to_json(ensure_ascii=False) }}

'München'
```



2.7 版本新增。

为了解析多文档 YAML 字符串，提供了ansible.builtin.from_yaml_all过滤器。ansible.builtin.from_yaml_all过滤器将返回解析的 YAML 文档的生成器。

例如：

```
tasks:
  - name: Register a file content as a variable
    ansible.builtin.shell: cat /some/path/to/multidoc-file.yaml
    register: result

  - name: Print the transformed variable
    ansible.builtin.debug:
      msg: '{{ item }}'
    loop: '{{ result.stdout | from_yaml_all | list }}'
```



## 组合和选择数据

您可以组合来自多个来源和类型的數據，并从大型数据结构中选择值，从而精确控制复杂数据。



### 组合来自多个列表的项目：zip 和 zip_longest

2.3 版本新增。

要获取组合其他列表元素的列表，请使用ansible.builtin.zip

```
- name: Give me list combo of two lists
  ansible.builtin.debug:
    msg: "{{ [1,2,3,4,5,6] | zip(['a','b','c','d','e','f']) | list }}"

# => [[1, "a"], [2, "b"], [3, "c"], [4, "d"], [5, "e"], [6, "f"]]

- name: Give me the shortest combo of two lists
  ansible.builtin.debug:
    msg: "{{ [1,2,3] | zip(['a','b','c','d','e','f']) | list }}"

# => [[1, "a"], [2, "b"], [3, "c"]]
```



要始终遍历所有列表，请使用ansible.builtin.zip_longest

```
- name: Give me the longest combo of three lists, fill with X
  ansible.builtin.debug:
    msg: "{{ [1,2,3] | zip_longest(['a','b','c','d','e','f'], [21, 22, 23], fillvalue='X') | list }}"

# => [[1, "a", 21], [2, "b", 22], [3, "c", 23], ["X", "d", "X"], ["X", "e", "X"], ["X", "f", "X"]]
```



与上面提到的ansible.builtin.items2dict 过滤器输出类似，这些过滤器可用于构建dict

```
{{ dict(keys_list | zip(values_list)) }}
```



列表数据（应用ansible.builtin.zip 过滤器之前）

```
keys_list:
  - one
  - two
values_list:
  - apple
  - orange
```



字典数据（应用ansible.builtin.zip 过滤器之后）

```
one: apple
two: orange
```



### 组合对象和子元素

2.7 版本新增。

ansible.builtin.subelements 过滤器生成对象及其子元素值的乘积，类似于ansible.builtin.subelements 查询。这允许您指定要在模板中使用的各个子元素。例如，此表达式

```
{{ users | subelements('groups', skip_missing=True) }}
```



应用ansible.builtin.subelements 过滤器之前的數據

```
users:
- name: alice
  authorized:
  - /tmp/alice/onekey.pub
  - /tmp/alice/twokey.pub
  groups:
  - wheel
  - docker
- name: bob
  authorized:
  - /tmp/bob/id_rsa.pub
  groups:
  - docker
```



应用ansible.builtin.subelements 过滤器之后的數據

```
-
  - name: alice
    groups:
    - wheel
    - docker
    authorized:
    - /tmp/alice/onekey.pub
    - /tmp/alice/twokey.pub
  - wheel
-
  - name: alice
    groups:
    - wheel
    - docker
    authorized:
    - /tmp/alice/onekey.pub
    - /tmp/alice/twokey.pub
  - docker
-
  - name: bob
    authorized:
    - /tmp/bob/id_rsa.pub
    groups:
    - docker
  - docker
```



您可以将转换后的数据与loop一起使用，以迭代多个对象的相同子元素

```
- name: Set authorized ssh key, extracting just that data from 'users'
  ansible.posix.authorized_key:
    user: "{{ item.0.name }}"
    key: "{{ lookup('file', item.1) }}"
  loop: "{{ users | subelements('authorized') }}"
```





### 组合哈希/字典

2.0 版本中的新功能。

ansible.builtin.combine 过滤器允许合并哈希。例如，以下操作将覆盖一个哈希中的键

```
{{ {'a':1, 'b':2} | combine({'b':3}) }}
```



生成的哈希将是

```
{'a':1, 'b':3}
```



过滤器还可以接受多个参数进行合并

```
{{ a | combine(b, c, d) }}
{{ [a, b, c, d] | combine }}
```



在这种情况下，d中的键将覆盖c中的键，后者将覆盖b中的键，依此类推。

该过滤器还接受两个可选参数：**[recursive](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/combine_filter.html#ansible-collections-ansible-builtin-combine-filter-parameter-recursive)** 和 **[list_merge](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/combine_filter.html#ansible-collections-ansible-builtin-combine-filter-parameter-list-merge)**。

- recursive

  是一个布尔值，默认为False。ansible.builtin.combine 是否应递归合并嵌套哈希。注意：它不依赖于ansible.cfg中hash_behaviour设置的值。

- list_merge

  是一个字符串，其可能的值为replace（默认）、keep、append、prepend、append_rp 或 prepend_rp。当要合并的哈希包含数组/列表时，它会修改ansible.builtin.combine 的行为。

```
default:
  a:
    x: default
    y: default
  b: default
  c: default
patch:
  a:
    y: patch
    z: patch
  b: patch
```



如果recursive=False（默认值），则不会合并嵌套哈希

```
{{ default | combine(patch) }}
```



这将导致

```
a:
  y: patch
  z: patch
b: patch
c: default
```



如果recursive=True，则递归进入嵌套哈希并合并它们的键

```
{{ default | combine(patch, recursive=True) }}
```



这将导致

```
a:
  x: default
  y: patch
  z: patch
b: patch
c: default
```



如果list_merge='replace'（默认值），则右侧哈希中的数组将“替换”左侧哈希中的数组

```
default:
  a:
    - default
patch:
  a:
    - patch
```



```
{{ default | combine(patch) }}
```



这将导致

```
a:
  - patch
```



如果list_merge='keep'，则将保留左侧哈希中的数组

```
{{ default | combine(patch, list_merge='keep') }}
```



这将导致

```
a:
  - default
```



如果list_merge='append'，则右侧哈希中的数组将附加到左侧哈希中的数组

```
{{ default | combine(patch, list_merge='append') }}
```



这将导致

```
a:
  - default
  - patch
```



如果list_merge='prepend'，则右侧哈希中的数组将添加到左侧哈希中的数组前面

```
{{ default | combine(patch, list_merge='prepend') }}
```



这将导致

```
a:
  - patch
  - default
```



如果list_merge='append_rp'，则右侧哈希中的数组将附加到左侧哈希中的数组。左侧哈希中也出现在右侧哈希相应数组中的数组元素将被删除（“rp”代表“remove present”）。哈希中都不存在的重复元素将被保留

```
default:
  a:
    - 1
    - 1
    - 2
    - 3
patch:
  a:
    - 3
    - 4
    - 5
    - 5
```



```
{{ default | combine(patch, list_merge='append_rp') }}
```



这将导致

```
a:
  - 1
  - 1
  - 2
  - 3
  - 4
  - 5
  - 5
```



如果list_merge='prepend_rp'，则行为类似于append_rp，但右侧哈希中的数组元素将被添加到前面

```
{{ default | combine(patch, list_merge='prepend_rp') }}
```



这将导致

```
a:
  - 3
  - 4
  - 5
  - 5
  - 1
  - 1
  - 2
```



**[recursive](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/combine_filter.html#ansible-collections-ansible-builtin-combine-filter-parameter-recursive)** 和 **[list_merge](https://docs.ansible.org.cn/ansible/latest/collections/ansible/builtin/combine_filter.html#ansible-collections-ansible-builtin-combine-filter-parameter-list-merge)** 可以一起使用

```
default:
  a:
    a':
      x: default_value
      y: default_value
      list:
        - default_value
  b:
    - 1
    - 1
    - 2
    - 3
patch:
  a:
    a':
      y: patch_value
      z: patch_value
      list:
        - patch_value
  b:
    - 3
    - 4
    - 4
    - key: value
```



```
{{ default | combine(patch, recursive=True, list_merge='append_rp') }}
```



这将导致

```
a:
  a':
    x: default_value
    y: patch_value
    z: patch_value
    list:
      - default_value
      - patch_value
b:
  - 1
  - 1
  - 2
  - 3
  - 4
  - 4
  - key: value
```





### 从数组或哈希表中选择值

2.1 版本中的新功能。

使用extract 过滤器将索引列表映射到容器（哈希或数组）中的值列表

```
{{ [0,2] | map('extract', ['x','y','z']) | list }}
{{ ['x','y'] | map('extract', {'x': 42, 'y': 31}) | list }}
```



上述表达式的结果将是

```
['x', 'z']
[42, 31]
```



过滤器可以接受另一个参数

```
{{ groups['x'] | map('extract', hostvars, 'ec2_ip_address') | list }}
```



这将获取“x”组中的主机列表，在hostvars中查找它们，然后查找结果的ec2_ip_address。“x”组中主机的最终结果是 IP 地址列表。

过滤器的第三个参数也可以是列表，用于在容器内进行递归查找

```
{{ ['a'] | map('extract', b, ['x','y']) | list }}
```



这将返回一个包含b'a'['y']值的列表。

### 组合列表

这组过滤器返回组合列表的列表。

#### 排列

要获取列表的排列

```
- name: Give me the largest permutations (order matters)
  ansible.builtin.debug:
    msg: "{{ [1,2,3,4,5] | ansible.builtin.permutations | list }}"

- name: Give me permutations of sets of three
  ansible.builtin.debug:
    msg: "{{ [1,2,3,4,5] | ansible.builtin.permutations(3) | list }}"
```



#### 组合

组合始终需要一个集合大小

```
- name: Give me combinations for sets of two
  ansible.builtin.debug:
    msg: "{{ [1,2,3,4,5] | ansible.builtin.combinations(2) | list }}"
```



另请参见zip_filter

#### 乘积

乘积过滤器返回输入可迭代对象的笛卡尔积。这大致相当于生成器表达式中的嵌套 for 循环。

例如

```
- name: Generate multiple hostnames
  ansible.builtin.debug:
    msg: "{{ ['foo', 'bar'] | product(['com']) | map('join', '.') | join(',') }}"
```



这将导致

```
{ "msg": "foo.com,bar.com" }
```



### 选择 JSON 数据：JSON 查询

要从 JSON 格式的复杂数据结构（例如 Ansible facts）中选择单个元素或数据子集，请使用community.general.json_query 过滤器。community.general.json_query 过滤器允许您查询复杂的 JSON 结构并使用循环结构对其进行迭代。

考虑以下数据结构

```
{
    "domain": {
        "cluster": [
            {
                "name": "cluster1"
            },
            {
                "name": "cluster2"
            }
        ],
        "server": [
            {
                "name": "server11",
                "cluster": "cluster1",
                "port": "8080"
            },
            {
                "name": "server12",
                "cluster": "cluster1",
                "port": "8090"
            },
            {
                "name": "server21",
                "cluster": "cluster2",
                "port": "9080"
            },
            {
                "name": "server22",
                "cluster": "cluster2",
                "port": "9090"
            }
        ],
        "library": [
            {
                "name": "lib1",
                "target": "cluster1"
            },
            {
                "name": "lib2",
                "target": "cluster2"
            }
        ]
    }
}
```



要从此结构中提取所有集群，可以使用以下查询

```
- name: Display all cluster names
  ansible.builtin.debug:
    var: item
  loop: "{{ domain_definition | community.general.json_query('domain.cluster[*].name') }}"
```



要提取所有服务器名称

```
- name: Display all server names
  ansible.builtin.debug:
    var: item
  loop: "{{ domain_definition | community.general.json_query('domain.server[*].name') }}"
```



要从 cluster1 提取端口

```
- name: Display all ports from cluster1
  ansible.builtin.debug:
    var: item
  loop: "{{ domain_definition | community.general.json_query(server_name_cluster1_query) }}"
  vars:
    server_name_cluster1_query: "domain.server[?cluster=='cluster1'].port"
```



要以逗号分隔的字符串形式打印 cluster1 的端口

```
- name: Display all ports from cluster1 as a string
  ansible.builtin.debug:
    msg: "{{ domain_definition | community.general.json_query('domain.server[?cluster==`cluster1`].port') | join(', ') }}"
```



您可以使用 YAML 单引号转义

```
- name: Display all ports from cluster1
  ansible.builtin.debug:
    var: item
  loop: "{{ domain_definition | community.general.json_query('domain.server[?cluster==''cluster1''].port') }}"
```



要获取包含集群所有端口和名称的哈希映射

```
- name: Display all server ports and names from cluster1
  ansible.builtin.debug:
    var: item
  loop: "{{ domain_definition | community.general.json_query(server_name_cluster1_query) }}"
  vars:
    server_name_cluster1_query: "domain.server[?cluster=='cluster1'].{name: name, port: port}"
```



要提取名称以“server1”开头的所有集群的端口

```
- name: Display ports from all clusters with the name starting with 'server1'
  ansible.builtin.debug:
    msg: "{{ domain_definition | to_json | from_json | community.general.json_query(server_name_query) }}"
  vars:
    server_name_query: "domain.server[?starts_with(name,'server1')].port"
```



要提取名称包含“server1”的所有集群的端口

```
- name: Display ports from all clusters with the name containing 'server1'
  ansible.builtin.debug:
    msg: "{{ domain_definition | to_json | from_json | community.general.json_query(server_name_query) }}"
  vars:
    server_name_query: "domain.server[?contains(name,'server1')].port"
```



## 随机化数据

当您需要随机生成的值时，请使用以下过滤器之一。



### 随机 MAC 地址

2.6 版本新增。

此过滤器可用于根据字符串前缀生成随机 MAC 地址。

要从以“52:54:00”开头的字符串前缀获取随机 MAC 地址

```
"{{ '52:54:00' | community.general.random_mac }}"
# => '52:54:00:ef:1c:03'
```



请注意，如果前缀字符串有任何错误，过滤器将发出错误。

> 2.9 版中的新增功能。

从 Ansible 2.9 版开始，您还可以从种子初始化随机数生成器以创建随机但幂等的 MAC 地址

```
"{{ '52:54:00' | community.general.random_mac(seed=inventory_hostname) }}"
```





### 随机项目或数字

Ansible 中的 ansible.builtin.random 过滤器是默认 Jinja2 随机过滤器的扩展，可用于从一系列项目中返回随机项目或根据范围生成随机数。

要从列表中获取随机项目

```
"{{ ['a','b','c'] | random }}"
# => 'c'
```



要获取 0（包含）到指定整数（不包含）之间的随机数

```
"{{ 60 | random }} * * * * root /script/from/cron"
# => '21 * * * * root /script/from/cron'
```



要获取 0 到 100 之间的随机数，但步长为 10

```
{{ 101 | random(step=10) }}
# => 70
```



要获取 1 到 100 之间的随机数，但步长为 10

```
{{ 101 | random(1, 10) }}
# => 31
{{ 101 | random(start=1, step=10) }}
# => 51
```



您可以从种子初始化随机数生成器以创建随机但幂等的数字

```
"{{ 60 | random(seed=inventory_hostname) }} * * * * root /script/from/cron"
```



### 随机排列列表

ansible.builtin.shuffle 过滤器会随机化现有列表，每次调用都会给出不同的顺序。

要从现有列表中获取随机列表

```
{{ ['a','b','c'] | shuffle }}
# => ['c','a','b']
{{ ['a','b','c'] | shuffle }}
# => ['b','c','a']
```



您可以从种子初始化随机排列生成器以生成随机但幂等的顺序

```
{{ ['a','b','c'] | shuffle(seed=inventory_hostname) }}
# => ['b','a','c']
```



shuffle 过滤器尽可能返回列表。如果将其与非“列表化”项目一起使用，则过滤器将不执行任何操作。



## 管理列表变量

您可以搜索列表中的最小值或最大值，或展平多级列表。

要从数字列表中获取最小值

```
{{ list1 | min }}
```



2.11 版中的新增功能。

要获取对象列表中的最小值

```
{{ [{'val': 1}, {'val': 2}] | min(attribute='val') }}
```



要从数字列表中获取最大值

```
{{ [3, 4, 2] | max }}
```



2.11 版中的新增功能。

要获取对象列表中的最大值

```
{{ [{'val': 1}, {'val': 2}] | max(attribute='val') }}
```



2.5 版中的新增功能。

展平列表（与 flatten 查找执行的操作相同）

```
{{ [3, [4, 2] ] | flatten }}
# => [3, 4, 2]
```



仅展平列表的第一级（类似于 items 查找）

```
{{ [3, [4, [2]] ] | flatten(levels=1) }}
# => [3, 4, [2]]
```



2.11 版中的新增功能。

保留列表中的空值，默认情况下，flatten 会移除它们。

```
{{ [3, None, [4, [2]] ] | flatten(levels=1, skip_nulls=False) }}
# => [3, None, 4, [2]]
```





## 从集合或列表中选择（集合论）

您可以从集合或列表中选择或组合项目。

1.4 版中的新增功能。

要从列表中获取唯一集合

```
# list1: [1, 2, 5, 1, 3, 4, 10]
{{ list1 | unique }}
# => [1, 2, 5, 3, 4, 10]
```



要获取两个列表的并集

```
# list1: [1, 2, 5, 1, 3, 4, 10]
# list2: [1, 2, 3, 4, 5, 11, 99]
{{ list1 | union(list2) }}
# => [1, 2, 5, 1, 3, 4, 10, 11, 99]
```



要获取两个列表的交集（两个列表中所有项目的唯一列表）

```
# list1: [1, 2, 5, 3, 4, 10]
# list2: [1, 2, 3, 4, 5, 11, 99]
{{ list1 | intersect(list2) }}
# => [1, 2, 5, 3, 4]
```



要获取两个列表的差集（列表 1 中不存在于列表 2 中的项目）

```
# list1: [1, 2, 5, 1, 3, 4, 10]
# list2: [1, 2, 3, 4, 5, 11, 99]
{{ list1 | difference(list2) }}
# => [10]
```



要获取两个列表的对称差集（每个列表独有的项目）

```
# list1: [1, 2, 5, 1, 3, 4, 10]
# list2: [1, 2, 3, 4, 5, 11, 99]
{{ list1 | symmetric_difference(list2) }}
# => [10, 11, 99]
```





## 计算数字（数学）

1.9 版中的新增功能。

您可以使用 Ansible 过滤器计算数字的对数、幂和根。Jinja2 提供其他数学函数，例如 abs() 和 round()。

获取对数（默认为 e）

```
{{ 8 | log }}
# => 2.0794415416798357
```



获取以 10 为底的对数

```
{{ 8 | log(10) }}
# => 0.9030899869919435
```



给我 2 的幂！（或 5）

```
{{ 8 | pow(5) }}
# => 32768.0
```



平方根或 5 次方根

```
{{ 8 | root }}
# => 2.8284271247461903

{{ 8 | root(5) }}
# => 1.5157165665103982
```



## 管理网络交互

这些过滤器可以帮助您完成常见的网络任务。



### IP 地址过滤器

1.9 版中的新增功能。

测试字符串是否为有效的 IP 地址

```
{{ myvar | ansible.utils.ipaddr }}
```



您还可以要求特定的 IP 协议版本

```
{{ myvar | ansible.utils.ipv4 }}
{{ myvar | ansible.utils.ipv6 }}
```



IP 地址过滤器还可以用于从 IP 地址中提取特定信息。例如，要从 CIDR 获取 IP 地址本身，可以使用

```
{{ '192.0.2.1/24' | ansible.utils.ipaddr('address') }}
# => 192.0.2.1
```



有关 ansible.utils.ipaddr 过滤器的更多信息和完整的用法指南，请参见 Ansible.Utils。



### 网络 CLI 过滤器

2.4 版中的新增功能。

要将网络设备 CLI 命令的输出转换为结构化的 JSON 输出，请使用 ansible.netcommon.parse_cli 过滤器

```
{{ output | ansible.netcommon.parse_cli('path/to/spec') }}
```



ansible.netcommon.parse_cli 过滤器将加载规范文件并将命令输出传递到该文件中，返回 JSON 输出。YAML 规范文件定义了如何解析 CLI 输出。

规范文件应为格式正确的 YAML 文件。它定义了如何解析 CLI 输出并返回 JSON 数据。下面是一个有效的规范文件示例，它将解析 show vlan 命令的输出。

```
---
vars:
  vlan:
    vlan_id: "{{ item.vlan_id }}"
    name: "{{ item.name }}"
    enabled: "{{ item.state != 'act/lshut' }}"
    state: "{{ item.state }}"

keys:
  vlans:
    value: "{{ vlan }}"
    items: "^(?P<vlan_id>\\d+)\\s+(?P<name>\\w+)\\s+(?P<state>active|act/lshut|suspended)"
  state_static:
    value: present
```



上面的规范文件将返回一个 JSON 数据结构，该结构是包含已解析 VLAN 信息的哈希列表。

可以使用 key 和 values 指令将相同的命令解析为哈希。以下是如何使用相同的 show vlan 命令将输出解析为哈希值的一个示例。

```
---
vars:
  vlan:
    key: "{{ item.vlan_id }}"
    values:
      vlan_id: "{{ item.vlan_id }}"
      name: "{{ item.name }}"
      enabled: "{{ item.state != 'act/lshut' }}"
      state: "{{ item.state }}"

keys:
  vlans:
    value: "{{ vlan }}"
    items: "^(?P<vlan_id>\\d+)\\s+(?P<name>\\w+)\\s+(?P<state>active|act/lshut|suspended)"
  state_static:
    value: present
```



解析 CLI 命令的另一个常见用例是将大型命令分解为可以解析的块。这可以使用 start_block 和 end_block 指令来完成，将命令分解为可以解析的块。

```
---
vars:
  interface:
    name: "{{ item[0].match[0] }}"
    state: "{{ item[1].state }}"
    mode: "{{ item[2].match[0] }}"

keys:
  interfaces:
    value: "{{ interface }}"
    start_block: "^Ethernet.*$"
    end_block: "^$"
    items:
      - "^(?P<name>Ethernet\\d\\/\\d*)"
      - "admin state is (?P<state>.+),"
      - "Port mode is (.+)"
```



上面的示例将 show interface 的输出解析为哈希列表。

网络过滤器还支持使用 TextFSM 库解析 CLI 命令的输出。要使用 TextFSM 解析 CLI 输出，请使用以下过滤器

```
{{ output.stdout[0] | ansible.netcommon.parse_cli_textfsm('path/to/fsm') }}
```



使用 TextFSM 过滤器需要安装 TextFSM 库。

### 网络 XML 过滤器

2.5 版中的新增功能。

要将网络设备命令的 XML 输出转换为结构化的 JSON 输出，请使用 ansible.netcommon.parse_xml 过滤器

```
{{ output | ansible.netcommon.parse_xml('path/to/spec') }}
```



ansible.netcommon.parse_xml 过滤器将加载规范文件并将命令输出传递到格式化为 JSON 的文件中。

规范文件应为格式正确的 YAML 文件。它定义了如何解析 XML 输出并返回 JSON 数据。

下面是一个有效的规范文件示例，它将解析 show vlan | display xml 命令的输出。

```
---
vars:
  vlan:
    vlan_id: "{{ item.vlan_id }}"
    name: "{{ item.name }}"
    desc: "{{ item.desc }}"
    enabled: "{{ item.state.get('inactive') != 'inactive' }}"
    state: "{% if item.state.get('inactive') == 'inactive'%} inactive {% else %} active {% endif %}"

keys:
  vlans:
    value: "{{ vlan }}"
    top: configuration/vlans/vlan
    items:
      vlan_id: vlan-id
      name: name
      desc: description
      state: ".[@inactive='inactive']"
```



上面的规范文件将返回一个 JSON 数据结构，该结构是包含已解析 VLAN 信息的哈希列表。

可以使用 key 和 values 指令将相同的命令解析为哈希。以下是如何使用相同的 show vlan | display xml 命令将输出解析为哈希值的一个示例。

```
---
vars:
  vlan:
    key: "{{ item.vlan_id }}"
    values:
        vlan_id: "{{ item.vlan_id }}"
        name: "{{ item.name }}"
        desc: "{{ item.desc }}"
        enabled: "{{ item.state.get('inactive') != 'inactive' }}"
        state: "{% if item.state.get('inactive') == 'inactive'%} inactive {% else %} active {% endif %}"

keys:
  vlans:
    value: "{{ vlan }}"
    top: configuration/vlans/vlan
    items:
      vlan_id: vlan-id
      name: name
      desc: description
      state: ".[@inactive='inactive']"
```



top的值是相对于XML根节点的XPath表达式。在下面的示例XML输出中，top的值为configuration/vlans/vlan，这是一个相对于根节点（`）的XPath表达式。top值中的configuration是最外层的容器节点，vlan`是最内层的容器节点。

items是一个键值对字典，它将用户定义的名称映射到选择元素的XPath表达式。XPath表达式相对于top中包含的XPath值。例如，规范文件中的vlan_id是用户定义的名称，其值vlan-id是相对于top中XPath值的相对路径。

可以使用XPath表达式提取XML标签的属性。规范中state的值是一个XPath表达式，用于获取输出XML中vlan标签的属性。

```
<rpc-reply>
  <configuration>
    <vlans>
      <vlan inactive="inactive">
       <name>vlan-1</name>
       <vlan-id>200</vlan-id>
       <description>This is vlan-1</description>
      </vlan>
    </vlans>
  </configuration>
</rpc-reply>
```



### 网络VLAN过滤器

2.8 版本新增。

使用ansible.netcommon.vlan_parser过滤器将VLAN整数的无序列表转换为根据类似IOS的VLAN列表规则排序的整数字符串列表。此列表具有以下属性：

- VLAN按升序排列。
- 三个或更多连续的VLAN用短横线列出。
- 列表的第一行可以是first_line_len个字符长。
- 后续列表行可以是other_line_len个字符长。

要排序VLAN列表：

```
{{ [3003, 3004, 3005, 100, 1688, 3002, 3999] | ansible.netcommon.vlan_parser }}
```



此示例呈现以下排序列表：

```
['100,1688,3002-3005,3999']
```



另一个Jinja模板示例：

```
{% set parsed_vlans = vlans | ansible.netcommon.vlan_parser %}
switchport trunk allowed vlan {{ parsed_vlans[0] }}
{% for i in range (1, parsed_vlans | count) %}
switchport trunk allowed vlan add {{ parsed_vlans[i] }}
{% endfor %}
```



这允许在Cisco IOS标记接口上动态生成VLAN列表。您可以存储接口所需确切VLAN的详尽原始列表，然后将其与将实际为配置生成的已解析IOS输出进行比较。



## 字符串和密码的哈希和加密

1.9 版中的新增功能。

要获取字符串的sha1哈希值：

```
{{ 'test1' | hash('sha1') }}
# => "b444ac06613fc8d63795be9ad0beaf55011936ac"
```



要获取字符串的md5哈希值：

```
{{ 'test1' | hash('md5') }}
# => "5a105e8b9d40e1329780d62ea2265d8a"
```



获取字符串校验和：

```
{{ 'test2' | checksum }}
# => "109f4b3c50d7b0df729d299bc6f8e9ef9066971f"
```



其他哈希（依赖于平台）：

```
{{ 'test2' | hash('blowfish') }}
```



要获取sha512密码哈希（随机盐）：

```
{{ 'passwordsaresecret' | password_hash('sha512') }}
# => "$6$UIv3676O/ilZzWEE$ktEfFF19NQPF2zyxqxGkAceTnbEgpEKuGBtk6MlU4v2ZorWaVQUMyurgmHCh2Fr4wpmQ/Y.AlXMJkRnIS4RfH/"
```



要获取具有特定盐的sha256密码哈希：

```
{{ 'secretpassword' | password_hash('sha256', 'mysecretsalt') }}
# => "$5$mysecretsalt$ReKNyDYjkKNqRVwouShhsEqZ3VOE8eoVO4exihOfvG4"
```



一种生成每个系统唯一哈希的幂等方法是使用在运行之间一致的盐。

```
{{ 'secretpassword' | password_hash('sha512', 65534 | random(seed=inventory_hostname) | string) }}
# => "$6$43927$lQxPKz2M2X.NWO.gK.t7phLwOKQMcSq72XxDZQ0XzYV6DlL1OD72h417aj16OnHTGxNzhftXJQBcjbunLEepM0"
```



可用的哈希类型取决于运行Ansible的控制系统，ansible.builtin.hash依赖于hashlib，ansible.builtin.password_hash依赖于passlib。crypt用作回退，如果未安装passlib。

2.7 版本新增。

某些哈希类型允许提供rounds参数。

```
{{ 'secretpassword' | password_hash('sha256', 'mysecretsalt', rounds=10000) }}
# => "$5$rounds=10000$mysecretsalt$Tkm80llAxD4YHll6AgNIztKn0vzAACsuuEfYeGP7tm7"
```



过滤器password_hash产生的结果取决于您是否安装了passlib。

为了确保幂等性，请指定rounds既不是crypt的默认值也不是passlib的默认值，对于crypt是5000，对于passlib是可变值（sha256为535000，sha512为656000）。

```
{{ 'secretpassword' | password_hash('sha256', 'mysecretsalt', rounds=5001) }}
# => "$5$rounds=5001$mysecretsalt$wXcTWWXbfcR8er5IVf7NuquLvnUA6s8/qdtOhAZ.xN."
```



哈希类型“blowfish”（BCrypt）提供了指定BCrypt算法版本的功能。

```
{{ 'secretpassword' | password_hash('blowfish', '1234567890123456789012', ident='2b') }}
# => "$2b$12$123456789012345678901uuJ4qFdej6xnWjOQT.FStqfdoY8dYUPC"
```



2.12版本的新增功能。

您还可以使用Ansible ansible.builtin.vault过滤器加密数据。

```
# simply encrypt my key in a vault
vars:
  myvaultedkey: "{{ keyrawdata|vault(passphrase) }}"

- name: save templated vaulted data
  template: src=dump_template_data.j2 dest=/some/key/vault.txt
  vars:
    mysalt: '{{ 2**256|random(seed=inventory_hostname) }}'
    template_data: '{{ secretdata|vault(vaultsecret, salt=mysalt) }}'
```



然后使用unvault过滤器解密它。

```
# simply decrypt my key from a vault
vars:
  mykey: "{{ myvaultedkey|unvault(passphrase) }}"

- name: save templated unvaulted data
  template: src=dump_template_data.j2 dest=/some/key/clear.txt
  vars:
    template_data: '{{ secretdata|unvault(vaultsecret) }}'
```





## 操作文本

几个过滤器可用于处理文本，包括URL、文件名和路径名。



### 向文件添加注释

ansible.builtin.comment过滤器允许您使用各种注释样式从模板中的文本创建文件中的注释。默认情况下，Ansible 使用# 来开始注释行，并在注释文本的上方和下方添加一个空注释行。例如，以下内容：

```
{{ "Plain style (default)" | comment }}
```



产生以下输出：

```
#
# Plain style (default)
#
```



Ansible 提供了 C (//...)、C 块 (/*...*/)、Erlang (%...) 和 XML (<!--...-->) 注释样式。

```
{{ "C style" | comment('c') }}
{{ "C block style" | comment('cblock') }}
{{ "Erlang style" | comment('erlang') }}
{{ "XML style" | comment('xml') }}
```



您可以定义自定义注释字符。此过滤器：

```
{{ "My Special Case" | comment(decoration="! ") }}
```



产生：

```
!
! My Special Case
!
```



您可以完全自定义注释样式：

```
{{ "Custom style" | comment('plain', prefix='#######\n#', postfix='#\n#######\n   ###\n    #') }}
```



这将创建以下输出：

```
#######
#
# Custom style
#
#######
   ###
    #
```



该过滤器还可以应用于任何Ansible变量。例如，为了使ansible_managed变量的输出更易读，我们可以在ansible.cfg文件中将其定义更改为此：

```
[defaults]

ansible_managed = This file is managed by Ansible.%n
  template: {file}
  date: %Y-%m-%d %H:%M:%S
  user: {uid}
  host: {host}
```



然后使用带有comment过滤器的变量：

```
{{ ansible_managed | comment }}
```



这将产生以下输出：

```
#
# This file is managed by Ansible.
#
# template: /home/ansible/env/dev/ansible_managed/roles/role1/templates/test.j2
# date: 2015-09-10 11:02:58
# user: ansible
# host: myhost
#
```



### URL编码变量

urlencode过滤器使用UTF-8对数据进行引用，以便在URL路径或查询中使用。

```
{{ 'Trollhättan' | urlencode }}
# => 'Trollh%C3%A4ttan'
```



### 拆分URL

2.4 版中的新增功能。

ansible.builtin.urlsplit过滤器从URL中提取片段、主机名、netloc、密码、路径、端口、查询、方案和用户名。没有参数时，返回所有字段的字典。

```
{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('hostname') }}
# => 'www.acme.com'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('netloc') }}
# => 'user:[email protected]:9000'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('username') }}
# => 'user'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('password') }}
# => 'password'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('path') }}
# => '/dir/index.html'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('port') }}
# => '9000'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('scheme') }}
# => 'http'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('query') }}
# => 'query=term'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit('fragment') }}
# => 'fragment'

{{ "http://user:[email protected]:9000/dir/index.html?query=term#fragment" | urlsplit }}
# =>
#   {
#       "fragment": "fragment",
#       "hostname": "www.acme.com",
#       "netloc": "user:[email protected]:9000",
#       "password": "password",
#       "path": "/dir/index.html",
#       "port": 9000,
#       "query": "query=term",
#       "scheme": "http",
#       "username": "user"
#   }
```



### 使用正则表达式搜索字符串

要使用正则表达式在字符串中搜索或提取字符串的部分，请使用ansible.builtin.regex_search过滤器。

```
# Extracts the database name from a string
{{ 'server1/database42' | regex_search('database[0-9]+') }}
# => 'database42'

# Example for a case insensitive search in multiline mode
{{ 'foo\nBAR' | regex_search('^bar', multiline=True, ignorecase=True) }}
# => 'BAR'

# Example for a case insensitive search in multiline mode using inline regex flags
{{ 'foo\nBAR' | regex_search('(?im)^bar') }}
# => 'BAR'

# Extracts server and database id from a string
{{ 'server1/database42' | regex_search('server([0-9]+)/database([0-9]+)', '\\1', '\\2') }}
# => ['1', '42']

# Extracts dividend and divisor from a division
{{ '21/42' | regex_search('(?P<dividend>[0-9]+)/(?P<divisor>[0-9]+)', '\\g<dividend>', '\\g<divisor>') }}
# => ['21', '42']
```



ansible.builtin.regex_search过滤器如果找不到匹配项，则返回空字符串。

```
{{ 'ansible' | regex_search('foobar') }}
# => ''
```



要提取字符串中正则表达式匹配的所有匹配项，请使用ansible.builtin.regex_findall过滤器。

```
# Returns a list of all IPv4 addresses in the string
{{ 'Some DNS servers are 8.8.8.8 and 8.8.4.4' | regex_findall('\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b') }}
# => ['8.8.8.8', '8.8.4.4']

# Returns all lines that end with "ar"
{{ 'CAR\ntar\nfoo\nbar\n' | regex_findall('^.ar$', multiline=True, ignorecase=True) }}
# => ['CAR', 'tar', 'bar']

# Returns all lines that end with "ar" using inline regex flags for multiline and ignorecase
{{ 'CAR\ntar\nfoo\nbar\n' | regex_findall('(?im)^.ar$') }}
# => ['CAR', 'tar', 'bar']
```



要使用正则表达式替换字符串中的文本，请使用ansible.builtin.regex_replace过滤器。

```
# Convert "ansible" to "able"
{{ 'ansible' | regex_replace('^a.*i(.*)$', 'a\\1') }}
# => 'able'

# Convert "foobar" to "bar"
{{ 'foobar' | regex_replace('^f.*o(.*)$', '\\1') }}
# => 'bar'

# Convert "localhost:80" to "localhost, 80" using named groups
{{ 'localhost:80' | regex_replace('^(?P<host>.+):(?P<port>\\d+)$', '\\g<host>, \\g<port>') }}
# => 'localhost, 80'

# Convert "localhost:80" to "localhost"
{{ 'localhost:80' | regex_replace(':80') }}
# => 'localhost'

# Comment all lines that end with "ar"
{{ 'CAR\ntar\nfoo\nbar\n' | regex_replace('^(.ar)$', '#\\1', multiline=True, ignorecase=True) }}
# => '#CAR\n#tar\nfoo\n#bar\n'

# Comment all lines that end with "ar" using inline regex flags for multiline and ignorecase
{{ 'CAR\ntar\nfoo\nbar\n' | regex_replace('(?im)^(.ar)$', '#\\1') }}
# => '#CAR\n#tar\nfoo\n#bar\n'
```



```
# add "https://" prefix to each item in a list
GOOD:
{{ hosts | map('regex_replace', '^(.*)$', 'https://\\1') | list }}
{{ hosts | map('regex_replace', '(.+)', 'https://\\1') | list }}
{{ hosts | map('regex_replace', '^', 'https://') | list }}

BAD:
{{ hosts | map('regex_replace', '(.*)', 'https://\\1') | list }}

# append ':80' to each item in a list
GOOD:
{{ hosts | map('regex_replace', '^(.*)$', '\\1:80') | list }}
{{ hosts | map('regex_replace', '(.+)', '\\1:80') | list }}
{{ hosts | map('regex_replace', '$', ':80') | list }}

BAD:
{{ hosts | map('regex_replace', '(.*)', '\\1:80') | list }}
```



2.0 版本中的新功能。

要转义标准Python正则表达式中的特殊字符，请使用ansible.builtin.regex_escape过滤器（使用默认的re_type='python'选项）。

```
# convert '^f.*o(.*)$' to '\^f\.\*o\(\.\*\)\$'
{{ '^f.*o(.*)$' | regex_escape() }}
```



2.8 版本新增。

要转义POSIX基本正则表达式中的特殊字符，请使用ansible.builtin.regex_escape过滤器以及re_type='posix_basic'选项。

```
# convert '^f.*o(.*)$' to '\^f\.\*o(\.\*)\$'
{{ '^f.*o(.*)$' | regex_escape('posix_basic') }}
```



### 管理文件名和路径名

要获取文件路径的最后一个名称，例如从“/etc/asdf/foo.txt”中获取“foo.txt”：

```
{{ path | basename }}
```



要获取Windows样式文件路径的最后一个名称（2.0版本中的新增功能）：

```
{{ path | win_basename }}
```



要将Windows驱动器号与文件路径的其余部分分开（2.0版本中的新增功能）：

```
{{ path | win_splitdrive }}
```



仅获取Windows驱动器号：

```
{{ path | win_splitdrive | first }}
```



获取路径中不包含驱动器号的其余部分

```
{{ path | win_splitdrive | last }}
```



从路径中获取目录

```
{{ path | dirname }}
```



从Windows路径中获取目录（2.0新版本）

```
{{ path | win_dirname }}
```



展开包含波浪号（~）字符的路径（1.5版本新增）

```
{{ path | expanduser }}
```



展开包含环境变量的路径

```
{{ path | expandvars }}
```



2.6 版本新增。

获取链接的真实路径（1.8版本新增）

```
{{ path | realpath }}
```



获取链接相对于起始点的相对路径（1.7版本新增）

```
{{ path | relpath('/etc') }}
```



获取路径或文件名的根和扩展名（2.0新版本）

```
# with path == 'nginx.conf' the return would be ('nginx', '.conf')
{{ path | splitext }}
```



ansible.builtin.splitext 过滤器始终返回一对字符串。可以使用 first 和 last 过滤器访问各个组件

```
# with path == 'nginx.conf' the return would be 'nginx'
{{ path | splitext | first }}

# with path == 'nginx.conf' the return would be '.conf'
{{ path | splitext | last }}
```



连接一个或多个路径组件

```
{{ ('/etc', path, 'subdir', file) | path_join }}
```



2.10版本新增。

## 操作字符串

为shell使用添加引号

```
- name: Run a shell command
  ansible.builtin.shell: echo {{ string_value | quote }}
```



（文档：ansible.builtin.quote）

将列表连接成字符串

```
{{ list | join(" ") }}
```



将字符串分割成列表

```
{{ csv_string | split(",") }}
```



2.11 版中的新增功能。

使用Base64编码的字符串

```
{{ encoded | b64decode }}
{{ decoded | string | b64encode }}
```



（文档：ansible.builtin.b64encode）

从2.6版本开始，您可以定义要使用的编码类型，默认为 utf-8

```
{{ encoded | b64decode(encoding='utf-16-le') }}
{{ decoded | string | b64encode(encoding='utf-16-le') }}
```



（文档：ansible.builtin.b64decode）

2.6 版本新增。

## 管理UUID

创建命名空间UUIDv5

```
{{ string | to_uuid(namespace='11111111-2222-3333-4444-555555555555') }}
```



2.10版本新增。

使用默认的Ansible命名空间“361E6D51-FAEC-444A-9079-341386DA8E2E”创建命名空间UUIDv5

```
{{ string | to_uuid }}
```



1.9 版中的新增功能。

要使用复杂变量列表中每个项目的某个属性，请使用Jinja2 map filter

```
# get a comma-separated list of the mount points (for example, "/,/mnt/stuff") on a host
{{ ansible_mounts | map(attribute='mount') | join(',') }}
```



## 处理日期和时间

要从字符串获取日期对象，请使用to_datetime过滤器

```
# Get the total amount of seconds between two dates. Default date format is %Y-%m-%d %H:%M:%S but you can pass your own format
{{ (("2016-08-14 20:00:12" | to_datetime) - ("2015-12-25" | to_datetime('%Y-%m-%d'))).total_seconds()  }}

# Get remaining seconds after delta has been calculated. NOTE: This does NOT convert years, days, hours, and so on to seconds. For that, use total_seconds()
{{ (("2016-08-14 20:00:12" | to_datetime) - ("2016-08-14 18:00:00" | to_datetime)).seconds  }}
# This expression evaluates to "12" and not "132". Delta is 2 hours, 12 seconds

# get amount of days between two dates. This returns only the number of days and discards remaining hours, minutes, and seconds
{{ (("2016-08-14 20:00:12" | to_datetime) - ("2015-12-25" | to_datetime('%Y-%m-%d'))).days  }}
```



2.4 版中的新增功能。

要使用字符串格式化日期（如使用shell date命令），请使用“strftime”过滤器

```
# Display year-month-day
{{ '%Y-%m-%d' | strftime }}
# => "2021-03-19"

# Display hour:min:sec
{{ '%H:%M:%S' | strftime }}
# => "21:51:04"

# Use ansible_date_time.epoch fact
{{ '%Y-%m-%d %H:%M:%S' | strftime(ansible_date_time.epoch) }}
# => "2021-03-19 21:54:09"

# Use arbitrary epoch value
{{ '%Y-%m-%d' | strftime(0) }}          # => 1970-01-01
{{ '%Y-%m-%d' | strftime(1441357287) }} # => 2015-09-04
```



2.13版本新增。

strftime带有一个可选的utc参数，默认为False，这意味着时间为本地时区

```
{{ '%H:%M:%S' | strftime }}           # time now in local timezone
{{ '%H:%M:%S' | strftime(utc=True) }} # time now in UTC
```



## 获取Kubernetes资源名称

使用“k8s_config_resource_name”过滤器获取Kubernetes ConfigMap或Secret的名称，包括其哈希值

```
{{ configmap_resource_definition | kubernetes.core.k8s_config_resource_name }}
```



然后可以使用它来引用Pod规范中的哈希值

```
my_secret:
  kind: Secret
  metadata:
    name: my_secret_name

deployment_resource:
  kind: Deployment
  spec:
    template:
      spec:
        containers:
        - envFrom:
            - secretRef:
                name: {{ my_secret | kubernetes.core.k8s_config_resource_name }}
```

*2.8 版本新增。*
