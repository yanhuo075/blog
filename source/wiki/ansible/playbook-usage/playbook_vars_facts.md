---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.19 发现变量
order: 18
---

# 发现变量：facts 和魔术变量

使用 Ansible，您可以检索或发现包含有关远程系统或 Ansible 自身信息的特定变量。与远程系统相关的变量称为 facts。通过 facts，您可以使用一个系统的行为或状态作为其他系统上的配置。例如，您可以使用一个系统的 IP 地址作为另一个系统上的配置值。与 Ansible 相关的变量称为魔术变量。

- Ansible facts
  - 收集 facts 的软件包要求
  - 缓存 facts
  - 禁用 facts
  - 添加自定义 facts
    - facts.d 或本地 facts
- 关于 Ansible 的信息：魔术变量
  - Ansible 版本

## Ansible facts

Ansible facts 是与远程系统相关的数据，包括操作系统、IP 地址、附加的文件系统等。您可以在 ansible_facts 变量中访问此数据。默认情况下，您还可以使用 ansible_ 前缀作为顶级变量访问某些 Ansible facts。您可以使用 INJECT_FACTS_AS_VARS 设置禁用此行为。要查看所有可用的 facts，请将此任务添加到 Play 中

```
- name: Print all available facts
  ansible.builtin.debug:
    var: ansible_facts
```



要以收集的“原始”信息查看，请在命令行运行此命令

```
ansible <hostname> -m ansible.builtin.setup
```



Facts 包含大量变量数据，可能如下所示

