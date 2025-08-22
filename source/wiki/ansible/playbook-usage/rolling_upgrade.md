---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.20 持续交付和滚动升级
order: 19
---

# 剧本示例：持续交付和滚动升级

- 什么是持续交付？
- 站点部署
- 可重用内容：角色
- 配置：组变量
- 滚动升级
- 管理其他负载均衡器
- 端到端持续交付



## 什么是持续交付？

持续交付 (CD) 意味着频繁地向您的软件应用程序交付更新。

其理念是，通过更频繁地更新，您无需等待特定时间段，并且您的组织在响应变化方面会变得更好。

一些 Ansible 用户每小时甚至更频繁地向其最终用户部署更新——有时每次有经过批准的代码更改时都会部署。为实现此目的，您需要能够以零停机时间快速应用这些更新的工具。

本文档详细介绍了如何使用 Ansible 最完整的示例剧本之一 lamp_haproxy 作为模板来实现此目标。此示例使用了许多 Ansible 功能：角色、模板和组变量，并且还附带了一个编排剧本，可以对 Web 应用程序堆栈执行零停机时间滚动升级。

这些剧本将 Apache、PHP、MySQL、Nagios 和 HAProxy 部署到一组基于 CentOS 的服务器上。

我们在这里不会介绍如何运行这些剧本。请阅读 GitHub 项目中包含的 README 以及该信息的相关示例。相反，我们将仔细查看剧本的每个部分并描述其功能。



## 站点部署

让我们从 site.yml 开始。这是我们的站点范围部署剧本。它可用于最初部署站点，以及将更新推送到所有服务器。

```
---
# This playbook deploys the whole application stack in this site.

# Apply common configuration to all hosts
- hosts: all

  roles:
  - common

# Configure and deploy database servers.
- hosts: dbservers

  roles:
  - db

# Configure and deploy the web servers. Note that we include two roles
# here, the 'base-apache' role which simply sets up Apache, and 'web'
# which includes our example web application.

- hosts: webservers

  roles:
  - base-apache
  - web

# Configure and deploy the load balancer(s).
- hosts: lbservers

  roles:
  - haproxy

# Configure and deploy the Nagios monitoring node(s).
- hosts: monitoring

  roles:
  - base-apache
  - nagios
```



在这个剧本中，我们有 5 个剧目。第一个剧目针对 all 主机并将 common 角色应用于所有主机。这是针对站点范围的事情，例如 yum 存储库配置、防火墙配置以及需要应用于所有服务器的任何其他内容。

接下来的四个剧目针对特定主机组，并将特定角色应用于这些服务器。除了 Nagios 监控、数据库和 Web 应用程序的角色外，我们还实现了一个 base-apache 角色，用于安装和配置基本的 Apache 设置。这由示例 Web 应用程序和 Nagios 主机使用。



## 可重用内容：角色

现在您应该对角色及其在 Ansible 中的工作方式有所了解。角色是一种将内容（任务、处理器、模板和文件）组织成可重用组件的方法。

此示例有六个角色：common、base-apache、db、haproxy、nagios 和 web。如何组织角色取决于您和您的应用程序，但大多数站点将拥有一个或多个应用于所有系统的公共角色，然后是一系列特定于应用程序的角色，用于安装和配置站点的特定部分。

角色可以具有变量和依赖项，您可以向角色传递参数以修改其行为。您可以在 角色 部分中阅读有关角色的更多信息。



## 配置：组变量

组变量是应用于服务器组的变量。它们可以在模板和剧本中使用，以自定义行为并提供易于更改的设置和参数。它们存储在与清单位于同一位置的名为 group_vars 的目录中。以下是 lamp_haproxy 的 group_vars/all 文件。正如您可能预期的那样，这些变量将应用于清单中的所有机器。

```
---
httpd_port: 80
ntpserver: 192.0.2.23
```



这是一个 YAML 文件，您可以创建列表和字典以获得更复杂的变量结构。在这种情况下，我们只设置了两个变量，一个用于 Web 服务器的端口，另一个用于机器应使用的时间同步的 NTP 服务器。

这是另一个组变量文件。这是 group_vars/dbservers，它应用于 dbservers 组中的主机。

```
---
mysqlservice: mysqld
mysql_port: 3306
dbuser: root
dbname: foodb
upassword: usersecret
```



