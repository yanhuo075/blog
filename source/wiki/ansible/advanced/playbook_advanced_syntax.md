---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.1 高级剧本语法
order: 0
---

# 高级剧本语法

本页上的高级 YAML 语法示例使您可以更好地控制 Ansible 使用的 YAML 文件中放置的数据。您可以在官方 PyYAML 文档中找到有关 Python 特定 YAML 的其他信息。



## 不安全或原始字符串

在处理查找插件返回的值时，Ansible 使用一种名为 unsafe 的数据类型来阻止模板化。将数据标记为不安全可以防止恶意用户滥用 Jinja2 模板在目标计算机上执行任意代码。Ansible 的实现确保永远不会对不安全的值进行模板化。它比使用 {% raw %} ... {% endraw %} 标签转义 Jinja2 更全面。

您可以在定义的变量中使用相同的 unsafe 数据类型，以防止模板化错误和信息泄露。您可以将 vars_prompts 提供的标记值标记为不安全。您也可以在剧本中使用 unsafe。最常见的用例包括允许使用 { 或 % 等特殊字符的密码，以及看起来像模板但不应进行模板化的 JSON 参数。例如：

```
---
mypassword: !unsafe 234%234{435lkj{{lkjsdf
```



在剧本中

```
---
hosts: all
vars:
  my_unsafe_variable: !unsafe 'unsafe % value'
tasks:
    ...
```



对于哈希或数组等复杂变量，请对各个元素使用 !unsafe

```
---
my_unsafe_array:
  - !unsafe 'unsafe element'
  - 'safe element'

my_unsafe_hash:
  unsafe_key: !unsafe 'unsafe value'
```





## YAML 锚点和别名：共享变量值

YAML 锚点和别名可帮助您定义、维护和灵活使用共享的变量值。您可以使用 & 定义锚点，然后使用别名（用 * 表示）引用它。以下示例使用锚点设置三个值，使用别名使用其中两个值，并覆盖第三个值：

```
---
...
vars:
  app1:
    jvm: &jvm_opts
      opts: '-Xms1G -Xmx2G'
      port: 1000
      path: /usr/lib/app1
  app2:
    jvm:
      <<: *jvm_opts
      path: /usr/lib/app2
...
```



在这里，app1 和 app2 使用锚点 &jvm_opts 和别名 *jvm_opts 共享 opts 和 port 的值。path 的值由 << 或 merge 运算符 合并。

锚点和别名还允许您共享复杂的变量值集，包括嵌套变量。如果有一个变量值包含另一个变量值，则可以分别定义它们：

```
vars:
  webapp_version: 1.0
  webapp_custom_name: ToDo_App-1.0
```



这效率低下，并且从规模上来说，意味着需要进行更多维护。要将版本值合并到名称中，可以在 app_version 中使用锚点，并在 custom_name 中使用别名：

```
vars:
  webapp:
      version: &my_version 1.0
      custom_name:
          - "ToDo_App"
          - *my_version
```



现在，您可以在 custom_name 的值中重用 app_version 的值，并在模板中使用输出：

```
---
- name: Using values nested inside dictionary
  hosts: localhost
  vars:
    webapp:
      version: &my_version 1.0
      custom_name:
        - "ToDo_App"
        - *my_version
  tasks:
  - name: Using Anchor value
    ansible.builtin.debug:
      msg: My app is called "{{ webapp.custom_name | join('-') }}".
```



您已使用 &my_version 锚点锚定了 version 的值，并使用 *my_version 别名对其进行了重用。锚点和别名允许您访问字典内的嵌套值。