```
{
    "ansible_all_ipv4_addresses": [
        "REDACTED IP ADDRESS"
    ],
    "ansible_all_ipv6_addresses": [
        "REDACTED IPV6 ADDRESS"
    ],
    "ansible_apparmor": {
        "status": "disabled"
    },
    "ansible_architecture": "x86_64",
    "ansible_bios_date": "11/28/2013",
    "ansible_bios_version": "4.1.5",
    "ansible_cmdline": {
        "BOOT_IMAGE": "/boot/vmlinuz-3.10.0-862.14.4.el7.x86_64",
        "console": "ttyS0,115200",
        "no_timer_check": true,
        "nofb": true,
        "nomodeset": true,
        "ro": true,
        "root": "LABEL=cloudimg-rootfs",
        "vga": "normal"
    },
    "ansible_date_time": {
        "date": "2018-10-25",
        "day": "25",
        "epoch": "1540469324",
        "hour": "12",
        "iso8601": "2018-10-25T12:08:44Z",
        "iso8601_basic": "20181025T120844109754",
        "iso8601_basic_short": "20181025T120844",
        "iso8601_micro": "2018-10-25T12:08:44.109968Z",
        "minute": "08",
        "month": "10",
        "second": "44",
        "time": "12:08:44",
        "tz": "UTC",
        "tz_offset": "+0000",
        "weekday": "Thursday",
        "weekday_number": "4",
        "weeknumber": "43",
        "year": "2018"
    },
    "ansible_default_ipv4": {
        "address": "REDACTED",
        "alias": "eth0",
        "broadcast": "REDACTED",
        "gateway": "REDACTED",
        "interface": "eth0",
        "macaddress": "REDACTED",
        "mtu": 1500,
        "netmask": "255.255.255.0",
        "network": "REDACTED",
        "type": "ether"
    },
    "ansible_default_ipv6": {},
    "ansible_device_links": {
        "ids": {},
        "labels": {
            "xvda1": [
                "cloudimg-rootfs"
            ],
            "xvdd": [
                "config-2"
            ]
        },
        "masters": {},
        "uuids": {
            "xvda1": [
                "cac81d61-d0f8-4b47-84aa-b48798239164"
            ],
            "xvdd": [
                "2018-10-25-12-05-57-00"
            ]
        }
    },
    "ansible_devices": {
        "xvda": {
            "holders": [],
            "host": "",
            "links": {
                "ids": [],
                "labels": [],
                "masters": [],
                "uuids": []
            },
            "model": null,
            "partitions": {
                "xvda1": {
                    "holders": [],
                    "links": {
                        "ids": [],
                        "labels": [
                            "cloudimg-rootfs"
                        ],
                        "masters": [],
                        "uuids": [
                            "cac81d61-d0f8-4b47-84aa-b48798239164"
                        ]
                    },
                    "sectors": "83883999",
                    "sectorsize": 512,
                    "size": "40.00 GB",
                    "start": "2048",
                    "uuid": "cac81d61-d0f8-4b47-84aa-b48798239164"
                }
            },
            "removable": "0",
            "rotational": "0",
            "sas_address": null,
            "sas_device_handle": null,
            "scheduler_mode": "deadline",
            "sectors": "83886080",
            "sectorsize": "512",
            "size": "40.00 GB",
            "support_discard": "0",
            "vendor": null,
            "virtual": 1
        },
        "xvdd": {
            "holders": [],
            "host": "",
            "links": {
                "ids": [],
                "labels": [
                    "config-2"
                ],
                "masters": [],
                "uuids": [
                    "2018-10-25-12-05-57-00"
                ]
            },
            "model": null,
            "partitions": {},
            "removable": "0",
            "rotational": "0",
            "sas_address": null,
            "sas_device_handle": null,
            "scheduler_mode": "deadline",
            "sectors": "131072",
            "sectorsize": "512",
            "size": "64.00 MB",
            "support_discard": "0",
            "vendor": null,
            "virtual": 1
        },
        "xvde": {
            "holders": [],
            "host": "",
            "links": {
                "ids": [],
                "labels": [],
                "masters": [],
                "uuids": []
            },
            "model": null,
            "partitions": {
                "xvde1": {
                    "holders": [],
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "sectors": "167770112",
                    "sectorsize": 512,
                    "size": "80.00 GB",
                    "start": "2048",
                    "uuid": null
                }
            },
            "removable": "0",
            "rotational": "0",
            "sas_address": null,
            "sas_device_handle": null,
            "scheduler_mode": "deadline",
            "sectors": "167772160",
            "sectorsize": "512",
            "size": "80.00 GB",
            "support_discard": "0",
            "vendor": null,
            "virtual": 1
        }
    },
    "ansible_distribution": "CentOS",
    "ansible_distribution_file_parsed": true,
    "ansible_distribution_file_path": "/etc/redhat-release",
    "ansible_distribution_file_variety": "RedHat",
    "ansible_distribution_major_version": "7",
    "ansible_distribution_release": "Core",
    "ansible_distribution_version": "7.5.1804",
    "ansible_dns": {
        "nameservers": [
            "127.0.0.1"
        ]
    },
    "ansible_domain": "",
    "ansible_effective_group_id": 1000,
    "ansible_effective_user_id": 1000,
    "ansible_env": {
        "HOME": "/home/zuul",
        "LANG": "en_US.UTF-8",
        "LESSOPEN": "||/usr/bin/lesspipe.sh %s",
        "LOGNAME": "zuul",
        "MAIL": "/var/mail/zuul",
        "PATH": "/usr/local/bin:/usr/bin",
        "PWD": "/home/zuul",
        "SELINUX_LEVEL_REQUESTED": "",
        "SELINUX_ROLE_REQUESTED": "",
        "SELINUX_USE_CURRENT_RANGE": "",
        "SHELL": "/bin/bash",
        "SHLVL": "2",
        "SSH_CLIENT": "REDACTED 55672 22",
        "SSH_CONNECTION": "REDACTED 55672 REDACTED 22",
        "USER": "zuul",
        "XDG_RUNTIME_DIR": "/run/user/1000",
        "XDG_SESSION_ID": "1",
        "_": "/usr/bin/python2"
    },
    "ansible_eth0": {
        "active": true,
        "device": "eth0",
        "ipv4": {
            "address": "REDACTED",
            "broadcast": "REDACTED",
            "netmask": "255.255.255.0",
            "network": "REDACTED"
        },
        "ipv6": [
            {
                "address": "REDACTED",
                "prefix": "64",
                "scope": "link"
            }
        ],
        "macaddress": "REDACTED",
        "module": "xen_netfront",
        "mtu": 1500,
        "pciid": "vif-0",
        "promisc": false,
        "type": "ether"
    },
    "ansible_eth1": {
        "active": true,
        "device": "eth1",
        "ipv4": {
            "address": "REDACTED",
            "broadcast": "REDACTED",
            "netmask": "255.255.224.0",
            "network": "REDACTED"
        },
        "ipv6": [
            {
                "address": "REDACTED",
                "prefix": "64",
                "scope": "link"
            }
        ],
        "macaddress": "REDACTED",
        "module": "xen_netfront",
        "mtu": 1500,
        "pciid": "vif-1",
        "promisc": false,
        "type": "ether"
    },
    "ansible_fips": false,
    "ansible_form_factor": "Other",
    "ansible_fqdn": "centos-7-rax-dfw-0003427354",
    "ansible_hostname": "centos-7-rax-dfw-0003427354",
    "ansible_interfaces": [
        "lo",
        "eth1",
        "eth0"
    ],
    "ansible_is_chroot": false,
    "ansible_kernel": "3.10.0-862.14.4.el7.x86_64",
    "ansible_lo": {
        "active": true,
        "device": "lo",
        "ipv4": {
            "address": "127.0.0.1",
            "broadcast": "host",
            "netmask": "255.0.0.0",
            "network": "127.0.0.0"
        },
        "ipv6": [
            {
                "address": "::1",
                "prefix": "128",
                "scope": "host"
            }
        ],
        "mtu": 65536,
        "promisc": false,
        "type": "loopback"
    },
    "ansible_local": {},
    "ansible_lsb": {
        "codename": "Core",
        "description": "CentOS Linux release 7.5.1804 (Core)",
        "id": "CentOS",
        "major_release": "7",
        "release": "7.5.1804"
    },
    "ansible_machine": "x86_64",
    "ansible_machine_id": "2db133253c984c82aef2fafcce6f2bed",
    "ansible_memfree_mb": 7709,
    "ansible_memory_mb": {
        "nocache": {
            "free": 7804,
            "used": 173
        },
        "real": {
            "free": 7709,
            "total": 7977,
            "used": 268
        },
        "swap": {
            "cached": 0,
            "free": 0,
            "total": 0,
            "used": 0
        }
    },
    "ansible_memtotal_mb": 7977,
    "ansible_mounts": [
        {
            "block_available": 7220998,
            "block_size": 4096,
            "block_total": 9817227,
            "block_used": 2596229,
            "device": "/dev/xvda1",
            "fstype": "ext4",
            "inode_available": 10052341,
            "inode_total": 10419200,
            "inode_used": 366859,
            "mount": "/",
            "options": "rw,seclabel,relatime,data=ordered",
            "size_available": 29577207808,
            "size_total": 40211361792,
            "uuid": "cac81d61-d0f8-4b47-84aa-b48798239164"
        },
        {
            "block_available": 0,
            "block_size": 2048,
            "block_total": 252,
            "block_used": 252,
            "device": "/dev/xvdd",
            "fstype": "iso9660",
            "inode_available": 0,
            "inode_total": 0,
            "inode_used": 0,
            "mount": "/mnt/config",
            "options": "ro,relatime,mode=0700",
            "size_available": 0,
            "size_total": 516096,
            "uuid": "2018-10-25-12-05-57-00"
        }
    ],
    "ansible_nodename": "centos-7-rax-dfw-0003427354",
    "ansible_os_family": "RedHat",
    "ansible_pkg_mgr": "yum",
    "ansible_processor": [
        "0",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "1",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "2",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "3",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "4",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "5",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "6",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz",
        "7",
        "GenuineIntel",
        "Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz"
    ],
    "ansible_processor_cores": 8,
    "ansible_processor_count": 8,
    "ansible_processor_nproc": 8,
    "ansible_processor_threads_per_core": 1,
    "ansible_processor_vcpus": 8,
    "ansible_product_name": "HVM domU",
    "ansible_product_serial": "REDACTED",
    "ansible_product_uuid": "REDACTED",
    "ansible_product_version": "4.1.5",
    "ansible_python": {
        "executable": "/usr/bin/python2",
        "has_sslcontext": true,
        "type": "CPython",
        "version": {
            "major": 2,
            "micro": 5,
            "minor": 7,
            "releaselevel": "final",
            "serial": 0
        },
        "version_info": [
            2,
            7,
            5,
            "final",
            0
        ]
    },
    "ansible_python_version": "2.7.5",
    "ansible_real_group_id": 1000,
    "ansible_real_user_id": 1000,
    "ansible_selinux": {
        "config_mode": "enforcing",
        "mode": "enforcing",
        "policyvers": 31,
        "status": "enabled",
        "type": "targeted"
    },
    "ansible_selinux_python_present": true,
    "ansible_service_mgr": "systemd",
    "ansible_ssh_host_key_ecdsa_public": "REDACTED KEY VALUE",
    "ansible_ssh_host_key_ed25519_public": "REDACTED KEY VALUE",
    "ansible_ssh_host_key_rsa_public": "REDACTED KEY VALUE",
    "ansible_swapfree_mb": 0,
    "ansible_swaptotal_mb": 0,
    "ansible_system": "Linux",
    "ansible_system_capabilities": [
        ""
    ],
    "ansible_system_capabilities_enforced": "True",
    "ansible_system_vendor": "Xen",
    "ansible_uptime_seconds": 151,
    "ansible_user_dir": "/home/zuul",
    "ansible_user_gecos": "",
    "ansible_user_gid": 1000,
    "ansible_user_id": "zuul",
    "ansible_user_shell": "/bin/bash",
    "ansible_user_uid": 1000,
    "ansible_userspace_architecture": "x86_64",
    "ansible_userspace_bits": "64",
    "ansible_virtualization_role": "guest",
    "ansible_virtualization_type": "xen",
    "gather_subset": [
        "all"
    ],
    "module_setup": true
}
```



