---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.8 网络资源模块
order: 7
---

# 网络资源模块

Ansible 网络资源模块简化并标准化了您管理不同网络设备的方式。网络设备将配置分为适用于网络服务的各个部分（例如接口和 VLAN）。Ansible 网络资源模块利用这一点，允许您配置网络设备配置中的子部分或资源。网络资源模块为不同的网络设备提供了始终如一的体验。

- 网络资源模块状态
- 使用网络资源模块
- 示例：验证网络设备配置未更改
- 示例：获取和更新网络设备上的 VLAN

## 网络资源模块状态

您可以通过为模块要执行的操作分配状态来使用网络资源模块。资源模块支持以下状态：

- merged (合并)

  Ansible 将设备上的配置与任务中提供的配置合并。

- replaced (替换)

  Ansible 将设备上的配置子部分替换为任务中提供的配置子部分。

- overridden (覆盖)

  Ansible 使用任务中提供的配置覆盖设备上资源的配置。使用此状态时请谨慎，因为您可能会失去对设备的访问权限（例如，通过覆盖管理接口配置）。

- deleted (删除)

  Ansible 删除设备上的配置子部分并恢复任何默认设置。

- gathered (收集)

  Ansible 显示从网络设备收集并通过结果中的gathered键访问的资源详细信息。

- rendered (渲染)

  Ansible 以设备原生格式（例如 Cisco IOS CLI）呈现任务中提供的配置。Ansible 将此呈现的配置返回到结果中的rendered键中。请注意，此状态不会与网络设备通信，可以在脱机状态下使用。

- parsed (解析)

  Ansible 将来自running_config选项的配置解析为结果中parsed键中的 Ansible 结构化数据。请注意，这不会从网络设备收集配置，因此此状态可以在脱机状态下使用。

## 使用网络资源模块

此示例根据不同的状态设置配置 Cisco IOS 设备上的 L3 接口资源。

> ```
> - name: configure l3 interface
>   cisco.ios.ios_l3_interfaces:
>     config: "{{ config }}"
>     state: <state>
> ```

下表显示了此任务的不同状态如何改变初始资源配置的示例。

| 资源初始配置                                                 | 任务提供的配置 (YAML)                                        | 设备上的最终资源配置                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| interface loopback100 ip address 10.10.1.100 255.255.255.0 ipv6 address FC00:100/64 | config: - ipv6: - address: fc00::100/64 - address: fc00::101/64 name: loopback100 | merged (合并)interface loopback100 ip address 10.10.1.100 255.255.255.0 ipv6 address FC00:100/64 ipv6 address FC00:101/64 |
| replaced (替换)interface loopback100 no ip address ipv6 address FC00:100/64 ipv6 address FC00:101/64 |                                                              |                                                              |
| overridden (覆盖)错误用例。这将删除设备上的所有接口（包括管理接口）除了配置的 loopback100 |                                                              |                                                              |
| deleted (删除)interface loopback100 no ip address            |                                                              |                                                              |

网络资源模块返回以下详细信息：

- 之前状态 - 任务执行之前存在的资源配置。
- 之后状态 - 任务执行后网络设备上存在的新的资源配置。
- 命令 - 在设备上配置的任何命令。

```
ok: [nxos101] =>
  result:
    after:
      contact: IT Support
      location: Room E, Building 6, Seattle, WA 98134
      users:
      - algorithm: md5
        group: network-admin
        localized_key: true
        password: '0x73fd9a2cc8c53ed3dd4ed8f4ff157e69'
        privacy_password: '0x73fd9a2cc8c53ed3dd4ed8f4ff157e69'
        username: admin
    before:
      contact: IT Support
      location: Room E, Building 5, Seattle HQ
      users:
      - algorithm: md5
        group: network-admin
        localized_key: true
        password: '0x73fd9a2cc8c53ed3dd4ed8f4ff157e69'
        privacy_password: '0x73fd9a2cc8c53ed3dd4ed8f4ff157e69'
        username: admin
    changed: true
    commands:
    - snmp-server location Room E, Building 6, Seattle, WA 98134
    failed: false
```



## 示例：验证网络设备配置未更改

以下剧本使用arista.eos.eos_l3_interfaces模块收集网络设备配置的子集（仅限第3层接口）并验证信息准确且未更改。此剧本将arista.eos.eos_facts的结果直接传递给arista.eos.eos_l3_interfaces模块。

```
- name: Example of facts being pushed right back to device.
  hosts: arista
  gather_facts: false
  tasks:
    - name: grab arista eos facts
      arista.eos.eos_facts:
        gather_subset: min
        gather_network_resources: l3_interfaces

- name: Ensure that the IP address information is accurate.
  arista.eos.eos_l3_interfaces:
    config: "{{ ansible_network_resources['l3_interfaces'] }}"
    register: result

- name: Ensure config did not change.
  assert:
    that: not result.changed
```



## 示例：获取和更新网络设备上的 VLAN

此示例演示如何使用资源模块：

1. 检索网络设备上的当前配置。
2. 在本地保存该配置。
3. 更新该配置并将其应用于网络设备。

此示例使用cisco.ios.ios_vlans资源模块来检索和更新 IOS 设备上的 VLAN。

1. 检索当前的 IOS VLAN 配置

```
- name: Gather VLAN information as structured data
  cisco.ios.ios_facts:
     gather_subset:
      - '!all'
      - '!min'
     gather_network_resources:
     - 'vlans'
```



1. 在本地存储 VLAN 配置

```
- name: Store VLAN facts to host_vars
  copy:
    content: "{{ ansible_network_resources | to_nice_yaml }}"
    dest: "{{ playbook_dir }}/host_vars/{{ inventory_hostname }}"
```



1. 修改存储的文件以在本地更新 VLAN 配置。
2. 将更新的 VLAN 配置与设备上的现有配置合并

```
- name: Make VLAN config changes by updating stored facts on the control node.
  cisco.ios.ios_vlans:
    config: "{{ vlans }}"
    state: merged
  tags: update_config
```