如果您查看示例，就会发现类似地，还有针对 webservers 组和 lbservers 组的组变量。

这些变量在许多地方使用。您可以在剧本中使用它们，如下所示，在 roles/db/tasks/main.yml 中。

```
- name: Create Application Database
  mysql_db:
    name: "{{ dbname }}"
    state: present

- name: Create Application DB User
  mysql_user:
    name: "{{ dbuser }}"
    password: "{{ upassword }}"
    priv: "*.*:ALL"
    host: '%'
    state: present
```



您也可以在模板中使用这些变量，如下所示，在 roles/common/templates/ntp.conf.j2 中。

```
driftfile /var/lib/ntp/drift

restrict 127.0.0.1
restrict -6 ::1

server {{ ntpserver }}

includefile /etc/ntp/crypto/pw

keys /etc/ntp/keys
```



您可以看到，{{ 和 }} 的变量替换语法对于模板和变量都是相同的。大括号内的语法是 Jinja2，您可以对其中的数据进行各种操作并应用不同的过滤器。在模板中，您还可以使用 for 循环和 if 语句来处理更复杂的情况，如下所示，在 roles/common/templates/iptables.j2 中。

```
{% if inventory_hostname in groups['dbservers'] %}
-A INPUT -p tcp  --dport 3306 -j  ACCEPT
{% endif %}
```



这是测试我们当前正在操作的机器的清单名称 (inventory_hostname) 是否存在于清单组 dbservers 中。如果是，则该机器将获得端口 3306 的 iptables ACCEPT 行。

这是来自同一模板的另一个示例。

```
{% for host in groups['monitoring'] %}
-A INPUT -p tcp -s {{ hostvars[host].ansible_default_ipv4.address }} --dport 5666 -j ACCEPT
{% endfor %}
```



这将循环遍历名为 monitoring 的组中的所有主机，并将每个监控主机的默认 IPv4 地址的 ACCEPT 行添加到当前机器的 iptables 配置中，以便 Nagios 可以监控这些主机。

您可以在这里了解更多关于 Jinja2 及其功能的信息 here，并且您可以在 使用变量 部分中阅读更多关于 Ansible 变量的信息。



## 滚动升级

现在您拥有一个完全部署的站点，其中包含 Web 服务器、负载均衡器和监控。如何更新它？这就是 Ansible 的编排功能发挥作用的地方。虽然某些应用程序使用术语“编排”来表示基本的排序或命令爆炸，但 Ansible 将编排称为“像管弦乐队一样指挥机器”，并且拥有一个相当复杂的引擎来实现它。

Ansible 能够以协调的方式对多层应用程序进行操作，从而可以轻松地编排我们 Web 应用程序的复杂零停机时间滚动升级。这在名为 rolling_update.yml 的单独剧本中实现。

查看剧本，您可以看到它由两个剧目组成。第一个剧目非常简单，如下所示：

```
- hosts: monitoring
  tasks: []
```



这里发生了什么，为什么没有任务？您可能知道，Ansible会在操作服务器之前收集服务器的“事实”（facts）。这些事实对于各种事情都很有用：网络信息、操作系统/发行版版本等等。在本例中，我们需要在执行更新之前了解环境中所有监控服务器的一些信息，因此这个简单的 playbook 强制在我们的监控服务器上执行事实收集步骤。您有时会看到这种模式，这是一个有用的技巧。

接下来是更新 playbook。第一部分如下所示

```
- hosts: webservers
  user: root
  serial: 1
```



这只是一个普通的 playbook 定义，作用于webservers组。 serial关键字告诉 Ansible 每次操作多少台服务器。如果未指定，Ansible 将并行化这些操作，直到配置文件中指定的默认“forks”限制。但是对于零停机滚动升级，您可能不想一次操作那么多主机。如果您只有少数几台 web 服务器，您可能希望将serial设置为 1，一次一台主机。如果您有 100 台，也许您可以将serial设置为 10，一次十台。

这是更新 playbook 的下一部分

```
pre_tasks:
- name: disable nagios alerts for this host webserver service
  nagios:
    action: disable_alerts
    host: "{{ inventory_hostname }}"
    services: webserver
  delegate_to: "{{ item }}"
  loop: "{{ groups.monitoring }}"

- name: disable the server in haproxy
  shell: echo "disable server myapplb/{{ inventory_hostname }}" | socat stdio /var/lib/haproxy/stats
  delegate_to: "{{ item }}"
  loop: "{{ groups.lbservers }}"
```