您可以在模板或 Playbook 中引用上述 facts 中第一个磁盘的型号，如下所示

```
{{ ansible_facts['devices']['xvda']['model'] }}
```



要引用系统主机名

```
{{ ansible_facts['nodename'] }}
```



您可以在条件语句（请参阅 条件语句）和模板中使用 facts。您还可以使用 facts 创建匹配特定条件的动态主机组，请参阅 group_by 模块 文档了解详细信息。



### 收集 facts 的软件包要求

在某些发行版中，您可能会看到缺少 facts 值或 facts 设置为默认值，因为默认情况下未安装支持收集这些 facts 的软件包。您可以使用操作系统软件包管理器在远程主机上安装必要的软件包。已知的依赖项包括

- Linux 网络 facts 收集 - 取决于 ip 二进制文件，通常包含在 iproute2 软件包中。



### 缓存 facts

与注册的变量一样，facts 默认存储在内存中。但是，与注册的变量不同，facts 可以独立收集并缓存以供重复使用。使用缓存的 facts，您可以在配置第二个系统时引用来自一个系统的 facts，即使 Ansible 先在第二个系统上执行当前 Play 也是如此。例如

```
{{ hostvars['asdf.example.com']['ansible_facts']['os_family'] }}
```



缓存由缓存插件控制。默认情况下，Ansible 使用内存缓存插件，该插件在当前 Playbook 运行期间将 facts 存储在内存中。要保留 Ansible facts 以供重复使用，请选择不同的缓存插件。有关详细信息，请参阅 缓存插件。

