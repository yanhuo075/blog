---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 3.2 动态清单
order: 1
---

# 使用动态清单

如果你的Ansible清单随时间变化，主机根据业务需求启动和关闭，则如何构建你的清单中描述的静态清单解决方案将无法满足你的需求。你可能需要跟踪来自多个来源的主机：云提供商、LDAP、Cobbler和/或企业CMDB系统。

Ansible通过动态外部清单系统集成了所有这些选项。Ansible支持两种连接外部清单的方式：清单插件和清单脚本。

清单插件利用了Ansible Core代码的最新更新。我们推荐使用插件而不是脚本进行动态清单管理。你可以编写你自己的插件来连接到额外的动态清单源。

如果你选择，你仍然可以使用清单脚本。当我们实现清单插件时，我们通过脚本清单插件确保了向后兼容性。下面的示例说明了如何使用清单脚本。

如果你更喜欢使用GUI来处理动态清单，则AWX或Red Hat Ansible Automation Platform上的清单数据库与所有动态清单源同步，提供对结果的Web和REST访问，并提供图形化清单编辑器。有了所有主机数据库记录，你可以关联过去的事件历史，并查看哪些主机在上次Playbook运行时出现故障。



## 清单脚本示例：Cobbler

Ansible与Cobbler无缝集成，Cobbler是一个最初由Michael DeHaan编写，现在由在Ansible工作的James Cammarata领导的Linux安装服务器。

Cobbler主要用于启动操作系统安装和管理DHCP和DNS，但它还有一个通用层，可以表示多个配置管理系统的数据（甚至同时），并充当“轻量级CMDB”。

要将你的Ansible清单与Cobbler绑定，请将此脚本复制到/etc/ansible并使用chmod +x为文件设置权限。在每次使用Ansible时运行cobblerd，并使用-i命令行选项（例如，-i /etc/ansible/cobbler.py）通过Cobbler的XMLRPC API与Cobbler通信。

在/etc/ansible中添加一个cobbler.ini文件，以便Ansible知道Cobbler服务器在哪里，并可以使用一些缓存改进。例如：

```
[cobbler]

# Set Cobbler's hostname or IP address
host = http://127.0.0.1/cobbler_api

# API calls to Cobbler can be slow. For this reason, we cache the results of an API
# call. Set this to the path you want cache files to be written to. Two files
# will be written to this directory:
#   - ansible-cobbler.cache
#   - ansible-cobbler.index

cache_path = /tmp

# The number of seconds a cache file is considered valid. After this many
# seconds, a new API call will be made, and the cache file will be updated.

cache_max_age = 900
```



首先通过直接运行/etc/ansible/cobbler.py来测试脚本。你应该会看到一些JSON数据输出，但它可能暂时没有任何内容。

让我们来看看它的作用。在Cobbler中，假设一个类似于以下的场景：

```
cobbler profile add --name=webserver --distro=CentOS6-x86_64
cobbler profile edit --name=webserver --mgmt-classes="webserver" --ksmeta="a=2 b=3"
cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta" --ksmeta="c=4"
cobbler system edit --name=bar --dns-name="bar.example.com" --mgmt-classes="atlanta" --ksmeta="c=5"
```



在上面的示例中，系统“foo.example.com”可以直接被ansible寻址，但在使用组名“webserver”或“atlanta”时也可以寻址。由于Ansible使用SSH，它只通过“foo.example.com”联系系统foo，而不是只使用“foo”。同样，如果你尝试“ansible foo”，它将找不到系统……但“ansible ‘foo*’”可以，因为系统DNS名称以“foo”开头。

该脚本提供的不仅仅是主机和组信息。此外，作为额外奖励，当运行“setup”模块时（在使用playbook时会自动发生），变量“a”、“b”和“c”将在模板中自动填充。

```
# file: /srv/motd.j2
Welcome, I am templated with a value of a={{ a }}, b={{ b }}, and c={{ c }}
```



这可以像这样执行：

```
ansible webserver -m setup
ansible webserver -m template -a "src=/tmp/motd.j2 dest=/etc/motd"
```



因此，使用上面的模板（motd.j2），这将导致以下数据被写入系统“foo”的/etc/motd：

```
Welcome, I am templated with a value of a=2, b=3, and c=4
```



以及系统“bar”（bar.example.com）：

```
Welcome, I am templated with a value of a=2, b=3, and c=5
```



从技术上讲，虽然没有充分的理由这样做，但这也能工作：

```
ansible webserver -m ansible.builtin.shell -a "echo {{ a }}"
```



换句话说，你也可以在参数/操作中使用这些变量。



## 其他清单脚本

在Ansible 2.10及更高版本中，清单脚本已移至其关联的集合。许多现在位于ansible-community/contrib-scripts存储库。我们建议你改用清单插件。



## 使用清单目录和多个清单源

如果在Ansible中给-i提供的位置是一个目录（或在ansible.cfg中如此配置），Ansible可以同时使用多个清单源。这样做时，可以在同一个ansible运行中混合使用动态和静态管理的清单源。即时混合云！

在清单目录中，可执行文件被视为动态清单源，大多数其他文件被视为静态源。以以下任何结尾的文件将被忽略：

```
~, .orig, .bak, .ini, .cfg, .retry, .pyc, .pyo
```



你可以通过在ansible.cfg中配置inventory_ignore_extensions列表或设置ANSIBLE_INVENTORY_IGNORE环境变量来替换此列表。在这两种情况下，值都必须是一个逗号分隔的模式列表，如上所示。

清单目录中的任何group_vars和host_vars子目录都将按预期解释，这使得清单目录成为组织不同配置集的强大方法。有关更多信息，请参阅传递多个清单源。



## 静态组的动态组

在静态清单文件中定义组的组时，子组也必须在静态清单文件中定义，否则ansible将返回错误。如果要定义动态子组的静态组，请在静态清单文件中将动态组定义为空。例如：

```
[tag_Name_staging_foo]

[tag_Name_staging_bar]

[staging:children]
tag_Name_staging_foo
tag_Name_staging_bar
```
