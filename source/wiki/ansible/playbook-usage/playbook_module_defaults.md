---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.16 模块默认值
order: 15
---

# 模块默认值

如果您经常使用相同的参数调用同一个模块，那么使用 module_defaults 关键字为该特定模块定义默认参数会很有用。

这是一个基本示例

```
- hosts: localhost
  module_defaults:
    ansible.builtin.file:
      owner: root
      group: root
      mode: 0755
  tasks:
    - name: Create file1
      ansible.builtin.file:
        state: touch
        path: /tmp/file1

    - name: Create file2
      ansible.builtin.file:
        state: touch
        path: /tmp/file2

    - name: Create file3
      ansible.builtin.file:
        state: touch
        path: /tmp/file3
```



module_defaults 关键字可以在 play、block 和 task 级别使用。在 task 中显式指定的任何模块参数都将覆盖该模块参数的任何已建立的默认值。

```
- block:
    - name: Print a message
      ansible.builtin.debug:
        msg: "Different message"
  module_defaults:
    ansible.builtin.debug:
      msg: "Default message"
```



您可以通过指定一个空字典来删除之前为模块建立的任何默认值。

```
- name: Create file1
  ansible.builtin.file:
    state: touch
    path: /tmp/file1
  module_defaults:
    file: {}
```



以下是此功能的一些更实际的用例。

与需要身份验证的 API 交互。

```
- hosts: localhost
  module_defaults:
    ansible.builtin.uri:
      force_basic_auth: true
      user: some_user
      password: some_password
  tasks:
    - name: Interact with a web service
      ansible.builtin.uri:
        url: http://some.api.host/v1/whatever1

    - name: Interact with a web service
      ansible.builtin.uri:
        url: http://some.api.host/v1/whatever2

    - name: Interact with a web service
      ansible.builtin.uri:
        url: http://some.api.host/v1/whatever3
```



为特定的 EC2 相关模块设置默认的 AWS 区域。

```
- hosts: localhost
  vars:
    my_region: us-west-2
  module_defaults:
    amazon.aws.ec2:
      region: '{{ my_region }}'
    community.aws.ec2_instance_info:
      region: '{{ my_region }}'
    amazon.aws.ec2_vpc_net_info:
      region: '{{ my_region }}'
```





## 模块默认组

模块默认组允许为属于同一组的模块提供通用参数。集合可以在其 meta/runtime.yml 文件中定义此类组。

以下是 ns.coll 集合的示例 runtime.yml 文件。此文件定义了一个名为 ns.coll.my_group 的操作组，并将 ns.coll 中的 sample_module 和 another.collection 中的 another_module 放入该组。

```
# collections/ansible_collections/ns/coll/meta/runtime.yml
action_groups:
  my_group:
    - sample_module
    - another.collection.another_module
```



现在可以在 Playbook 中像这样使用此组

```
- hosts: localhost
  module_defaults:
    group/ns.coll.my_group:
      option_name: option_value
  tasks:
    - ns.coll.sample_module:
    - another.collection.another_module:
```



出于历史原因和向后兼容性，有一些特殊的组

| 组      | 扩展模块组                                                   |
| ------- | ------------------------------------------------------------ |
| aws     | amazon.aws.aws 和 community.aws.aws                          |
| azure   | azure.azcollection.azure                                     |
| gcp     | google.cloud.gcp                                             |
| k8s     | community.kubernetes.k8s、community.general.k8s、community.kubevirt.k8s、community.okd.k8s 和 kubernetes.core.k8s |
| os      | openstack.cloud.os                                           |
| acme    | community.crypto.acme                                        |
| docker* | community.general.docker 和 community.docker.docker          |
| ovirt   | ovirt.ovirt.ovirt 和 community.general.ovirt                 |
| vmware  | community.vmware.vmware                                      |

- 查看集合或其 meta/runtime.yml 的文档，以了解哪些操作插件和模块包含在该组中。

通过在组名前加上 group/ 来使用带有 module_defaults 的组 - 例如 group/aws。

在 Playbook 中，您可以为整组模块设置模块默认值，例如设置通用的 AWS 区域。

```
# example_play.yml
- hosts: localhost
  module_defaults:
    group/aws:
      region: us-west-2
  tasks:
  - name: Get info
    aws_s3_bucket_info:

  # now the region is shared between both info modules

  - name: Get info
    ec2_ami_info:
      filters:
        name: 'RHEL*7.5*'
```



有关 meta/runtime.yml 的更多信息，包括 action_groups 的完整格式，请参阅 [runtime.yml](https://docs.ansible.org.cn/ansible/latest/dev_guide/developing_collections_structure.html#meta-runtime-yml)。