Fact 缓存可以提高性能。如果您管理数千台主机，您可以配置 fact 缓存以每晚运行，然后在一天中定期管理较小的一组服务器上的配置。使用缓存的 facts，即使您只管理少量服务器，您也可以访问有关所有主机的变量和信息。



### 禁用 facts

默认情况下，Ansible 在每个 Play 的开头收集 facts。如果您不需要收集 facts（例如，如果您知道有关系统的所有信息），您可以在 Play 级别关闭 fact 收集以提高可伸缩性。在具有大量系统的推送模式下，或者如果您在实验性平台上使用 Ansible，禁用 facts 可能会特别提高性能。要禁用 fact 收集

```
- hosts: whatever
  gather_facts: false
```



### 添加自定义 facts

Ansible 中的 setup 模块会自动发现有关每个主机的标准 facts 集。如果您想将自定义值添加到您的 facts 中，您可以编写自定义 facts 模块，使用 ansible.builtin.set_fact 任务设置临时 facts，或使用 facts.d 目录提供永久自定义 facts。



#### facts.d 或本地 facts

1.3 版本中的新增功能。

您可以通过将静态文件添加到 facts.d 来添加静态自定义 facts，或者通过将可执行脚本添加到 facts.d 来添加动态 facts。例如，您可以通过在 facts.d 中创建并运行脚本，将主机上的所有用户列表添加到您的 facts 中。

