---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 2.4 使用pipx安装和升级
order: 3
---

## 使用 pipx 安装和升级 Ansible

在某些系统上，由于操作系统开发人员做出的决定，可能无法使用 pip 安装 Ansible。在这种情况下，pipx 是一个广泛可用的替代方案。

这些说明不会介绍安装 pipx 的步骤；如果需要这些说明，请继续查看 pipx 安装说明 以了解更多信息。



### 安装 Ansible

在您的环境中使用 `pipx` 安装完整的 Ansible 软件包

```
pipx install --include-deps ansible
```



您可以安装最小的 `ansible-core` 软件包

```
pipx install ansible-core
```



或者，您可以安装特定版本的 `ansible-core`

```
pipx install ansible-core==2.12.3
```



### 升级 Ansible

要将现有 Ansible 安装升级到最新的发行版本

```
pipx upgrade --include-injected ansible
```



### 安装额外的 Python 依赖项

要安装可能需要的其他 python 依赖项，例如安装下面描述的 `argcomplete` python 包

```
pipx inject ansible argcomplete
```



包含 `--include-apps` 选项可在您的 PATH 上使用其他 python 依赖项中的应用程序。这允许您从 shell 执行这些应用程序的命令。

```
pipx inject --include-apps ansible argcomplete
```



## 使用 pip 安装和升级 Ansible

### 查找 Python

找到并记住您希望用来运行 Ansible 的 Python 解释器的路径。以下说明将此 Python 称为 python3。例如，如果您确定要使用 /usr/bin/python3.9 中的 Python 来安装 Ansible，请指定它而不是 python3。

### 确保 pip 可用

要验证您首选的 Python 是否已安装 `pip`

```
python3 -m pip -V
```



如果一切顺利，您应该会看到如下内容：

```
python3 -m pip -V
pip 21.0.1 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```



如果是这样，pip 可用，您可以继续执行 下一步。

如果您看到类似 `No module named pip` 的错误，则需要在继续操作之前，在您选择的 Python 解释器下安装 `pip`。这可能意味着安装额外的操作系统软件包（例如，`python3-pip`），或直接从 Python Packaging Authority 安装最新的 `pip`，方法是运行以下命令：

```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
```



您可能需要进行一些额外的配置才能运行 Ansible。有关更多信息，请参阅 Python 文档中关于 [安装到用户站点](https://packaging.pythonlang.cn/tutorials/installing-packages/#installing-to-the-user-site) 的内容。



### 安装 Ansible

在您选择的 Python 环境中使用 pip 为当前用户安装完整的 Ansible 软件包：

```
python3 -m pip install --user ansible
```



您可以为当前用户安装最小化的 ansible-core 软件包：

```
python3 -m pip install --user ansible-core
```



或者，您可以安装特定版本的 ansible-core

```
python3 -m pip install --user ansible-core==2.12.3
```



### 升级 Ansible

要将此 Python 环境中现有的 Ansible 安装升级到最新版本，只需在上面的命令中添加 --upgrade：

```
python3 -m pip install --upgrade --user ansible
```





### 使用 pip 从 GitHub 安装 devel 分支

您可以使用 pip 直接从 GitHub 安装 ansible-core 的 devel 分支：

```
python3 -m pip install --user https://github.com/ansible/ansible/archive/devel.tar.gz
```



您可以将上面提到的 URL 中的 devel 替换为 GitHub 上的任何其他分支或标签，以安装旧版本的 Ansible、标记为 alpha 或 beta 版本以及候选版本。



### 从克隆运行 devel 分支

ansible-core 很容易从源代码运行。您不需要 root 权限即可使用它，也不需要实际安装任何软件。不需要守护进程或数据库设置。

1. 克隆 ansible-core 仓库：

   ```
   git clone https://github.com/ansible/ansible.git
   cd ./ansible
   ```

   

2. 设置 Ansible 环境：

   使用 Bash：

   ```
   source ./hacking/env-setup
   ```

   

   使用 Fish：

   ```
   source ./hacking/env-setup.fish
   ```

   

   要抑制虚假警告/错误，请使用 -q

   ```
   source ./hacking/env-setup -q
   ```

   

3. 安装 Python 依赖项：

   ```
   python3 -m pip install --user -r ./requirements.txt
   ```

   

4. 更新本地机器上的 ansible-core 的 devel 分支：

   使用带 rebase 的拉取，以便重新播放任何本地更改。

   ```
   git pull --rebase
   ```

   

## 确认您的安装

您可以通过检查版本来测试 Ansible 是否已正确安装：

```
ansible --version
```



此命令显示的版本是已安装的关联 ansible-core 软件包的版本。

要检查已安装的 ansible 软件包的版本：

```
ansible-community --version
```



## 添加 Ansible 命令行自动补全

您可以通过安装名为 argcomplete 的可选依赖项来添加 Ansible 命令行工具的 shell 自动补全功能。它支持 bash，并对 zsh 和 tcsh 的支持有限。

有关安装和配置的更多信息，请参阅 argcomplete 文档。

### 安装 argcomplete

如果您选择 pipx 安装说明：

```
pipx inject --include-apps ansible argcomplete
```



如果您选择 pip 安装说明：

```
python3 -m pip install --user argcomplete
```



### 配置 argcomplete

有两种方法可以配置 argcomplete 以允许 Ansible 命令行工具的 shell 自动补全：全局配置或按命令配置。

#### 全局配置

全局自动补全需要 bash 4.2。

```
activate-global-python-argcomplete --user
```



这会将 bash 自动补全文件写入用户位置。使用 --dest 更改位置，或使用 sudo 全局设置自动补全。

#### 按命令配置

如果您没有 bash 4.2，则必须独立注册每个脚本。

```
eval $(register-python-argcomplete ansible)
eval $(register-python-argcomplete ansible-config)
eval $(register-python-argcomplete ansible-console)
eval $(register-python-argcomplete ansible-doc)
eval $(register-python-argcomplete ansible-galaxy)
eval $(register-python-argcomplete ansible-inventory)
eval $(register-python-argcomplete ansible-playbook)
eval $(register-python-argcomplete ansible-pull)
eval $(register-python-argcomplete ansible-vault)
```



您应该将上述命令放入 shell 的配置文件中，例如 ~/.profile 或 ~/.bash_profile。
