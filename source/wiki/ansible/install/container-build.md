---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 2.3 在容器中安装
order: 2
---

# 在容器中安装 Ansible

您可以直接构建一个执行环境容器镜像，或使用现有的社区镜像作为您的控制节点，而不是手动安装 Ansible 内容。有关详细信息，请参阅 执行环境入门。



## 开发环境安装

如果您正在测试新功能、修复错误或与开发团队一起处理核心代码的更改，您可以从 GitHub 安装并运行源代码。

有关参与 Ansible 项目的更多信息，请参阅 [Ansible 社区指南](https://docs.ansible.org.cn/ansible/latest/community/index.html#ansible-community-guide)。

有关创建 Ansible 模块和集合的更多信息，请参阅 [开发者指南](https://docs.ansible.org.cn/ansible/latest/dev_guide/index.html#developer-guide)。



## 执行环境简介

Ansible 执行环境旨在解决复杂性问题，并提供容器化技术的所有优势。

## 降低复杂性

EE 可以降低三个主要方面的复杂性

- 软件依赖项
- 可移植性
- 内容分离

### 依赖项

软件应用程序通常具有依赖项，Ansible 也不例外。这些依赖项可能包括软件库、配置文件或其他服务等等。

传统上，管理员使用 RPM 或 Python-pip 等包管理工具在操作系统之上安装应用程序依赖项。这种方法的主要缺点是应用程序可能需要与默认提供的依赖项版本不同的依赖项版本。对于 Ansible，典型的安装包括ansible-core 和一组 Ansible 集合。许多集合都对其提供的插件、模块、角色和剧本有依赖项。

Ansible 集合可能依赖于以下软件及其版本：

- `ansible-core`
- Python
- Python 包
- 系统包
- 其他 Ansible 集合

这些依赖项必须安装，有时可能会彼此冲突。

一种**部分**解决依赖项问题的方法是在 Ansible 控制节点上使用 Python 虚拟环境。但是，应用于 Ansible 时，虚拟环境具有缺点和固有的局限性。

### 可移植性

Ansible 用户在本地编写 Ansible 内容，并希望利用容器技术使其自动化运行时可移植、可共享并易于部署到测试和生产环境。

### 内容分离

当多个用户使用 Ansible 控制节点或 Ansible AWX/Controller 等工具时，他们可能希望分离其内容以避免配置和依赖项冲突。



## Ansible 的 EE 工具

Ansible 生态系统中的项目还提供了一些可与 EE 一起使用的工具，例如：

- [Ansible Builder](https://ansible-builder.readthedocs.io/en/stable/)
- [Ansible Navigator](https://ansible-navigator.readthedocs.io/)
- [Ansible AWX](https://readthedocs.ansible.org.cn/projects/awx/en/latest/userguide/execution_environments.html#use-an-execution-environment-in-jobs)
- [Ansible Runner](https://ansible-runner.readthedocs.io/en/stable/)
- [VS Code Ansible](https://marketplace.visualstudio.com/items?itemName=redhat.ansible)
- [Dev Containers 扩展](https://vscode.js.cn/docs/devcontainers/containers)



## 设置您的环境

完成以下步骤以设置您第一个执行环境的本地环境

1. 确保您的系统上安装了以下软件包

   > - `podman` 或 `docker`
   > - `python3`
   > - `python3-pip`
   >
   > 如果您使用 DNF 包管理器，请按如下方式安装这些先决条件：
   >
   > ```
   > sudo dnf install -y podman python3 python3-pip
   > ```
   >
   > 

2. 安装 `ansible-navigator`

   > ```
   > pip3 install ansible-navigator
   > ```
   >
   > 
   >
   > 安装 `ansible-navigator` 使您可以通过命令行运行执行环境。它包含 `ansible-builder` 包来构建执行环境。
   >
   > 如果您想在不测试的情况下构建执行环境，只需安装 `ansible-builder`。
   >
   > ```
   > pip3 install ansible-builder
   > ```
   >
   > 

3. 使用以下命令验证您的环境：

   > ```
   > ansible-navigator --version
   > ansible-builder --version
   > ```



## 构建你的第一个执行环境

我们将构建一个代表 Ansible 控制节点的执行环境 (EE)，其中包含标准软件包，例如`ansible-core`和 Python，以及 Ansible 集合（`community.postgresql`）及其依赖项（`psycopg2-binary` Python 连接器）。

构建你的第一个执行环境

1. 在你的文件系统上创建一个项目文件夹。

   ```
   mkdir my_first_ee && cd my_first_ee
   ```

   

2. 创建一个`execution-environment.yml`文件，指定要包含在镜像中的依赖项。

   ```
   version: 3
   
   images:
     base_image:
       name: quay.io/fedora/fedora:39
   
   dependencies:
     ansible_core:
       package_pip: ansible-core
     ansible_runner:
       package_pip: ansible-runner
     system:
     - openssh-clients
     - sshpass
     galaxy:
       collections:
       - name: community.postgresql
   ```

   

3. 构建一个名为`postgresql_ee`的执行环境容器镜像。

   如果你使用 docker，请添加`--container-runtime docker`参数。

   ```
   ansible-builder build --tag postgresql_ee
   ```

   

4. 列出容器镜像以验证你是否成功构建它。

   ```
   podman image list
   
   localhost/postgresql_ee          latest      2e866777269b  6 minutes ago  1.11 GB
   ```

   

你可以通过检查`context`目录中的`Containerfile`或`Dockerfile`来查看其配置，从而验证你创建的镜像。

```
less context/Containerfile
```



你也可以使用 Ansible Navigator 查看有关镜像的详细信息。

运行ansible-navigator命令，在 TUI 中键入`:images`，然后选择`postgresql_ee`。

继续运行你的执行环境并测试你刚刚构建的执行环境。



## 运行您的执行环境

您可以使用 ansible-navigator 在命令行中针对 localhost 或远程目标运行您的执行环境。

### 针对本地主机运行

1. 创建一个 test_localhost.yml 剧本。

   ```
   - name: Gather and print local facts
     hosts: localhost
     become: true
     gather_facts: true
     tasks:
   
      - name: Print facts
        ansible.builtin.debug:
         var: ansible_facts
   ```

   

2. 在 postgresql_ee 执行环境中运行该剧本。

   ```
   ansible-navigator run test_localhost.yml --execution-environment-image postgresql_ee --mode stdout --pull-policy missing --container-options='--user=0'
   ```

您可能会注意到收集的事实信息是关于容器的，而不是开发人员机器的。这是因为 Ansible 剧本是在容器内运行的。



### 针对远程目标运行

在开始之前，请确保您拥有以下内容：

> - 至少一个远程目标的 IP 地址或可解析的主机名。
> - 远程主机的有效凭据。
> - 远程主机上具有 sudo 权限的用户。

如以下示例所示，在 postgresql_ee 执行环境中针对远程主机执行剧本

1. 为清单文件创建一个目录。

   ```
   mkdir inventory
   ```

   

2. 在 inventory 目录中创建 hosts.yml 清单文件。

   ```
   all:
     hosts:
       192.168.0.2  # Replace with the IP of your target host
   ```

   

3. 创建一个 test_remote.yml 剧本。

   ```
   - name: Gather and print facts
     hosts: all
     become: true
     gather_facts: true
     tasks:
   
      - name: Print facts
        ansible.builtin.debug:
         var: ansible_facts
   ```

   

4. 在 postgresql_ee 执行环境中运行该剧本。

   将 student 替换为适当的用户名。根据您的目标主机身份验证方法，命令中的一些参数可以是可选的。

   ```
   ansible-navigator run test_remote.yml -i inventory --execution-environment-image postgresql_ee:latest --mode stdout --pull-policy missing --enable-prompts -u student -k -K
   ```



## 使用社区 EE 镜像运行 Ansible

无需构建自定义 EE，你就可以使用社区镜像运行 Ansible。

使用包含仅 `ansible-core` 的 `community-ee-minimal` 镜像，或者也包含多个基础集合的 `community-ee-base` 镜像。运行以下命令查看 `community-ee-base` 镜像中包含的集合

```
ansible-navigator collections --execution-environment-image ghcr.io/ansible-community/community-ee-base:latest
```



在 `community-ee-minimal` 容器内针对 localhost 运行以下 Ansible ad-hoc 命令

```
ansible-navigator exec "ansible localhost -m setup" --execution-environment-image ghcr.io/ansible-community/community-ee-minimal:latest --mode stdout
```



现在，创建一个简单的测试 playbook 并针对容器内的 `localhost` 运行它

```
- name: Gather and print local facts
  hosts: localhost
  become: true
  gather_facts: true
  tasks:

   - name: Print facts
     ansible.builtin.debug:
      var: ansible_facts
```



```
ansible-navigator run test_localhost.yml --execution-environment-image ghcr.io/ansible-community/community-ee-minimal:latest --mode stdout
```