要使用 facts.d，请在远程主机上创建一个 /etc/ansible/facts.d 目录。如果您喜欢其他目录，请创建它并使用 fact_path Play 关键字指定它。将文件添加到目录中以提供您的自定义 facts。所有文件名必须以 .fact 结尾。这些文件可以是 JSON、INI 或返回 JSON 的可执行文件。

要添加静态 facts，只需添加一个带有 .fact 扩展名的文件。例如，创建 /etc/ansible/facts.d/preferences.fact，其中包含以下内容

```
[general]
asdf=1
bar=2
```



下次 facts 收集运行时，您的 facts 将包含一个名为 general 的哈希变量 fact，其中包含 asdf 和 bar 作为成员。要验证这一点，请运行以下命令

```
ansible <hostname> -m ansible.builtin.setup -a "filter=ansible_local"
```



您将看到您的自定义 fact 已添加

```
{
    "ansible_local": {
        "preferences": {
            "general": {
                "asdf" : "1",
                "bar"  : "2"
            }
        }
    }
}
```



ansible_local 命名空间将 facts.d 创建的自定义 facts 与系统 facts 或在 Playbook 中其他位置定义的变量分开，因此变量不会相互覆盖。您可以在模板或 Playbook 中访问此自定义 fact，如下所示

```
{{ ansible_local['preferences']['general']['asdf'] }}
```



你还可以使用 facts.d 在远程主机上执行脚本，为 ansible_local 命名空间生成动态自定义事实。例如，你可以生成远程主机上所有用户的列表作为该主机的事实。要使用 facts.d 生成动态自定义事实，请执行以下操作：

> 1. 编写并测试一个脚本，以生成你想要的 JSON 数据。
> 2. 将该脚本保存在你的 facts.d 目录中。
> 3. 确保你的脚本具有 .fact 文件扩展名。
> 4. 确保你的脚本可由 Ansible 连接用户执行。
> 5. 收集事实以执行该脚本，并将 JSON 输出添加到 ansible_local 中。

默认情况下，事实收集在每个 playbook 的开始时运行一次。如果你在 playbook 中使用 facts.d 创建自定义事实，它将在下一次收集事实的 play 中可用。如果你想在创建它的同一个 play 中使用它，则必须显式地重新运行 setup 模块。例如：

