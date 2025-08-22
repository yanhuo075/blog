---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 2.2 在特定操作系统上安装
order: 1
---

# 在特定操作系统上安装 Ansible

`ansible`包始终可以通过[使用 pip 从 PyPI 安装](https://docs.ansible.org.cn/ansible/latest/installation_guide/intro_installation.html#intro-installation-guide)在大多数系统上安装，但它也由社区为各种 Linux 发行版打包和维护。

本文档指导您通过不同发行版的软件包存储库安装 Ansible。

要在此指南中添加另一个发行版的说明，软件包维护者**必须**执行以下操作

> - 确保发行版提供`ansible`的相当新的版本。
> - 确保`ansible-core`和`ansible`版本在构建系统允许的范围内保持同步。
> - 提供一种联系发行版维护人员的方法作为说明的一部分。还鼓励发行版维护人员加入[Ansible Packaging](https://matrix.to/#/#packaging:ansible.com) Matrix 聊天室。

- [在 Fedora Linux 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-fedora-linux)
- [从 EPEL 安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-from-epel)
- [在 OpenSUSE Tumbleweed/Leap 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-opensuse-tumbleweed-leap)
- [在 Ubuntu 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu)
- [在 Debian 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian)
- [在 Arch Linux 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-arch-linux)
- [在 Windows 上安装 Ansible](https://docs.ansible.org.cn/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-windows)



## 在 Fedora Linux 上安装 Ansible

要安装完整的`ansible`包，请运行

```
$ sudo dnf install ansible
```



要安装最小的`ansible-core`包，请运行

```
$ sudo dnf install ansible-core
```



Fedora 存储库中还提供了一些 Ansible 集合，作为用户可以与`ansible-core`一起安装的独立包。例如，要安装`community.general`集合，请运行

```
$ sudo dnf install ansible-collection-community-general
```



有关 Fedora 中打包的 Ansible 集合的完整列表，请参阅[Fedora 软件包索引](https://packages.fedoraproject.org/search?query=ansible-collection)。

请[在 Red Hat Bugzilla 中针对`Fedora`产品提交错误报告](https://bugzilla.redhat.com/enter_bug.cgi)以联系软件包维护人员。



## 从 EPEL 安装 Ansible

CentOS Stream、Almalinux、Rocky Linux 和相关发行版的用户可以从社区维护的[EPEL](https://docs.fedoraproject.org/en-US/epel/)（企业 Linux 的额外软件包）存储库安装`ansible`或 Ansible 集合。

在[启用 EPEL 存储库](https://docs.fedoraproject.org/en-US/epel/#_quickstart)后，用户可以使用与 Fedora Linux 相同的`dnf`命令。

请[在 Red Hat Bugzilla 中针对`Fedora EPEL`产品提交错误报告](https://bugzilla.redhat.com/enter_bug.cgi)以联系软件包维护人员。



## 在 OpenSUSE Tumbleweed/Leap 上安装 Ansible

```
$ sudo zypper install ansible
```



有关在 OpenSUSE 上使用 Ansible 的更多帮助，请参阅[OpenSUSE 支持门户](https://en.opensuse.org/Portal:Support)。



## 在 Ubuntu 上安装 Ansible

Ubuntu 构建可在此处[PPA 中获得](https://launchpad.net/~ansible/+archive/ubuntu/ansible)。

要配置系统上的 PPA 并安装 Ansible，请运行以下命令

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

在[PPA 的问题跟踪器中](https://github.com/ansible-community/ppa/issues)提交任何问题。



## 在 Debian 上安装 Ansible

虽然 Ansible 可从[主要的 Debian 存储库](https://packages.debian.org/stable/ansible)获得，但它可能已过时。

要获得较新的版本，Debian 用户可以使用 Ubuntu PPA，方法如下表所示

| Debian               |      | Ubuntu                | UBUNTU_CODENAME |
| -------------------- | ---- | --------------------- | --------------- |
| Debian 12 (Bookworm) | ->   | Ubuntu 22.04 (Jammy)  | `jammy`         |
| Debian 11 (Bullseye) | ->   | Ubuntu 20.04 (Focal)  | `focal`         |
| Debian 10 (Buster)   | ->   | Ubuntu 18.04 (Bionic) | `bionic`        |

在以下示例中，我们假设您已安装 wget 和 gpg（`sudo apt install wget gpg`）。

运行以下命令以添加存储库并安装 Ansible。根据上表设置`UBUNTU_CODENAME=...`（在此示例中，我们使用`jammy`）。

```
$ UBUNTU_CODENAME=jammy
$ wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
$ echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
$ sudo apt update && sudo apt install ansible
```



注意：密钥服务器 URL 周围的“ ”很重要。在“echo deb”周围，使用“ ”而不是“ ' ”很重要。

这些命令下载签名密钥并向 apt 的源添加一个条目，指向 PPA。

以前，您可能使用了`apt-key add`。由于安全原因，这现在已[弃用](https://manpages.debian.org/testing/apt/apt-key.8.en.html)（在 Debian、Ubuntu 等）。有关更多详细信息，请参阅[此 AskUbuntu 帖子](https://askubuntu.com/a/1307181)。另请注意，出于安全原因，我们不会将密钥添加到`/etc/apt/trusted.gpg.d/`或`/etc/apt/trusted.gpg`，在那里它将被允许签署来自任何存储库的版本。



## 在 Arch Linux 上安装 Ansible

要安装完整的`ansible`包，请运行

```
$ sudo pacman -S ansible
```



要安装最小的`ansible-core`包，请运行

```
$ sudo pacman -S ansible-core
```



Arch Linux 存储库中还提供了一些 Ansible 生态系统软件包，作为用户可以与`ansible-core`一起安装的独立包。有关 Arch Linux 中 Ansible 软件包的完整列表，请参阅[Arch Linux 软件包索引](https://archlinux.org.cn/packages/?sort=&q=ansible)。

请[在相关的软件包 GitLab 存储库中打开一个问题](https://gitlab.archlinux.org/archlinux/packaging/packages)以联系软件包维护人员。



## 在 Windows 上安装 Ansible

您不能使用 Windows 系统作为 Ansible 控制节点。请参阅[Ansible 能在 Windows 上运行吗？](https://docs.ansible.org.cn/ansible/latest/os_guide/windows_faq.html#windows-faq-ansible)