pre_tasks关键字允许您列出在调用角色之前要运行的任务。过一会儿这就会更有意义。如果您查看这些任务的名称，您会发现我们正在禁用 Nagios 警报，然后从 HAProxy 负载均衡池中删除我们当前正在更新的 web 服务器。

delegate_to和loop参数一起使用，会导致 Ansible 遍历每个监控服务器和负载均衡器，并在监控或负载均衡服务器上执行该操作（委托该操作），“代表”web 服务器。用编程术语来说，外循环是 web 服务器列表，内循环是监控服务器列表。

请注意，HAProxy 步骤看起来有点复杂。我们在此示例中使用 HAProxy 是因为它可以免费使用，但是如果您在基础设施中拥有（例如）F5 或 Netscaler（或者您可能设置了 AWS 弹性 IP？），您可以使用 Ansible 模块与它们进行通信。您可能还想使用其他监控模块而不是 nagios，但这只是展示了“pre tasks”部分的主要目标——将服务器从监控中移除，并将其从轮换中移除。

下一步只需将正确的角色重新应用于 web 服务器。这将导致web和base-apache角色中的任何配置管理声明都应用于 web 服务器，包括 web 应用程序代码本身的更新。我们不必这样做——我们也可以只更新 web 应用程序，但这是一个很好的例子，说明如何使用角色来重用任务。

```
roles:
- common
- base-apache
- web
```



最后，在post_tasks部分，我们反转对 Nagios 配置的更改，并将 web 服务器放回负载均衡池。

```
post_tasks:
- name: Enable the server in haproxy
  shell: echo "enable server myapplb/{{ inventory_hostname }}" | socat stdio /var/lib/haproxy/stats
  delegate_to: "{{ item }}"
  loop: "{{ groups.lbservers }}"

- name: re-enable nagios alerts
  nagios:
    action: enable_alerts
    host: "{{ inventory_hostname }}"
    services: webserver
  delegate_to: "{{ item }}"
  loop: "{{ groups.monitoring }}"
```



同样，如果您使用的是 Netscaler 或 F5 或 Elastic Load Balancer，您只需替换相应的模块即可。



## 管理其他负载均衡器

在此示例中，我们使用简单的 HAProxy 负载均衡器作为 web 服务器的前端。它易于配置和管理。正如我们提到的，Ansible 支持各种其他负载均衡器，如 Citrix NetScaler、F5 BigIP、Amazon Elastic Load Balancers 等等。

对于其他负载均衡器，您可能需要向它们发送 shell 命令（就像我们上面对 HAProxy 所做的那样），或者调用 API（如果您的负载均衡器公开了一个 API）。对于 Ansible 具有模块的负载均衡器，如果它们联系 API，您可能希望将其作为local_action运行。您可以在控制任务运行位置：委托和本地操作部分阅读有关本地操作的更多信息。如果您为某些没有模块的硬件开发了一些有趣的东西，它可能会成为一个很好的贡献！



## 端到端持续交付

既然您已经有了自动化部署应用程序更新的方法，您如何将所有这些结合在一起？许多组织使用持续集成工具，如Jenkins或Atlassian Bamboo将开发、测试、发布和部署步骤结合在一起。您可能还想使用Gerrit之类的工具向对应用程序代码本身或 Ansible playbook 或两者的提交添加代码审查步骤。

根据您的环境，您可能正在持续部署到测试环境，对该环境运行集成测试电池，然后自动部署到生产环境。或者您可以保持简单，只需使用滚动更新即可按需部署到测试或生产环境。这完全取决于您。

对于与持续集成系统的集成，您可以使用ansible-playbook命令行工具轻松触发 playbook 运行，或者，如果您使用的是 AWX，则可以使用tower-cli命令或内置 REST API。（tower-cli 命令“joblaunch”将在 REST API 上生成一个远程作业，并且非常巧妙）。

这应该让您很好地了解如何使用 Ansible 构建多层应用程序并协调对该应用程序的操作，最终目标是持续交付给您的客户。您可以将滚动升级的想法扩展到应用程序的几个不同部分；也许可以添加前端 web 服务器以及应用程序服务器，或者用 NoSQL 数据库替换 SQL 数据库。Ansible 使您能够轻松管理复杂的环境并自动化常见操作。