```
- hosts: webservers
  tasks:

    - name: Create directory for ansible custom facts
      ansible.builtin.file:
        state: directory
        recurse: true
        path: /etc/ansible/facts.d

    - name: Install custom ipmi fact
      ansible.builtin.copy:
        src: ipmi.fact
        dest: /etc/ansible/facts.d

    - name: Re-read facts after adding custom fact
      ansible.builtin.setup:
        filter: ansible_local
```



如果你经常使用这种模式，自定义事实模块将比 facts.d 更有效率。



## 关于 Ansible 的信息：魔法变量

你可以使用“魔法”变量访问有关 Ansible 操作的信息，包括正在使用的 Python 版本、清单中的主机和组以及 playbook 和角色的目录。与连接变量一样，魔法变量是 特殊变量。魔法变量名是保留的 - 不要使用这些名称设置变量。变量 environment 也是保留的。

最常用的魔法变量是 hostvars、groups、group_names 和 inventory_hostname。使用 hostvars，你可以在 playbook 中的任何位置访问为 play 中任何主机定义的变量。你也可以使用 hostvars 变量访问 Ansible 事实，但前提是你已收集（或缓存）了事实。请注意，在 play 对象上定义的变量不是为特定主机定义的，因此不会映射到 hostvars。

如果你想使用另一个节点的事实值或分配给另一个节点的清单变量值来配置你的数据库服务器，你可以在模板或操作行中使用 hostvars。

```
{{ hostvars['test.example.com']['ansible_facts']['distribution'] }}
```



使用 groups，即清单中所有组（和主机）的列表，你可以枚举一个组内的所有主机。例如：

```
{% for host in groups['app_servers'] %}
   # something that applies to all app servers.
{% endfor %}
```



你可以将 groups 和 hostvars 结合使用，以查找一个组中的所有 IP 地址。

```
{% for host in groups['app_servers'] %}
   {{ hostvars[host]['ansible_facts']['eth0']['ipv4']['address'] }}
{% endfor %}
```



你可以使用这种方法将前端代理服务器指向你的应用服务器组中的所有主机，设置服务器之间正确的防火墙规则等等。你必须先缓存事实或收集这些主机的事实，然后再执行填充模板的任务。

使用 group_names，即当前主机所在的所有组的列表（数组），你可以创建基于主机组 membership（或角色）而变化的模板文件。

```
{% if 'webserver' in group_names %}
   # some part of a configuration file that only applies to webservers
{% endif %}
```



当禁用事实收集时，你可以使用魔法变量 inventory_hostname，即清单中配置的主机名，来代替 ansible_hostname。 如果你有一个很长的 FQDN，你可以使用 inventory_hostname_short，它包含第一个句点之前的部分，不包含其余的域。

其他有用的魔法变量引用当前的 play 或 playbook。这些变量对于使用多个主机名填充模板或将列表注入到负载均衡器的规则中非常有用。

ansible_play_hosts 是当前 play 中仍然处于活动状态的所有主机的列表。

ansible_play_batch 是当前 play 的“批次”范围内的主机名列表。

批次大小由 serial 定义，当未设置时，它等同于整个 play（使其与 ansible_play_hosts 相同）。

ansible_playbook_python 是用于调用 Ansible 命令行工具的 Python 可执行文件的路径。

inventory_dir 是保存 Ansible 清单主机文件的目录的路径名。

inventory_file 是指向 Ansible 清单主机文件的路径名和文件名。

playbook_dir 包含 playbook 的基本目录。

role_path 包含当前角色的路径名，并且仅在角色内部有效。

ansible_check_mode 是一个布尔值，如果你使用 --check 运行 Ansible，则设置为 True。



### Ansible 版本

版本 1.8 中的新功能。

要使 playbook 的行为适应不同版本的 Ansible，你可以使用变量 ansible_version，它具有以下结构：

```
{
    "ansible_version": {
        "full": "2.10.1",
        "major": 2,
        "minor": 10,
        "revision": 1,
        "string": "2.10.1"
    }
}
```
