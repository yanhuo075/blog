---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 8.7 使用网络连接选项
order: 6
---

# 使用网络连接选项

网络模块可以支持多种连接协议，例如 ansible.netcommon.network_cli、ansible.netcommon.netconf 和 ansible.netcommon.httpapi。这些连接包含一些您可以设置的常用选项，以控制与网络设备连接的行为。

常用选项包括：

- become 和 become_method，如特权提升：启用模式、become 和授权中所述。
- network_os - 设置为匹配您正在与之通信的网络平台。请参阅特定于平台的页面。
- remote_user，如设置远程用户中所述。
- 超时选项 - persistent_command_timeout、persistent_connect_timeout 和 timeout。



## 设置超时选项

与远程设备通信时，您可以控制 Ansible 维持与该设备连接的时间长度，以及 Ansible 等待该设备上命令完成的时间长度。每个选项都可以在 playbook 文件、环境变量或ansible.cfg 文件中的设置中作为变量设置。

例如，控制连接超时的三个选项如下所示。

使用变量（每个任务）

```
- name: save running-config
  cisco.ios.ios_command:
    commands: copy running-config startup-config
  vars:
    ansible_command_timeout: 30
```



使用环境变量

```
$export ANSIBLE_PERSISTENT_COMMAND_TIMEOUT=30
```



使用全局配置（在 ansible.cfg 中）

```
[persistent_connection]
command_timeout = 30
```



有关这些变量的相对优先级详情，请参阅变量优先级：我应该在哪里放置变量？。请参阅各个连接类型以了解每个选项。
