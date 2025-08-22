---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.7 Ansible 在 Windows和BSD上使用
order: 6
---

# 在 Windows 和 BSD 上使用 Ansible

欢迎来到 Microsoft Windows 和 BSD 的 Ansible 指南。由于 Windows 不是符合 POSIX 标准的操作系统，因此 Ansible 与 Windows 主机的交互方式与 Linux/Unix 主机不同。同样，管理运行 BSD 的主机也与管理其他类 Unix 主机操作系统不同。了解在 Windows 和 BSD 主机上使用 Ansible 所需的一切。

- 设置 Windows 主机
  - 主机要求
  - WinRM 设置
  - Windows SSH 设置
- 使用 Ansible 和 Windows
  - 用例
  - Windows 的路径格式
  - 限制
  - 开发 Windows 模块
- Windows 远程管理
  - 什么是 WinRM？
  - WinRM 身份验证选项
  - 非管理员帐户
  - WinRM 加密
  - 清单选项
  - IPv6 地址
  - HTTPS 证书验证
  - TLS 1.2 支持
  - WinRM 限制
- 期望状态配置
  - 什么是期望状态配置？
  - 主机要求
  - 为什么要使用 DSC？
  - 如何使用 DSC？
  - 自定义 DSC 资源
  - 示例
- Windows 性能
  - 优化 PowerShell 性能以减少 Ansible 任务开销
  - 修复虚拟机/云实例的启动时高 CPU 占用率
- Windows 常见问题
  - Ansible 是否支持 Windows XP 或 Server 2003？
  - 我可以使用 Ansible 管理 Windows Nano Server 吗？
  - Ansible 可以运行在 Windows 上吗？
  - 我可以使用 SSH 密钥对 Windows 主机进行身份验证吗？
  - 为什么我可以在本地运行但在 Ansible 中却无法运行命令？
  - 此程序无法在 Ansible 的 Windows 上安装
  - 有哪些可用的 Windows 模块？
  - 我可以在 Windows 主机上运行 Python 模块吗？
  - 我可以通过 SSH 连接到 Windows 主机吗？
  - 为什么通过 SSH 连接到 Windows 主机失败？
  - 为什么我的凭据被拒绝？
  - 为什么我收到错误 SSL CERTIFICATE_VERIFY_FAILED？
- 使用 Ansible 管理 BSD 主机
  - 连接到 BSD 节点
  - 引导 BSD
  - 设置 Python 解释器
  - 有哪些可用的模块？
  - 使用 BSD 作为控制节点
  - BSD 事实
  - BSD 工作和贡献



## 设置 Windows 主机

本文档讨论了 Ansible 与 Microsoft Windows 主机通信之前所需的设置。

- 主机要求
  - 升级 PowerShell 和 .NET Framework
  - WinRM 内存热修复
- WinRM 设置
  - WinRM 监听器
    - 设置 WinRM 监听器
    - 删除 WinRM 监听器
  - WinRM 服务选项
  - 常见的 WinRM 问题
    - HTTP 401/凭据被拒绝
    - HTTP 500 错误
    - 超时错误
    - 连接被拒绝错误
    - 加载内置模块失败
- Windows SSH 设置
  - 使用 Windows 设置安装 OpenSSH
  - 安装 Win32-OpenSSH
  - 配置 Win32-OpenSSH shell
  - Win32-OpenSSH 身份验证
  - 配置 Ansible 以在 Windows 上使用 SSH
  - Windows 上 SSH 的已知问题

### 主机要求

为了让 Ansible 与 Windows 主机通信并使用 Windows 模块，Windows 主机必须满足以下基本连接要求

- 使用 Ansible，您通常可以管理 Microsoft 当前和扩展支持下的 Windows 版本。您还可以管理桌面操作系统，包括 Windows 10 和 11，以及服务器操作系统，包括 Windows Server 2016、2019 和 2022。
- 您需要在 Windows 主机上安装 PowerShell 5.1 或更高版本，以及至少 .NET 4.0。
- 您需要创建并激活 WinRM 监听器。更多详细信息，请参阅 WinRM 监听器。

#### 升级 PowerShell 和 .NET Framework

Ansible 需要 PowerShell 版本 5.1 和 .NET Framework 4.6 或更高版本才能正常工作。较旧的不受支持的操作系统基础映像不满足这些要求。您可以使用 Upgrade-PowerShell.ps1 脚本来更新这些。

这是如何从 PowerShell 运行此脚本的示例

```
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Upgrade-PowerShell.ps1"
$file = "$env:temp\Upgrade-PowerShell.ps1"
$username = "Administrator"
$password = "Password"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force

&$file -Version 5.1 -Username $username -Password $password -Verbose
```



在脚本中，file 值可以是 PowerShell 版本 3.0、4.0 或 5.1。

完成后，您需要运行以下 PowerShell 命令

1. 作为可选但良好的安全实践，您可以将执行策略设置回默认值。

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Force
```



对 Windows 服务器使用 RemoteSigned 值，或对 Windows 客户端使用 Restricted。

1. 删除自动登录。

```
$reg_winlogon_path = "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon"
Set-ItemProperty -Path $reg_winlogon_path -Name AutoAdminLogon -Value 0
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultUserName -ErrorAction SilentlyContinue
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultPassword -ErrorAction SilentlyContinue
```



该脚本确定您需要安装哪些程序（例如 .NET Framework 4.5.2）以及需要存在哪个 PowerShell 版本。如果需要重新启动并且设置了 username 和 password 参数，则该脚本将自动重新启动计算机然后登录。如果未设置 username 和 password 参数，则该脚本将提示用户在需要时手动重新启动并登录。当用户下次登录时，该脚本将从上次停止的地方继续，并且该过程将继续，直到不再需要任何操作为止。

#### WinRM 内存热修复

在 PowerShell v3.0 上，存在一个限制 WinRM 服务可用内存量的错误。使用 Install-WMF3Hotfix.ps1 脚本在受影响的主机上安装热修复程序，作为系统引导或映像过程的一部分。如果没有此热修复程序，Ansible 将无法在 Windows 主机上执行某些命令。

要安装热修复程序

```
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Install-WMF3Hotfix.ps1"
$file = "$env:temp\Install-WMF3Hotfix.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file -Verbose
```



有关更多详细信息，请参阅 “内存不足”错误，在已设置自定义 MaxMemoryPerShellMB 配额并安装了 WMF 3.0 的计算机上 文章。

### WinRM 设置

您需要配置 WinRM 服务，以便 Ansible 可以连接到它。WinRM 服务有两个主要组件控制 Ansible 如何与 Windows 主机交互：listener 和 service 配置设置。



#### WinRM 监听器

WinRM 服务在 一个或多个端口上侦听请求。每个端口都必须创建和配置一个监听器。

要查看在 WinRM 服务上运行的当前监听器

```
winrm enumerate winrm/config/Listener
```



这将输出类似以下内容

```
Listener
    Address = *
    Transport = HTTP
    Port = 5985
    Hostname
    Enabled = true
    URLPrefix = wsman
    CertificateThumbprint
    ListeningOn = 10.0.2.15, 127.0.0.1, 192.168.56.155, ::1, fe80::5efe:10.0.2.15%6, fe80::5efe:192.168.56.155%8, fe80::
ffff:ffff:fffe%2, fe80::203d:7d97:c2ed:ec78%3, fe80::e8ea:d765:2c69:7756%7

Listener
    Address = *
    Transport = HTTPS
    Port = 5986
    Hostname = SERVER2016
    Enabled = true
    URLPrefix = wsman
    CertificateThumbprint = E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE
    ListeningOn = 10.0.2.15, 127.0.0.1, 192.168.56.155, ::1, fe80::5efe:10.0.2.15%6, fe80::5efe:192.168.56.155%8, fe80::
ffff:ffff:fffe%2, fe80::203d:7d97:c2ed:ec78%3, fe80::e8ea:d765:2c69:7756%7
```



在上面的示例中，激活了两个监听器。一个在 HTTP 上侦听端口 5985，另一个在 HTTPS 上侦听端口 5986。一些有用的关键选项是

- Transport：监听器是通过 HTTP 还是 HTTPS 运行。我们建议您使用 HTTPS 上的监听器，因为数据在不需要任何进一步更改的情况下进行了加密。
- Port：监听器运行的端口。默认情况下，HTTP 为 5985，HTTPS 为 5986。此端口可以根据需要更改，并与主机变量 ansible_port 相对应。
- URLPrefix：要侦听的 URL 前缀。默认情况下，它是 wsman。如果更改此选项，则需要将主机变量 ansible_winrm_path 设置为相同的值。
- CertificateThumbprint：如果您使用 HTTPS 监听器，这是 Windows 证书存储中用于连接的证书的指纹。要获取证书本身的详细信息，请在 PowerShell 中使用相关的证书指纹运行此命令

```
$thumbprint = "E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE"
Get-ChildItem -Path cert:\LocalMachine\My -Recurse | Where-Object { $_.Thumbprint -eq $thumbprint } | Select-Object *
```



##### 设置 WinRM 监听器

有三种方法可以设置 WinRM 监听器

- 对 HTTP 使用 winrm quickconfig，或对 HTTPS 使用 winrm quickconfig -transport:https。当在域环境之外运行时，需要简单的监听器时，这是最容易使用的选项。与其他选项不同，此过程还具有为所需端口打开防火墙并启动 WinRM 服务的额外好处。

- 使用组策略对象 (GPO)。当主机是域的成员时，这是创建监听器的最佳方法，因为配置会自动完成，而无需任何用户输入。有关组策略对象的更多信息，请参阅 组策略对象文档。

- 使用 PowerShell 创建具有特定配置的监听器。这可以通过运行以下 PowerShell 命令来完成

  ```
  $selector_set = @{
      Address = "*"
      Transport = "HTTPS"
  }
  $value_set = @{
      CertificateThumbprint = "E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE"
  }
  
  New-WSManInstance -ResourceURI "winrm/config/Listener" -SelectorSet $selector_set -ValueSet $value_set
  ```

  

  要查看此 PowerShell 命令的其他选项，请参阅 New-WSManInstance 文档。

##### 删除 WinRM 监听器

- 要删除所有 WinRM 监听器

```
Remove-Item -Path WSMan:\localhost\Listener\* -Recurse -Force
```



- 要仅删除通过 HTTPS 运行的那些监听器

```
Get-ChildItem -Path WSMan:\localhost\Listener | Where-Object { $_.Keys -contains "Transport=HTTPS" } | Remove-Item -Recurse -Force
```



#### WinRM 服务选项

您可以控制 WinRM 服务组件的行为，包括身份验证选项和内存设置。

要获取当前服务配置选项的输出，请运行以下命令

```
winrm get winrm/config/Service
winrm get winrm/config/Winrs
```



这将输出类似以下内容

```
Service
    RootSDDL = O:NSG:BAD:P(A;;GA;;;BA)(A;;GR;;;IU)S:P(AU;FA;GA;;;WD)(AU;SA;GXGW;;;WD)
    MaxConcurrentOperations = 4294967295
    MaxConcurrentOperationsPerUser = 1500
    EnumerationTimeoutms = 240000
    MaxConnections = 300
    MaxPacketRetrievalTimeSeconds = 120
    AllowUnencrypted = false
    Auth
        Basic = true
        Kerberos = true
        Negotiate = true
        Certificate = true
        CredSSP = true
        CbtHardeningLevel = Relaxed
    DefaultPorts
        HTTP = 5985
        HTTPS = 5986
    IPv4Filter = *
    IPv6Filter = *
    EnableCompatibilityHttpListener = false
    EnableCompatibilityHttpsListener = false
    CertificateThumbprint
    AllowRemoteAccess = true

Winrs
    AllowRemoteShellAccess = true
    IdleTimeout = 7200000
    MaxConcurrentUsers = 2147483647
    MaxShellRunTime = 2147483647
    MaxProcessesPerShell = 2147483647
    MaxMemoryPerShellMB = 2147483647
    MaxShellsPerUser = 2147483647
```



您无需更改这些选项中的大多数。但是，需要了解一些重要的选项是

- Service\AllowUnencrypted - 指定 WinRM 是否允许不进行消息加密的 HTTP 流量。只有当 ansible_winrm_transport 变量为 ntlm、kerberos 或 credssp 时，消息级加密才有可能。默认情况下，此值为 false，您只应在调试 WinRM 消息时将其设置为 true。
- Service\Auth\* - 定义您可以与 WinRM 服务一起使用的身份验证选项。默认情况下，启用 Negotiate (NTLM) 和 Kerberos。
- Service\Auth\CbtHardeningLevel - 指定是否不验证通道绑定令牌（None），验证但不要求（Relaxed），或验证且要求（Strict）。只有在使用 NT LAN Manager (NTLM) 或 Kerberos 通过 HTTPS 连接时才使用 CBT。
- Service\CertificateThumbprint - 用于加密 CredSSP 身份验证的 TLS 通道的证书指纹。默认情况下，此值为空。当 WinRM 服务启动时，会生成自签名证书，并在 TLS 进程中使用。
- Winrs\MaxShellRunTime - 允许远程命令执行的最大时间（以毫秒为单位）。
- Winrs\MaxMemoryPerShellMB - 每个 shell（包括其子进程）分配的最大内存量。

要在 PowerShell 中修改 Service 键下的设置，您需要在 winrm/config/Service 之后提供选项的路径

```
Set-Item -Path WSMan:\localhost\Service\{path} -Value {some_value}
```



例如，要更改 Service\Auth\CbtHardeningLevel

```
Set-Item -Path WSMan:\localhost\Service\Auth\CbtHardeningLevel -Value Strict
```



要在 PowerShell 中修改 Winrs 键下的设置，您需要在 winrm/config/Winrs 之后提供选项的路径

```
Set-Item -Path WSMan:\localhost\Shell\{path} -Value {some_value}
```



例如，要更改 Winrs\MaxShellRunTime

```
Set-Item -Path WSMan:\localhost\Shell\MaxShellRunTime -Value 2147483647
```



#### 常见的 WinRM 问题

WinRM 有大量的配置选项，这使得其配置变得复杂。因此，Ansible 显示的错误实际上可能是主机设置的问题。

要识别主机问题，请从另一台 Windows 主机运行以下命令以连接到目标 Windows 主机。

- 要测试 HTTP

```
winrs -r:http://server:5985/wsman -u:Username -p:Password ipconfig
```



- 要测试 HTTPS

```
winrs -r:https://server:5986/wsman -u:Username -p:Password -ssl ipconfig
```



如果证书不可验证，则该命令将失败。

- 要测试 HTTPS，忽略证书验证

```
$username = "Username"
$password = ConvertTo-SecureString -String "Password" -AsPlainText -Force
$cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, $password

$session_option = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
Invoke-Command -ComputerName server -UseSSL -ScriptBlock { ipconfig } -Credential $cred -SessionOption $session_option
```



如果上述任何命令失败，则问题可能与 WinRM 设置有关。

##### HTTP 401/凭据被拒绝

HTTP 401 错误表示初始连接期间身份验证过程失败。您可以检查以下内容以进行故障排除

- 凭据正确，并在清单中使用 ansible_user 和 ansible_password 变量正确设置。
- 该用户是本地 Administrators 组的成员，或者已被明确授予访问权限。您可以使用 winrs 命令执行连接测试以排除此问题。
- 在 Service\Auth\* 下启用了 ansible_winrm_transport 变量设置的身份验证选项。
- 如果在 HTTP 而不是 HTTPS 上运行，请使用 ntlm、kerberos 或 credssp 以及 ansible_winrm_message_encryption: auto 自定义清单变量以启用消息加密。如果您使用其他身份验证选项，或者无法升级已安装的 pywinrm 包，则可以将 Service\AllowUnencrypted 设置为 true。仅建议用于故障排除。
- 下游包 pywinrm、requests-ntlm、requests-kerberos 和/或 requests-credssp 使用 pip 更新。
- 对于 Kerberos 身份验证，请确保 Service\Auth\CbtHardeningLevel 未设置为 Strict。
- 对于基本或证书身份验证，请确保该用户是本地帐户。域帐户不适用于基本和证书身份验证。

##### HTTP 500 错误

HTTP 500 错误表示 WinRM 服务存在问题。您可以检查以下内容以进行故障排除

- 您当前打开的 shell 数没有超过 WinRsMaxShellsPerUser。或者，您没有超出任何其他 Winrs 配额。

##### 超时错误

有时 Ansible 无法访问主机。这些实例通常表示网络连接存在问题。您可以检查以下内容以进行故障排除

- 防火墙未设置为阻止配置的 WinRM 监听器端口。
- 在主机变量设置的端口和路径上启用了 WinRM 监听器。
- winrm 服务正在 Windows 主机上运行，并配置为自动启动。

##### 连接被拒绝错误

当您与主机上的 WinRM 服务通信时，您可能会遇到一些问题。检查以下内容以帮助进行故障排除

- WinRM 服务在主机上启动并运行。使用 (Get-Service -Name winrm).Status 命令获取服务状态。
- 主机防火墙允许通过 WinRM 端口传输流量。默认情况下，HTTP 为 5985，HTTPS 为 5986。

有时，安装程序可能会重新启动 WinRM 或 HTTP 服务，并导致此错误。解决此问题的最佳方法是使用另一台 Windows 主机的 win_psexec 模块。

##### 加载内置模块失败

有时，PowerShell 会失败并显示类似于以下内容的错误消息

```
The 'Out-String' command was found in the module 'Microsoft.PowerShell.Utility', but the module could not be loaded.
```



在这种情况下，尝试访问 PSModulePath 环境变量指定的所有路径时可能会出现问题。

此问题的一个常见原因是 PSModulePath 包含指向文件共享的通用命名约定 (UNC) 路径。此外，双跳/凭据委派问题会导致 Ansible 进程无法访问这些文件夹。要解决此问题，请执行以下任一操作

- 从 PSModulePath 中删除 UNC 路径。

或

- 使用支持凭据委派的身份验证选项，如 credssp 或 kerberos。您需要启用凭据委派。

有关此问题的更多信息，请参阅 KB4076842。

### Windows SSH 设置

Ansible 2.8 为 Windows 管理的节点添加了实验性的 SSH 连接。

#### 使用 Windows 设置安装 OpenSSH

您可以使用 OpenSSH 将 Windows 10 客户端连接到 Windows Server 2019。OpenSSH 客户端可在 Windows 10 版本 1809 及更高版本上安装。OpenSSH 服务器可在 Windows Server 2019 及更高版本上安装。

有关更多信息，请参阅开始使用 Windows 版 OpenSSH。

#### 安装 Win32-OpenSSH

要安装用于 Ansible 的 Win32-OpenSSH 服务，请选择以下安装选项之一

- 按照 Microsoft 提供的安装说明，手动安装 Win32-OpenSSH。
- 使用 Chocolatey

```
choco install --package-parameters=/SSHServerFeature openssh
```



- 使用 win_chocolatey Ansible 模块

```
- name: install the Win32-OpenSSH service
  win_chocolatey:
    name: openssh
    package_params: /SSHServerFeature
    state: present
```



- 安装一个 Ansible Galaxy 角色，例如 jborean93.win_openssh

```
ansible-galaxy install jborean93.win_openssh
```



- 在您的 playbook 中使用该角色

```
- name: install Win32-OpenSSH service
  hosts: windows
  gather_facts: false
  roles:
  - role: jborean93.win_openssh
    opt_openssh_setup_service: True
```



#### 配置 Win32-OpenSSH shell

默认情况下，Win32-OpenSSH 使用 cmd.exe 作为 shell。

- 要配置不同的 shell，请使用 Ansible playbook，其中包含一个用于定义注册表设置的任务

```
- name: set the default shell to PowerShell
  win_regedit:
    path: HKLM:\SOFTWARE\OpenSSH
    name: DefaultShell
    data: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
    type: string
    state: present
```



- 要将设置恢复为默认 shell

```
- name: set the default shell to cmd
  win_regedit:
    path: HKLM:\SOFTWARE\OpenSSH
    name: DefaultShell
    state: absent
```



#### Win32-OpenSSH 身份验证

Windows 的 Win32-OpenSSH 身份验证类似于 Unix/Linux 主机上的 SSH 身份验证。您可以使用纯文本密码或 SSH 公钥身份验证。

对于基于密钥的身份验证

- 将您的公钥添加到用户配置文件目录的 .ssh 文件夹中的 authorized_key 文件中。
- 使用 sshd_config 文件配置 SSH 服务。

当使用 Ansible 的 SSH 密钥身份验证时，远程会话将无法访问用户凭据，并且在尝试访问网络资源时会失败。这也称为双跳或凭据委托问题。要解决此问题

- 通过设置 ansible_password 变量来使用纯文本密码身份验证。
- 在需要访问远程资源的用户凭据的任务上使用 become 指令。

#### 配置 Ansible 以在 Windows 上使用 SSH

要配置 Ansible 以将 SSH 用于 Windows 主机，您必须设置两个连接变量

- 将 ansible_connection 设置为 ssh
- 将 ansible_shell_type 设置为 cmd 或 powershell

ansible_shell_type 变量应反映 Windows 主机上配置的 DefaultShell。对于默认 shell，请将 ansible_shell_type 设置为 cmd。或者，如果您将 DefaultShell 更改为 PowerShell，则将 ansible_shell_type 设置为 powershell。

#### Windows 上使用 SSH 的已知问题

在 Windows 上使用 SSH 尚处于试验阶段。目前，存在的问题是

- 当 powershell 是 shell 类型时，旧于 v7.9.0.0p1-Beta 的 Win32-OpenSSH 版本无法工作。
- 虽然安全复制协议 (SCP) 应该可以工作，但 SSH 文件传输协议 (SFTP) 是复制或获取文件时推荐使用的机制。



## 使用 Ansible 和 Windows

当使用 Ansible 管理 Windows 时，许多适用于 Unix/Linux 主机的语法和规则也适用于 Windows，但当涉及到路径分隔符和特定于操作系统的任务等组件时，仍然存在一些差异。本文档涵盖了使用 Ansible for Windows 的具体细节。

主题

- 用例
  - 安装软件
  - 安装更新
  - 设置用户和组
    - 本地
    - 域
  - 运行命令
    - 选择 Command 或 Shell
    - 参数规则
  - 创建和运行计划任务
- Windows 的路径格式化
  - YAML 样式
  - 旧式的 key=value 样式
- 限制
- 开发 Windows 模块

### 用例

Ansible 可用于协调 Windows 服务器上的大量任务。以下是一些示例以及有关常见任务的信息。

#### 安装软件

Ansible 可以通过三种主要方式来安装软件

- 使用 win_chocolatey 模块。这从默认的公共 Chocolatey 存储库中获取程序数据。可以通过设置 source 选项来使用内部存储库。
- 使用 win_package 模块。这使用来自本地/网络路径或 URL 的 MSI 或 .exe 安装程序来安装软件。
- 使用 win_command 或 win_shell 模块手动运行安装程序。

建议使用 win_chocolatey 模块，因为它具有最完整的逻辑来检查是否已安装软件包并且是否为最新版本。

以下是一些使用所有三个选项安装 7-Zip 的示例

```
# Install/uninstall with chocolatey
- name: Ensure 7-Zip is installed through Chocolatey
  win_chocolatey:
    name: 7zip
    state: present

- name: Ensure 7-Zip is not installed through Chocolatey
  win_chocolatey:
    name: 7zip
    state: absent

# Install/uninstall with win_package
- name: Download the 7-Zip package
  win_get_url:
    url: https://www.7-zip.org/a/7z1701-x64.msi
    dest: C:\temp\7z.msi

- name: Ensure 7-Zip is installed through win_package
  win_package:
    path: C:\temp\7z.msi
    state: present

- name: Ensure 7-Zip is not installed through win_package
  win_package:
    path: C:\temp\7z.msi
    state: absent

# Install/uninstall with win_command
- name: Download the 7-Zip package
  win_get_url:
    url: https://www.7-zip.org/a/7z1701-x64.msi
    dest: C:\temp\7z.msi

- name: Check if 7-Zip is already installed
  win_reg_stat:
    name: HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{23170F69-40C1-2702-1701-000001000000}
  register: 7zip_installed

- name: Ensure 7-Zip is installed through win_command
  win_command: C:\Windows\System32\msiexec.exe /i C:\temp\7z.msi /qn /norestart
  when: 7zip_installed.exists == false

- name: Ensure 7-Zip is uninstalled through win_command
  win_command: C:\Windows\System32\msiexec.exe /x {23170F69-40C1-2702-1701-000001000000} /qn /norestart
  when: 7zip_installed.exists == true
```



某些安装程序（如 Microsoft Office 或 SQL Server）需要凭据委托或访问受 WinRM 限制的组件。绕过这些问题的最佳方法是在任务中使用 become。使用 become，Ansible 将运行安装程序，就像它在主机上以交互方式运行一样。

#### 安装更新

可以使用 win_updates 和 win_hotfix 模块在主机上安装更新或修补程序。模块 win_updates 用于按类别安装多个更新，而 win_hotfix 可用于安装已在本地下载的单个更新或修补程序文件。

以下示例显示了如何使用 win_updates

```
- name: Install all critical and security updates
  win_updates:
    category_names:
    - CriticalUpdates
    - SecurityUpdates
    state: installed
  register: update_result

- name: Reboot host if required
  win_reboot:
  when: update_result.reboot_required
```



以下示例显示了如何使用 win_hotfix 安装单个更新或修补程序

```
- name: Download KB3172729 for Server 2012 R2
  win_get_url:
    url: http://download.windowsupdate.com/d/msdownload/update/software/secu/2016/07/windows8.1-kb3172729-x64_e8003822a7ef4705cbb65623b72fd3cec73fe222.msu
    dest: C:\temp\KB3172729.msu

- name: Install hotfix
  win_hotfix:
    hotfix_kb: KB3172729
    source: C:\temp\KB3172729.msu
    state: present
  register: hotfix_result

- name: Reboot host if required
  win_reboot:
  when: hotfix_result.reboot_required
```



#### 设置用户和组

Ansible 可用于在本地和域上创建 Windows 用户和组。

##### 本地

模块 win_user、win_group 和 win_group_membership 在本地管理 Windows 用户、组和组成员资格。

以下是一个创建本地帐户和组的示例，这些帐户和组可以访问同一主机上的文件夹

```
- name: Create local group to contain new users
  win_group:
    name: LocalGroup
    description: Allow access to C:\Development folder

- name: Create local user
  win_user:
    name: '{{ item.name }}'
    password: '{{ item.password }}'
    groups: LocalGroup
    update_password: false
    password_never_expires: true
  loop:
  - name: User1
    password: Password1
  - name: User2
    password: Password2

- name: Create Development folder
  win_file:
    path: C:\Development
    state: directory

- name: Set ACL of Development folder
  win_acl:
    path: C:\Development
    rights: FullControl
    state: present
    type: allow
    user: LocalGroup

- name: Remove parent inheritance of Development folder
  win_acl_inheritance:
    path: C:\Development
    reorganize: true
    state: absent
```



##### 域

模块 win_domain_user 和 win_domain_group 管理域中的用户和组。以下是确保创建一批域用户的示例

```
- name: Ensure each account is created
  win_domain_user:
    name: '{{ item.name }}'
    upn: '{{ item.name }}@MY.DOMAIN.COM'
    password: '{{ item.password }}'
    password_never_expires: false
    groups:
    - Test User
    - Application
    company: Ansible
    update_password: on_create
  loop:
  - name: Test User
    password: Password
  - name: Admin User
    password: SuperSecretPass01
  - name: Dev User
    password: '@fvr3IbFBujSRh!3hBg%wgFucD8^x8W5'
```



#### 运行命令

在没有适合任务的模块可用时，可以使用 win_shell、win_command、raw 和 script 模块运行命令或脚本。

raw 模块只是远程执行 Powershell 命令。由于 raw 没有 Ansible 通常使用的任何包装器，因此 become、async 和环境变量不起作用。

script 模块在一个或多个 Windows 主机上从 Ansible 控制节点执行脚本。与 raw 类似，script 当前不支持 become、async 或环境变量。

win_command 模块用于执行命令，该命令可以是可执行文件或批处理文件，而 win_shell 模块用于在 shell 中执行命令。

##### 选择 Command 或 Shell

win_shell 和 win_command 模块都可以用于执行一个或多个命令。win_shell 模块在类似 shell 的进程（如 PowerShell 或 cmd）中运行，因此它可以访问 shell 运算符，例如 <、>、|、;、&& 和 ||。多行命令也可以在 win_shell 中运行。

win_command 模块只是在 shell 之外运行一个进程。它仍然可以通过将 shell 命令传递给类似 cmd.exe 或 PowerShell.exe 的 shell 可执行文件来运行 shell 命令，例如 mkdir 或 New-Item。

以下是一些使用 win_command 和 win_shell 的示例

```
- name: Run a command under PowerShell
  win_shell: Get-Service -Name service | Stop-Service

- name: Run a command under cmd
  win_shell: mkdir C:\temp
  args:
    executable: cmd.exe

- name: Run a multiple shell commands
  win_shell: |
    New-Item -Path C:\temp -ItemType Directory
    Remove-Item -Path C:\temp -Force -Recurse
    $path_info = Get-Item -Path C:\temp
    $path_info.FullName

- name: Run an executable using win_command
  win_command: whoami.exe

- name: Run a cmd command
  win_command: cmd.exe /c mkdir C:\temp

- name: Run a vbs script
  win_command: cscript.exe script.vbs
```



##### 参数规则

通过 win_command 运行命令时，适用标准的 Windows 参数规则

- 每个参数都用空格分隔，空格可以是空格或制表符。
- 参数可以用双引号 " 包围。引号内的任何内容都被解释为单个参数，即使它包含空格。
- 以反斜杠 \ 开头的双引号被解释为只是一个双引号 "，而不是参数分隔符。
- 反斜杠会被按字面意思解释，除非它紧跟在双引号之前；例如 \ == \ 和 \" == "
- 如果偶数个反斜杠后跟一个双引号，则每对反斜杠在参数中使用一个反斜杠，并且双引号用作参数的字符串分隔符。
- 如果奇数个反斜杠后跟一个双引号，则每对反斜杠在参数中使用一个反斜杠，并且双引号被转义，并在参数中成为字面双引号。

记住这些规则，以下是一些关于引号的示例

```
- win_command: C:\temp\executable.exe argument1 "argument 2" "C:\path\with space" "double \"quoted\""

argv[0] = C:\temp\executable.exe
argv[1] = argument1
argv[2] = argument 2
argv[3] = C:\path\with space
argv[4] = double "quoted"

- win_command: '"C:\Program Files\Program\program.exe" "escaped \\\" backslash" unquoted-end-backslash\'

argv[0] = C:\Program Files\Program\program.exe
argv[1] = escaped \" backslash
argv[2] = unquoted-end-backslash\

# Due to YAML and Ansible parsing '\"' must be written as '{% raw %}\\{% endraw %}"'
- win_command: C:\temp\executable.exe C:\no\space\path "arg with end \ before end quote{% raw %}\\{% endraw %}"

argv[0] = C:\temp\executable.exe
argv[1] = C:\no\space\path
argv[2] = arg with end \ before end quote\"
```



更多信息，请参阅 转义参数。

#### 创建和运行计划任务

WinRM 有一些限制，会在运行某些命令时导致错误。绕过这些限制的一种方法是通过计划任务运行命令。计划任务是一个 Windows 组件，它提供在计划时间和不同帐户下运行可执行文件的能力。

Ansible 2.5 版本添加了一些模块，使在 Windows 中处理计划任务更容易。以下示例演示了如何将脚本作为计划任务运行，并在运行后删除自身

```
- name: Create scheduled task to run a process
  win_scheduled_task:
    name: adhoc-task
    username: SYSTEM
    actions:
    - path: PowerShell.exe
      arguments: |
        Start-Sleep -Seconds 30  # This isn't required, just here as a demonstration
        New-Item -Path C:\temp\test -ItemType Directory
    # Remove this action if the task shouldn't be deleted on completion
    - path: cmd.exe
      arguments: /c schtasks.exe /Delete /TN "adhoc-task" /F
    triggers:
    - type: registration

- name: Wait for the scheduled task to complete
  win_scheduled_task_stat:
    name: adhoc-task
  register: task_stat
  until: (task_stat.state is defined and task_stat.state.status != "TASK_STATE_RUNNING") or (task_stat.task_exists == False)
  retries: 12
  delay: 10
```



### Windows 的路径格式

Windows 在许多方面与传统的 POSIX 操作系统不同。其中一个主要变化是将路径分隔符从 / 更改为 \。这可能会导致剧本编写方式出现重大问题，因为 \ 通常在 POSIX 系统中用作转义字符。

Ansible 允许两种不同的语法样式；每种样式处理 Windows 的路径分隔符的方式都不同

#### YAML 样式

当使用 YAML 语法编写任务时，规则由 YAML 标准明确定义

- 当使用普通字符串（不带引号）时，YAML 不会将反斜杠视为转义字符。
- 当使用单引号 ' 时，YAML 不会将反斜杠视为转义字符。
- 当使用双引号 " 时，反斜杠被视为转义字符，需要用另一个反斜杠进行转义。

YAML 规范考虑了以下 转义序列

- \0, \\, \", \_, \a, \b, \e, \f, \n, \r, \t, \v, \L, \N 和 \P – 单字符转义
- <TAB>, <SPACE>, <NBSP>, <LNSP>, <PSP> – 特殊字符
- \x.. – 2 位十六进制转义
- \u.... – 4 位十六进制转义
- \U........ – 8 位十六进制转义

以下是一些关于如何编写 Windows 路径的示例

```
# GOOD
tempdir: C:\Windows\Temp

# WORKS
tempdir: 'C:\Windows\Temp'
tempdir: "C:\\Windows\\Temp"

# BAD, BUT SOMETIMES WORKS
tempdir: C:\\Windows\\Temp
tempdir: 'C:\\Windows\\Temp'
tempdir: C:/Windows/Temp
```



这是一个将会失败的示例

```
# FAILS
tempdir: "C:\Windows\Temp"
```



此示例演示了在需要时使用单引号

```
---
- name: Copy tomcat config
  win_copy:
    src: log4j.xml
    dest: '{{tc_home}}\lib\log4j.xml'
```



#### 旧的 key=value 样式

旧的 key=value 语法用于命令行上的临时命令，或在剧本内部使用。不鼓励在剧本中使用这种样式，因为反斜杠字符需要被转义，这使得剧本更难阅读。旧的语法取决于 Ansible 中的特定实现，并且引号（单引号和双引号）对 Ansible 如何解析没有任何影响。

Ansible key=value 解析器 parse_kv() 考虑以下转义序列

- \, ', ", \a, \b, \f, \n, \r, \t 和 \v – 单字符转义
- \x.. – 2 位十六进制转义
- \u.... – 4 位十六进制转义
- \U........ – 8 位十六进制转义
- \N{...} – 按名称查找 Unicode 字符

这意味着反斜杠是某些序列的转义字符，因此在这种形式下通常更安全地转义反斜杠。

以下是一些关于使用 key=value 样式处理 Windows 路径的示例

```
# GOOD
tempdir=C:\\Windows\\Temp

# WORKS
tempdir='C:\\Windows\\Temp'
tempdir="C:\\Windows\\Temp"

# BAD, BUT SOMETIMES WORKS
tempdir=C:\Windows\Temp
tempdir='C:\Windows\Temp'
tempdir="C:\Windows\Temp"
tempdir=C:/Windows/Temp

# FAILS
tempdir=C:\Windows\temp
tempdir='C:\Windows\temp'
tempdir="C:\Windows\temp"
```



失败的示例不会直接失败，而是会将 \t 替换为 <TAB> 字符，导致 tempdir 变为 C:\Windows<TAB>emp。

### 限制

您无法使用 Ansible 和 Windows 执行以下操作

- 升级 PowerShell
- 与 WinRM 监听器交互

由于 WinRM 依赖于服务在正常操作期间保持在线和运行，因此您无法使用 Ansible 升级 PowerShell 或与 WinRM 监听器交互。这两种操作都会导致连接失败。从技术上讲，可以使用 async 或计划任务来避免这种情况，但如果它运行的进程破坏了 Ansible 使用的基础连接，则这些方法会很脆弱，最好将其留给引导过程或在创建映像之前进行。

### 开发 Windows 模块

因为用于 Windows 的 Ansible 模块是用 PowerShell 编写的，所以 Windows 模块的开发指南与标准模块的开发指南大不相同。请参阅 Windows 模块开发演练 获取更多信息。



## Windows 远程管理

与默认使用 SSH 的 Linux/Unix 主机不同，Windows 主机配置了 WinRM。本主题介绍如何配置和使用 Ansible 的 WinRM。

- 什么是 WinRM？
- WinRM 身份验证选项
  - 基本
  - 证书
  - NTLM
  - Kerberos
  - CredSSP
- 非管理员帐户
- WinRM 加密
- 清单选项
- IPv6 地址
- HTTPS 证书验证
- TLS 1.2 支持
- WinRM 限制

### 什么是 WinRM？

WinRM 是 Windows 使用的管理协议，用于与另一台服务器进行远程通信。它是一种基于 SOAP 的协议，通过 HTTP/HTTPS 进行通信，并包含在所有最新的 Windows 操作系统中。自 Windows Server 2012 起，WinRM 默认启用，但在大多数情况下，需要额外的配置才能将 WinRM 与 Ansible 一起使用。

Ansible 使用 pywinrm 包通过 WinRM 与 Windows 服务器通信。它不是 Ansible 包默认安装的。

如果您选择了 pipx 安装说明，则可以通过运行以下命令安装它

```
pipx inject ansible pywinrm  # if you installed ansible with pipx
pipx inject ansible-core pywinrm  # if you installed ansible-core with pipx
```



或者，如果您选择了 pip 安装说明

```
pip install "pywinrm>=0.3.0"
```





### WinRM 身份验证选项

连接到 Windows 主机时，使用帐户进行身份验证时可以使用几种不同的选项。身份验证类型可以在清单主机或组中使用 ansible_winrm_transport 变量进行设置。

以下矩阵是对选项的高级概述

| 选项     | 本地帐户 | Active Directory 帐户 | 凭据委托 | HTTP 加密 |
| -------- | -------- | --------------------- | -------- | --------- |
| 基本     | 是       | 否                    | 否       | 否        |
| 证书     | 是       | 否                    | 否       | 否        |
| Kerberos | 否       | 是                    | 是       | 是        |
| NTLM     | 是       | 是                    | 否       | 是        |
| CredSSP  | 是       | 是                    | 是       | 是        |



#### 基本

基本身份验证是最简单的身份验证选项之一，但也是最不安全的。这是因为用户名和密码只是 base64 编码的，如果未使用安全通道（例如，HTTPS），则任何人都可以对其进行解码。基本身份验证只能用于本地帐户（而不是域帐户）。

以下示例显示了为基本身份验证配置的主机变量

```
ansible_user: LocalUsername
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: basic
```



默认情况下，Windows 主机上未启用基本身份验证，但可以通过在 PowerShell 中运行以下命令来启用

```
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
```



#### 证书

证书身份验证使用证书作为密钥，类似于 SSH 密钥对，但文件格式和密钥生成过程不同。

以下示例显示了为证书身份验证配置的主机变量

```
ansible_connection: winrm
ansible_winrm_cert_pem: /path/to/certificate/public/key.pem
ansible_winrm_cert_key_pem: /path/to/certificate/private/key.pem
ansible_winrm_transport: certificate
```



默认情况下，Windows 主机上未启用证书身份验证，但可以通过在 PowerShell 中运行以下命令来启用

```
Set-Item -Path WSMan:\localhost\Service\Auth\Certificate -Value $true
```



.._winrm_certificate_generate

##### 生成证书

必须先生成证书，然后才能将其映射到本地用户。可以使用以下方法之一完成此操作

- OpenSSL
- PowerShell，使用 New-SelfSignedCertificate cmdlet
- Active Directory 证书服务

Active Directory 证书服务不在本文档的范围内，但在域环境中运行时，它可能是最佳选择。有关更多信息，请参见 Active Directory 证书服务文档。

要使用 OpenSSL 生成证书

```
# Set the name of the local user that will have the key mapped to
USERNAME="username"

cat > openssl.conf << EOL
distinguished_name = req_distinguished_name
[req_distinguished_name]
[v3_req_client]
extendedKeyUsage = clientAuth
subjectAltName = otherName:1.3.6.1.4.1.311.20.2.3;UTF8:$USERNAME@localhost
EOL

export OPENSSL_CONF=openssl.conf
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out cert.pem -outform PEM -keyout cert_key.pem -subj "/CN=$USERNAME" -extensions v3_req_client
rm openssl.conf
```



要使用 New-SelfSignedCertificate 生成证书

```
# Set the name of the local user that will have the key mapped
$username = "username"
$output_path = "C:\temp"

# Instead of generating a file, the cert will be added to the personal
# LocalComputer folder in the certificate store
$cert = New-SelfSignedCertificate -Type Custom `
    -Subject "CN=$username" `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2","2.5.29.17={text}upn=$username@localhost") `
    -KeyUsage DigitalSignature,KeyEncipherment `
    -KeyAlgorithm RSA `
    -KeyLength 2048

# Export the public key
$pem_output = @()
$pem_output += "-----BEGIN CERTIFICATE-----"
$pem_output += [System.Convert]::ToBase64String($cert.RawData) -replace ".{64}", "$&`n"
$pem_output += "-----END CERTIFICATE-----"
[System.IO.File]::WriteAllLines("$output_path\cert.pem", $pem_output)

# Export the private key in a PFX file
[System.IO.File]::WriteAllBytes("$output_path\cert.pfx", $cert.Export("Pfx"))
```



##### 将证书导入到证书存储

生成证书后，需要将颁发证书导入 LocalMachine 存储的 Trusted Root Certificate Authorities 中，并且客户端证书公钥必须存在于 LocalMachine 存储的 Trusted People 文件夹中。在此示例中，颁发证书和公钥相同。

以下示例说明如何导入颁发证书

```
$cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 "cert.pem"

$store_name = [System.Security.Cryptography.X509Certificates.StoreName]::Root
$store_location = [System.Security.Cryptography.X509Certificates.StoreLocation]::LocalMachine
$store = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Store -ArgumentList $store_name, $store_location
$store.Open("MaxAllowed")
$store.Add($cert)
$store.Close()
```



导入客户端证书公钥的代码为

```
$cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 "cert.pem"

$store_name = [System.Security.Cryptography.X509Certificates.StoreName]::TrustedPeople
$store_location = [System.Security.Cryptography.X509Certificates.StoreLocation]::LocalMachine
$store = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Store -ArgumentList $store_name, $store_location
$store.Open("MaxAllowed")
$store.Add($cert)
$store.Close()
```





##### 将证书映射到帐户

导入证书后，将其映射到本地用户帐户

```
$username = "username"
$password = ConvertTo-SecureString -String "password" -AsPlainText -Force
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, $password

# This is the issuer thumbprint which in the case of a self generated cert
# is the public key thumbprint, additional logic may be required for other
# scenarios
$thumbprint = (Get-ChildItem -Path cert:\LocalMachine\root | Where-Object { $_.Subject -eq "CN=$username" }).Thumbprint

New-Item -Path WSMan:\localhost\ClientCertificate `
    -Subject "$username@localhost" `
    -URI * `
    -Issuer $thumbprint `
    -Credential $credential `
    -Force
```



完成后，应将主机变量 ansible_winrm_cert_pem 设置为公钥的路径，并将 ansible_winrm_cert_key_pem 变量设置为私钥的路径。



#### NTLM

NTLM 是 Microsoft 使用的较旧的身份验证机制，可以同时支持本地帐户和域帐户。默认情况下，WinRM 服务上启用 NTLM，因此在使用它之前无需进行任何设置。

NTLM 是最容易使用的身份验证协议，并且比 Basic 身份验证更安全。如果在域环境中运行，应使用 Kerberos 而不是 NTLM。

与使用 NTLM 相比，Kerberos 具有以下几个优势

- NTLM 是一种较旧的协议，不支持较新的加密协议。
- NTLM 的身份验证速度较慢，因为它在身份验证阶段需要更多往返主机的行程。
- 与 Kerberos 不同，NTLM 不允许凭据委托。

此示例显示了配置为使用 NTLM 身份验证的主机变量

```
ansible_user: LocalUsername
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: ntlm
```



#### Kerberos

在域环境中运行时，建议使用 Kerberos 作为身份验证选项。Kerberos 支持诸如凭据委派和通过 HTTP 进行消息加密等功能，并且是通过 WinRM 可用的更安全的选项之一。

Kerberos 需要在 Ansible 主机上进行一些额外的设置工作才能正常使用。

以下示例显示了为 Kerberos 身份验证配置的主机变量

```
ansible_user: [email protected]
ansible_password: Password
ansible_connection: winrm
ansible_port: 5985
ansible_winrm_transport: kerberos
```



从 Ansible 2.3 版本开始，Kerberos 票证将基于 ansible_user 和 ansible_password 创建。如果在较旧版本的 Ansible 上运行或当 ansible_winrm_kinit_mode 为 manual 时，必须已获得 Kerberos 票证。请参阅下文了解更多详细信息。

可以设置一些额外的主机变量

```
ansible_winrm_kinit_mode: managed/manual (manual means Ansible will not obtain a ticket)
ansible_winrm_kinit_cmd: the kinit binary to use to obtain a Kerberos ticket (default to kinit)
ansible_winrm_service: overrides the SPN prefix that is used, the default is ``HTTP`` and should rarely ever need changing
ansible_winrm_kerberos_delegation: allows the credentials to traverse multiple hops
ansible_winrm_kerberos_hostname_override: the hostname to be used for the Kerberos exchange
```



##### 安装 Kerberos 库

在使用 Kerberos 之前，必须安装一些系统依赖项。下面的脚本列出了基于发行版的依赖项

```
# Through Yum (RHEL/Centos/Fedora for the older version)
yum -y install gcc python-devel krb5-devel krb5-libs krb5-workstation

# Through DNF (RHEL/Centos/Fedora for the newer version)
dnf -y install gcc python3-devel krb5-devel krb5-libs krb5-workstation

# Through Apt (Ubuntu older than 20.04 LTS (focal))
sudo apt-get install python-dev libkrb5-dev krb5-user

# Through Apt (Ubuntu newer than 20.04 LTS)
sudo apt-get install python3-dev libkrb5-dev krb5-user

# Through Portage (Gentoo)
emerge -av app-crypt/mit-krb5
emerge -av dev-python/setuptools

# Through Pkg (FreeBSD)
sudo pkg install security/krb5

# Through OpenCSW (Solaris)
pkgadd -d http://get.opencsw.org/now
/opt/csw/bin/pkgutil -U
/opt/csw/bin/pkgutil -y -i libkrb5_3

# Through Pacman (Arch Linux)
pacman -S krb5
```



安装依赖项后，可以安装 python-kerberos 包装器。

如果您选择了 pipx 安装说明，则可以通过运行以下命令安装它

```
pipx inject ansible pywinrm[kerberos]  # if you installed ansible with pipx
pipx inject ansible-core pywinrm[kerberos]  # if you installed ansible-core with pipx
```



或者，如果您选择了 pip 安装说明

```
pip install pywinrm[kerberos]
```



##### 配置主机 Kerberos

安装依赖项后，需要配置 Kerberos 以使其可以与域通信。此配置通过 /etc/krb5.conf 文件完成，该文件与上面脚本中的包一起安装。

要配置 Kerberos，请在以以下内容开头的节中

```
[realms]
```



添加完整的域名以及主域和辅助 Active Directory 域控制器的完全限定域名。它应该看起来像这样

```
[realms]
    MY.DOMAIN.COM = {
        kdc = domain-controller1.my.domain.com
        kdc = domain-controller2.my.domain.com
    }
```



在以以下内容开头的节中

```
[domain_realm]
```



为 Ansible 需要访问的每个域添加如下行

```
[domain_realm]
    .my.domain.com = MY.DOMAIN.COM
```



您可以在此文件中配置其他设置，例如默认域。有关更多详细信息，请参阅 krb5.conf。



##### 自动 Kerberos 票证管理

当为主机指定 ansible_user 和 ansible_password 时，Ansible 2.3 及更高版本默认为自动管理 Kerberos 票证。在此过程中，将为每个主机在临时凭据缓存中创建一个新票证。这是在每次任务执行之前完成的，以最大程度地减少票证过期的可能性。临时凭据缓存将在每个任务完成后删除，并且不会干扰默认凭据缓存。

要禁用自动票证管理，请通过清单设置 ansible_winrm_kinit_mode=manual。

自动票证管理需要在控制主机系统路径上使用标准的 kinit 二进制文件。要指定不同的位置或二进制名称，请将 ansible_winrm_kinit_cmd hostvar 设置为与 MIT krbv5 kinit 兼容的二进制文件的完全限定路径。



##### 手动 Kerberos 票证管理

要手动管理 Kerberos 票证，请使用 kinit 二进制文件。要获取新票证，请使用以下命令

```
kinit [email protected]
```



要查看已获取的票证（如果有），请使用以下命令

```
klist
```



要销毁已获取的所有票证，请使用以下命令

```
kdestroy
```



##### Kerberos 故障排除

Kerberos 依赖于正确配置的环境才能工作。要解决 Kerberos 问题，请确保

- 为 Windows 主机设置的主机名是 FQDN，而不是 IP 地址。* 如果您使用 IP 地址连接，将收到错误消息 在 Kerberos 数据库中找不到服务器。* 要确定您是否使用 IP 地址或 FQDN 连接，请使用 -vvv 标志运行您的 playbook（或调用 win_ping 模块）。

- 正向和反向 DNS 查找在域中正常工作。要测试这一点，请通过名称 ping Windows 主机，然后使用 nslookup 返回的 IP 地址。在 IP 地址上使用 nslookup 时，应返回相同的名称。

- Ansible 主机的时钟与 AD 域控制器同步。Kerberos 对时间敏感，少量的时钟漂移可能会导致票证生成过程失败。

- 确保在 krb5.conf 文件中配置了域的完全限定域名。要检查这一点，请运行

  ```
  kinit -C [email protected]
  klist
  ```

  

  如果 klist 返回的域名与请求的域名不同，则正在使用别名。krb5.conf 文件需要更新，以便使用完全限定域名而不是别名。

- 如果默认的 Kerberos 工具已被替换或修改（某些 IdM 解决方案可能会这样做），则在安装或升级 Python Kerberos 库时可能会导致问题。在撰写本文时，此库名为 pykerberos，并且已知可与 MIT 和 Heimdal Kerberos 库一起使用。要解决 pykerberos 安装问题，请确保满足 Kerberos 的系统依赖项（请参阅：安装 Kerberos 库），从 PATH 环境变量中删除任何自定义 Kerberos 工具路径，然后重试安装 Python Kerberos 库包。



#### CredSSP

CredSSP 身份验证是一种较新的身份验证协议，允许凭据委派。这是通过在身份验证成功后加密用户名和密码，然后使用 CredSSP 协议将其发送到服务器来实现的。

由于用户名和密码被发送到服务器以用于双跳身份验证，因此请确保 Windows 主机与之通信的主机未被入侵并且是受信任的。

CredSSP 可以用于本地帐户和域帐户，并且还支持通过 HTTP 进行消息加密。

要使用 CredSSP 身份验证，主机变量的配置如下

```
ansible_user: Username
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: credssp
```



一些额外的可以在下面设置的主机变量

```
ansible_winrm_credssp_disable_tlsv1_2: when true, will not use TLS 1.2 in the CredSSP auth process
```



默认情况下，Windows 主机上未启用 CredSSP 身份验证，但可以通过在 PowerShell 中运行以下命令来启用

```
Enable-WSManCredSSP -Role Server -Force
```



##### 安装 CredSSP 库

可以使用 pip 安装 requests-credssp 包装器

```
pip install pywinrm[credssp]
```



##### CredSSP 和 TLS 1.2

默认情况下，requests-credssp 库配置为通过 TLS 1.2 协议进行身份验证。对于 Windows Server 2012 和 Windows 8 及更高版本，默认情况下已安装并启用了 TLS 1.2。

有两种方法可以将较旧的主机与 CredSSP 一起使用

- 安装并启用修补程序以启用 TLS 1.2 支持（建议用于 Server 2008 R2 和 Windows 7）。
- 在清单中设置 ansible_winrm_credssp_disable_tlsv1_2=True 以通过 TLS 1.0 运行。这是连接到 Windows Server 2008 的唯一选项，Windows Server 2008 无法支持 TLS 1.2

有关如何在 Windows 主机上启用 TLS 1.2 的更多信息，请参阅 TLS 1.2 支持。



##### 设置 CredSSP 证书

CredSSP 通过 TLS 协议加密凭据，并且默认使用自签名证书。WinRM 服务配置下的 CertificateThumbprint 选项可用于指定另一个证书的指纹。

要显式设置用于 CredSSP 的证书

```
# Note the value $certificate_thumbprint will be different in each
# situation, this needs to be set based on the cert that is used.
$certificate_thumbprint = "7C8DCBD5427AFEE6560F4AF524E325915F51172C"

# Set the thumbprint value
Set-Item -Path WSMan:\localhost\Service\CertificateThumbprint -Value $certificate_thumbprint
```



### 非管理员帐户

默认情况下，WinRM 配置为仅允许来自本地 Administrators 组中帐户的连接。可以通过运行以下命令进行更改

```
winrm configSDDL default
```



这将显示一个 ACL 编辑器，您可以在其中添加新的用户或组。要通过 WinRM 运行命令，用户和组必须至少启用 Read 和 Execute 权限。

虽然非管理帐户可以与 WinRM 一起使用，但大多数典型的服务器管理任务都需要一定级别的管理访问权限，因此该实用程序的用途通常有限。



### WinRM 加密

默认情况下，当通过未加密通道运行时，WinRM 将无法工作。如果使用基于 HTTP 的 TLS (HTTPS) 或使用消息级加密，则 WinRM 协议认为该通道已加密。建议使用带有 TLS 的 WinRM，因为它适用于所有身份验证选项，但需要在 WinRM 侦听器上创建和使用证书。

如果在域环境中，ADCS 可以为由域本身颁发的主机创建证书。

如果无法使用 HTTPS，则当身份验证选项为 NTLM、Kerberos 或 CredSSP 时，可以使用 HTTP。这些协议将在 WinRM 有效负载发送到服务器之前使用它们自己的加密方法对其进行加密。当通过 HTTPS 运行而不是使用更安全的 TLS 协议时，不使用消息级加密。如果需要传输和消息加密，请在主机变量中设置 ansible_winrm_message_encryption=always。

最后的办法是禁用 Windows 主机上的加密要求。这应该仅用于开发和调试目的，因为从 Ansible 发送的任何内容都可以被查看或操纵，并且同一网络上的任何人都可以完全接管远程会话。要禁用加密要求

```
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true
```



### 清单选项

Ansible 的 Windows 支持依赖于一些标准变量来指示远程主机的主机名、密码和连接类型。这些变量最容易在清单中设置，但可以在 host_vars/ group_vars 级别设置。

设置清单时，需要以下变量

```
# It is suggested that these be encrypted with ansible-vault:
# ansible-vault edit group_vars/windows.yml
ansible_connection: winrm

# May also be passed on the command-line through --user
ansible_user: Administrator

# May also be supplied at runtime with --ask-pass
ansible_password: SecretPasswordGoesHere
```



使用上述变量，Ansible 将通过 HTTPS 使用基本身份验证连接到 Windows 主机。如果 ansible_user 具有像 username@MY.DOMAIN.COM 这样的 UPN 值，则除非 ansible_winrm_transport 已设置为除 kerberos 之外的其他值，否则身份验证选项将自动尝试使用 Kerberos。

还支持以下自定义清单变量，用于 WinRM 连接的附加配置

- ansible_port：WinRM 将在其上运行的端口，HTTPS 为 5986，这是默认值，而 HTTP 为 5985
- ansible_winrm_scheme：指定用于 WinRM 连接的连接方案（http 或 https）。除非 ansible_port 为 5985，否则 Ansible 默认使用 https
- ansible_winrm_path：指定 WinRM 端点的备用路径，Ansible 默认使用 /wsman
- ansible_winrm_realm：指定用于 Kerberos 身份验证的领域。如果 ansible_user 包含 @，Ansible 将默认使用 @ 后的用户名部分
- ansible_winrm_transport：以逗号分隔的列表形式指定一个或多个身份验证传输选项。默认情况下，如果安装了 kerberos 模块并且定义了领域，Ansible 将使用 kerberos, basic，否则它将是 plaintext
- ansible_winrm_server_cert_validation：指定服务器证书验证模式（ignore 或 validate）。Ansible 默认为 Python 2.7.9 及更高版本上的 validate，这将导致针对 Windows 自签名证书的证书验证错误。除非已在 WinRM 侦听器上配置了可验证的证书，否则应将其设置为 ignore
- ansible_winrm_operation_timeout_sec：增加 WinRM 操作的默认超时，Ansible 默认使用 20
- ansible_winrm_read_timeout_sec：增加 WinRM 读取超时，Ansible 默认使用 30。如果存在间歇性网络问题且不断发生读取超时错误，则此设置非常有用
- ansible_winrm_message_encryption：指定要使用的消息加密操作（auto、always、never），Ansible 默认使用 auto。auto 表示仅当 ansible_winrm_scheme 为 http 且 ansible_winrm_transport 支持消息加密时才使用消息加密。always 表示将始终使用消息加密，而 never 表示将永远不会使用消息加密
- ansible_winrm_ca_trust_path：用于指定与 certifi 模块中使用的不同的 cacert 容器。有关更多详细信息，请参阅 HTTPS 证书验证部分。
- ansible_winrm_send_cbt：当通过 HTTPS 使用 ntlm 或 kerberos 时，身份验证库将尝试发送通道绑定令牌以缓解中间人攻击。此标志控制是否发送这些绑定（默认值：true）。
- ansible_winrm_*：可以提供 winrm.Protocol 支持的任何其他关键字参数来代替 *

此外，对于每个身份验证选项，还需要设置特定的变量。有关更多信息，请参阅上面的身份验证部分。



### IPv6 地址

可以使用 IPv6 地址代替 IPv4 地址或主机名。此选项通常在清单中设置。Ansible 将尝试使用 ipaddress 包解析地址，并正确传递给 pywinrm。

使用 IPv6 地址定义主机时，只需像添加 IPv4 地址或主机名一样添加 IPv6 地址即可

```
[windows-server]
2001:db8::1

[windows-server:vars]
ansible_user=username
ansible_password=password
ansible_connection=winrm
```



### HTTPS 证书验证

作为 TLS 协议的一部分，将验证证书以确保主机与主题匹配，并且客户端信任服务器证书的颁发者。当使用自签名证书或设置 ansible_winrm_server_cert_validation: ignore 时，这些安全机制将被绕过。虽然自签名证书总是需要 ignore 标志，但是从证书颁发机构颁发的证书仍然可以验证。

在域环境中设置 HTTPS 侦听器的一种更常见的方法是使用 Active Directory 证书服务 (AD CS)。AD CS 用于从证书签名请求 (CSR) 生成签名证书。如果 WinRM HTTPS 侦听器正在使用由其他机构（如 AD CS）签名的证书，则可以将 Ansible 设置为信任该颁发者作为 TLS 握手的一部分。

为了让 Ansible 信任像 AD CS 这样的证书颁发机构 (CA)，可以将 CA 的颁发者证书导出为 PEM 编码的证书。然后，可以将此证书复制到 Ansible 控制节点本地，并用作证书验证的来源，也称为 CA 链。

CA 链可以包含一个或多个颁发者证书，每个条目都位于新的一行中。要将自定义 CA 链用作验证过程的一部分，请将 ansible_winrm_ca_trust_path 设置为该文件的路径。如果未设置此变量，则将使用默认 CA 链，该链位于 Python 包 certifi 的安装路径中。



### TLS 1.2 支持

由于 WinRM 通过 HTTP 协议运行，使用 HTTPS 意味着使用 TLS 协议来加密 WinRM 消息。TLS 将自动尝试协商客户端和服务器都可用的最佳协议和密码套件。如果找不到匹配项，则 Ansible 将报错，并显示类似于以下内容的消息：

```
HTTPSConnectionPool(host='server', port=5986): Max retries exceeded with url: /wsman (Caused by SSLError(SSLError(1, '[SSL: UNSUPPORTED_PROTOCOL] unsupported protocol (_ssl.c:1056)')))
```



通常，这是因为 Windows 主机尚未配置为支持 TLS v1.2，但也可能意味着 Ansible 控制节点安装了较旧的 OpenSSL 版本。

Windows 8 和 Windows Server 2012 默认安装并启用了 TLS v1.2，但较旧的主机（如 Server 2008 R2 和 Windows 7）必须手动启用。

要验证 Windows 主机支持的协议，可以在 Ansible 控制节点上运行以下命令：

```
openssl s_client -connect <hostname>:5986
```



输出将包含有关 TLS 会话的信息，并且 Protocol 行将显示协商的版本。

```
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1
    Cipher    : ECDHE-RSA-AES256-SHA
    Session-ID: 962A00001C95D2A601BE1CCFA7831B85A7EEE897AECDBF3D9ECD4A3BE4F6AC9B
    Session-ID-ctx:
    Master-Key: ....
    Start Time: 1552976474
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
---

New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: AE16000050DA9FD44D03BB8839B64449805D9E43DBD670346D3D9E05D1AEEA84
    Session-ID-ctx:
    Master-Key: ....
    Start Time: 1552976538
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
```



如果主机返回 TLSv1，则应将其配置为启用 TLS v1.2。您可以通过运行以下 PowerShell 脚本来实现此目的：

```
Function Enable-TLS12 {
    param(
        [ValidateSet("Server", "Client")]
        [String]$Component = "Server"
    )

    $protocols_path = 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols'
    New-Item -Path "$protocols_path\TLS 1.2\$Component" -Force
    New-ItemProperty -Path "$protocols_path\TLS 1.2\$Component" -Name Enabled -Value 1 -Type DWORD -Force
    New-ItemProperty -Path "$protocols_path\TLS 1.2\$Component" -Name DisabledByDefault -Value 0 -Type DWORD -Force
}

Enable-TLS12 -Component Server

# Not required but highly recommended to enable the Client side TLS 1.2 components
Enable-TLS12 -Component Client

Restart-Computer
```



以下 Ansible 任务也可用于启用 TLS v1.2：

```
- name: enable TLSv1.2 support
  win_regedit:
    path: HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\{{ item.type }}
    name: '{{ item.property }}'
    data: '{{ item.value }}'
    type: dword
    state: present
  register: enable_tls12
  loop:
  - type: Server
    property: Enabled
    value: 1
  - type: Server
    property: DisabledByDefault
    value: 0
  - type: Client
    property: Enabled
    value: 1
  - type: Client
    property: DisabledByDefault
    value: 0

- name: reboot if TLS config was applied
  win_reboot:
  when: enable_tls12 is changed
```



还有其他方法可以配置 TLS 协议以及 Windows 主机提供的密码套件。Nartac Software 提供的 IIS Crypto 工具可以为您提供一个 GUI 来管理这些设置。



### WinRM 限制

由于 WinRM 协议的设计，在使用 WinRM 时存在一些限制，这可能会在为 Ansible 创建 playbook 时导致问题。这些限制包括：

- 大多数身份验证类型的凭据都不会被委派，这会导致访问网络资源或安装某些程序时出现身份验证错误。
- 通过 WinRM 运行时，对 Windows Update API 的许多调用会被阻止。
- 由于没有凭据委派，或者因为它们通过 WinRM 访问禁止的 Windows API（如 WUA），某些程序无法使用 WinRM 安装。
- WinRM 下的命令是在非交互式会话下完成的，这可能会阻止某些命令或可执行文件运行。
- 您不能运行与 DPAPI 交互的进程，该进程被某些安装程序（如 Microsoft SQL Server）使用。

可以通过执行以下操作之一来缓解其中一些限制：

- 将 ansible_winrm_transport 设置为 credssp 或 kerberos（使用 ansible_winrm_kerberos_delegation=true）以绕过双跳问题并访问网络资源
- 使用 become 绕过所有 WinRM 限制，并像在本地一样运行命令。与使用 credssp 等身份验证传输不同，这还将消除非交互式限制和 API 限制（如 WUA 和 DPAPI）
- 使用计划任务运行可以使用 win_scheduled_task 模块创建的命令。与 become 类似，这会绕过所有 WinRM 限制，但只能运行命令而不能运行模块。



## 所需状态配置

主题

- 什么是所需状态配置？
- 主机要求
- 为什么要使用 DSC？
- 如何使用 DSC？
  - 属性类型
    - PSCredential
    - CimInstance 类型
    - HashTable 类型
    - 数组
    - DateTime
  - 以其他用户身份运行
- 自定义 DSC 资源
  - 查找自定义 DSC 资源
  - 安装自定义资源
- 示例
  - 提取 zip 文件
  - 创建目录
  - 与 Azure 交互
  - 设置 IIS 网站

### 什么是所需状态配置？

所需状态配置（或 DSC）是一个内置于 PowerShell 中的工具，可用于通过代码定义 Windows 主机设置。DSC 的总体目的与 Ansible 相同，只是执行方式不同。自 Ansible 2.4 起，添加了 win_dsc 模块，可在与 Windows 主机交互时利用现有的 DSC 资源。

有关 DSC 的更多详细信息，请查看 DSC 概述。

### 主机要求

要使用 win_dsc 模块，Windows 主机必须安装 PowerShell v5.0 或更高版本。所有支持的主机都可以升级到 PowerShell v5。

满足 PowerShell 要求后，使用 DSC 就像使用 win_dsc 模块创建任务一样简单。

### 为什么要使用 DSC？

DSC 和 Ansible 模块具有一个共同的目标，即定义和确保资源的状态。因此，可以使用 DSC 文件资源和 Ansible win_file 等资源来实现相同的结果。决定使用哪个取决于具体场景。

使用 Ansible 模块而不是 DSC 资源的原因

- 主机不支持 PowerShell v5.0，或者不容易升级
- DSC 资源不提供 Ansible 模块中存在的功能。例如，win_regedit 可以管理 REG_NONE 属性类型，而 DSC Registry 资源则不能
- DSC 资源对检查模式的支持有限，而某些 Ansible 模块具有更好的检查
- DSC 资源不支持 diff 模式，而某些 Ansible 模块支持
- 自定义资源需要在主机上运行进一步的安装步骤，而 Ansible 模块是内置于 Ansible 中的
- DSC 资源中存在错误，而 Ansible 模块可以正常工作

使用 DSC 资源而不是 Ansible 模块的原因

- Ansible 模块不支持 DSC 资源中存在的功能
- 没有可用的 Ansible 模块
- 现有 Ansible 模块中存在错误

最后，使用 DSC 还是 Ansible 模块执行任务并不重要；重要的是任务是否正确执行以及 playbook 是否仍然可读。如果您对 DSC 的经验比对 Ansible 多，并且它能够完成工作，只需为该任务使用 DSC 即可。

### 如何使用 DSC？

win_dsc 模块接受自由格式的选项，以便根据其管理的资源进行更改。可以在 resources 中找到内置资源的列表。

以 Registry 资源为例，这是 Microsoft 文档中记录的 DSC 定义

```
Registry [string] #ResourceName
{
    Key = [string]
    ValueName = [string]
    [ Ensure = [string] { Enable | Disable }  ]
    [ Force =  [bool]   ]
    [ Hex = [bool] ]
    [ DependsOn = [string[]] ]
    [ ValueData = [string[]] ]
    [ ValueType = [string] { Binary | Dword | ExpandString | MultiString | Qword | String }  ]
}
```



定义任务时，必须将 resource_name 设置为正在使用的 DSC 资源 - 在这种情况下，应将 resource_name 设置为 Registry。module_version 可以引用已安装的 DSC 资源的特定版本；如果留空，则默认为最新版本。其他选项是用于定义资源的参数，例如 Key 和 ValueName。虽然任务中的选项不区分大小写，但建议保持原样大小写，因为它使区分 DSC 资源选项和 Ansible 的 win_dsc 选项更加容易。

这是上述 DSC Registry 资源的 Ansible 任务版本

```
- name: Use win_dsc module with the Registry DSC resource
  win_dsc:
    resource_name: Registry
    Ensure: Present
    Key: HKEY_LOCAL_MACHINE\SOFTWARE\ExampleKey
    ValueName: TestValue
    ValueData: TestData
```



从 Ansible 2.8 开始，win_dsc 模块会自动使用 DSC 定义验证来自 Ansible 的输入选项。这意味着如果选项名称不正确、未设置强制选项或值不是有效选择，Ansible 将会失败。当以 3 或更高的详细级别运行 Ansible（-vvv）时，返回值将包含基于指定的 resource_name 的可能调用选项。以下是上述 Registry 任务的调用输出示例

```
changed: [2016] => {
    "changed": true,
    "invocation": {
        "module_args": {
            "DependsOn": null,
            "Ensure": "Present",
            "Force": null,
            "Hex": null,
            "Key": "HKEY_LOCAL_MACHINE\\SOFTWARE\\ExampleKey",
            "PsDscRunAsCredential_password": null,
            "PsDscRunAsCredential_username": null,
            "ValueData": [
                "TestData"
            ],
            "ValueName": "TestValue",
            "ValueType": null,
            "module_version": "latest",
            "resource_name": "Registry"
        }
    },
    "module_version": "1.1",
    "reboot_required": false,
    "verbose_set": [
        "Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = ResourceSet,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.",
        "An LCM method call arrived from computer SERVER2016 with user sid S-1-5-21-3088887838-4058132883-1884671576-1105.",
        "[SERVER2016]: LCM:  [ Start  Set      ]  [[Registry]DirectResourceAccess]",
        "[SERVER2016]:                            [[Registry]DirectResourceAccess] (SET) Create registry key 'HKLM:\\SOFTWARE\\ExampleKey'",
        "[SERVER2016]:                            [[Registry]DirectResourceAccess] (SET) Set registry key value 'HKLM:\\SOFTWARE\\ExampleKey\\TestValue' to 'TestData' of type 'String'",
        "[SERVER2016]: LCM:  [ End    Set      ]  [[Registry]DirectResourceAccess]  in 0.1930 seconds.",
        "[SERVER2016]: LCM:  [ End    Set      ]    in  0.2720 seconds.",
        "Operation 'Invoke CimMethod' complete.",
        "Time taken for configuration job to complete is 0.402 seconds"
    ],
    "verbose_test": [
        "Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = ResourceTest,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.",
        "An LCM method call arrived from computer SERVER2016 with user sid S-1-5-21-3088887838-4058132883-1884671576-1105.",
        "[SERVER2016]: LCM:  [ Start  Test     ]  [[Registry]DirectResourceAccess]",
        "[SERVER2016]:                            [[Registry]DirectResourceAccess] Registry key 'HKLM:\\SOFTWARE\\ExampleKey' does not exist",
        "[SERVER2016]: LCM:  [ End    Test     ]  [[Registry]DirectResourceAccess] False in 0.2510 seconds.",
        "[SERVER2016]: LCM:  [ End    Set      ]    in  0.3310 seconds.",
        "Operation 'Invoke CimMethod' complete.",
        "Time taken for configuration job to complete is 0.475 seconds"
    ]
}
```



invocation.module_args 键显示了实际设置的值以及其他未设置的可能值。遗憾的是，这不会显示 DSC 属性的默认值，只会显示 Ansible 任务中设置的值。任何 *_password 选项都会出于安全原因在输出中被屏蔽；如果还有其他敏感模块选项，请在任务上设置 no_log: True，以停止记录所有任务输出。

#### 属性类型

每个 DSC 资源属性都具有与之关联的类型。Ansible 将尝试在执行期间将定义的选项转换为正确的类型。对于像 [string] 和 [bool] 这样的简单类型，这是一个简单的操作，但是像 [PSCredential] 或数组（如 [string[]]）这样的复杂类型则需要一定的规则。

##### PSCredential

[PSCredential] 对象用于以安全的方式存储凭据，但 Ansible 无法通过 JSON 序列化此对象。要设置 DSC PSCredential 属性，该参数的定义应有两个条目，分别以 _username 和 _password 为后缀，分别表示用户名和密码。例如

```
PsDscRunAsCredential_username: '{{ ansible_user }}'
PsDscRunAsCredential_password: '{{ ansible_password }}'

SourceCredential_username: AdminUser
SourceCredential_password: PasswordForAdminUser
```



[PSCredential] 在 DSC 资源 MOF 定义中使用 EmbeddedInstance("MSFT_Credential") 定义。

##### CimInstance 类型

DSC 使用 [CimInstance] 对象来存储基于该资源定义的自定义类的字典对象。在 YAML 中定义一个接受 [CimInstance] 的值与在 YAML 中定义字典相同。例如，要在 Ansible 中定义 [CimInstance] 值

```
# [CimInstance]AuthenticationInfo == MSFT_xWebAuthenticationInformation
AuthenticationInfo:
  Anonymous: false
  Basic: true
  Digest: false
  Windows: true
```



在上面的示例中，CIM 实例是类 MSFT_xWebAuthenticationInformation 的表示。此类接受四个布尔变量：Anonymous、Basic、Digest 和 Windows。在 [CimInstance] 中使用的键取决于它所代表的类。请仔细阅读资源文档以确定可以使用的键以及每个键值的类型。类定义通常位于 <resource name>.schema.mof 中。

##### HashTable 类型

[HashTable] 对象也是一个字典，但没有严格的键集可以/需要定义。与 [CimInstance] 类似，在 YAML 中将其定义为普通的字典值。[HashTable]] 在 DSC 资源 MOF 定义中使用 EmbeddedInstance("MSFT_KeyValuePair") 定义。

##### 数组

像 [string[]] 或 [UInt32[]] 这样的简单类型数组被定义为一个列表或一个逗号分隔的字符串，然后将其强制转换为其类型。建议使用列表，因为在传递给 DSC 引擎之前，win_dsc 模块不会手动解析这些值。例如，要在 Ansible 中定义一个简单类型的数组

```
# [string[]]
ValueData: entry1, entry2, entry3
ValueData:
- entry1
- entry2
- entry3

# [UInt32[]]
ReturnCode: 0,3010
ReturnCode:
- 0
- 3010
```



像 [CimInstance[]]（字典数组）这样的复杂类型数组可以像以下示例一样定义

```
# [CimInstance[]]BindingInfo == MSFT_xWebBindingInformation
BindingInfo:
- Protocol: https
  Port: 443
  CertificateStoreName: My
  CertificateThumbprint: C676A89018C4D5902353545343634F35E6B3A659
  HostName: DSCTest
  IPAddress: '*'
  SSLFlags: 1
- Protocol: http
  Port: 80
  IPAddress: '*'
```



上面的示例是一个包含两个 MSFT_xWebBindingInformation 类值的数组。定义 [CimInstance[]] 时，请务必阅读资源文档以了解在定义中要使用的键。

##### DateTime

[DateTime] 对象是一个 DateTime 字符串，表示 ISO 8601 日期时间格式的日期和时间。[DateTime] 字段的值应在 YAML 中用引号引起来，以确保该字符串正确地序列化到 Windows 主机。以下是如何在 Ansible 中定义 [DateTime] 值的示例

```
# As UTC-0 (No timezone)
DateTime: '2019-02-22T13:57:31.2311892+00:00'

# As UTC+4
DateTime: '2019-02-22T17:57:31.2311892+04:00'

# As UTC-4
DateTime: '2019-02-22T09:57:31.2311892-04:00'
```



上面的所有值都等于 UTC 日期时间 2019 年 2 月 22 日下午 1:57，其中包含 31 秒和 2311892 毫秒。

#### 以其他用户身份运行

默认情况下，DSC 将每个资源作为 SYSTEM 帐户而不是 Ansible 用于运行模块的帐户运行。这意味着根据用户配置文件动态加载的资源（例如 HKEY_CURRENT_USER 注册表项）将在 SYSTEM 配置文件下加载。参数 PsDscRunAsCredential 是可以为每个 DSC 资源设置的参数，并强制 DSC 引擎在不同的帐户下运行。由于 PsDscRunAsCredential 的类型为 PSCredential，因此使用 _username 和 _password 后缀定义。

以 Registry 资源类型为例，以下是如何定义一个任务来访问 Ansible 用户的 HKEY_CURRENT_USER 项

```
- name: Use win_dsc with PsDscRunAsCredential to run as a different user
  win_dsc:
    resource_name: Registry
    Ensure: Present
    Key: HKEY_CURRENT_USER\ExampleKey
    ValueName: TestValue
    ValueData: TestData
    PsDscRunAsCredential_username: '{{ ansible_user }}'
    PsDscRunAsCredential_password: '{{ ansible_password }}'
  no_log: true
```



### 自定义 DSC 资源

DSC 资源不仅限于 Microsoft 的内置选项。可以安装自定义模块来管理通常不可用的其他资源。

#### 查找自定义 DSC 资源

您可以使用 PSGallery 来查找自定义资源，以及有关如何在 Windows 主机上安装它们的文档。

还可以使用 Find-DscResource cmdlet 来查找自定义资源。例如

```
# Find all DSC resources in the configured repositories
Find-DscResource

# Find all DSC resources that relate to SQL
Find-DscResource -ModuleName "*sql*"
```



#### 安装自定义资源

有三种方法可以在主机上安装 DSC 资源

- 使用 Install-Module cmdlet 手动安装
- 使用 win_psmodule Ansible 模块
- 手动保存模块并将其复制到另一台主机

以下是使用 win_psmodule 安装 xWebAdministration 资源的示例

```
- name: Install xWebAdministration DSC resource
  win_psmodule:
    name: xWebAdministration
    state: present
```



安装后，win_dsc 模块将能够通过使用 resource_name 选项引用资源来使用该资源。

只有当主机可以访问互联网时，上述前两种方法才有效。当主机无法访问互联网时，必须首先使用上述方法在另一台可以访问互联网的主机上安装模块，然后再复制过来。要将模块保存到本地文件路径，可以运行以下 PowerShell cmdlet

```
Save-Module -Name xWebAdministration -Path C:\temp
```



这将在 C:\temp 中创建一个名为 xWebAdministration 的文件夹，该文件夹可以复制到任何主机。为了使 PowerShell 能够看到此脱机资源，必须将其复制到 PSModulePath 环境变量中设置的目录中。在大多数情况下，C:\Program Files\WindowsPowerShell\Module 路径通过此变量设置，但可以使用 win_path 模块添加不同的路径。

### 示例

#### 提取 zip 文件

```
- name: Extract a zip file
  win_dsc:
    resource_name: Archive
    Destination: C:\temp\output
    Path: C:\temp\zip.zip
    Ensure: Present
```



### 创建目录

```
- name: Create file with some text
  win_dsc:
    resource_name: File
    DestinationPath: C:\temp\file
    Contents: |
        Hello
        World
    Ensure: Present
    Type: File

- name: Create directory that is hidden is set with the System attribute
  win_dsc:
    resource_name: File
    DestinationPath: C:\temp\hidden-directory
    Attributes: Hidden,System
    Ensure: Present
    Type: Directory
```



#### 与 Azure 交互

```
- name: Install xAzure DSC resources
  win_psmodule:
    name: xAzure
    state: present

- name: Create virtual machine in Azure
  win_dsc:
    resource_name: xAzureVM
    ImageName: a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201409.01-en.us-127GB.vhd
    Name: DSCHOST01
    ServiceName: ServiceName
    StorageAccountName: StorageAccountName
    InstanceSize: Medium
    Windows: true
    Ensure: Present
    Credential_username: '{{ ansible_user }}'
    Credential_password: '{{ ansible_password }}'
```



#### 设置 IIS 网站

```
- name: Install xWebAdministration module
  win_psmodule:
    name: xWebAdministration
    state: present

- name: Install IIS features that are required
  win_dsc:
    resource_name: WindowsFeature
    Name: '{{ item }}'
    Ensure: Present
  loop:
  - Web-Server
  - Web-Asp-Net45

- name: Setup web content
  win_dsc:
    resource_name: File
    DestinationPath: C:\inetpub\IISSite\index.html
    Type: File
    Contents: |
      <html>
      <head><title>IIS Site</title></head>
      <body>This is the body</body>
      </html>
    Ensure: present

- name: Create new website
  win_dsc:
    resource_name: xWebsite
    Name: NewIISSite
    State: Started
    PhysicalPath: C:\inetpub\IISSite\index.html
    BindingInfo:
    - Protocol: https
      Port: 8443
      CertificateStoreName: My
      CertificateThumbprint: C676A89018C4D5902353545343634F35E6B3A659
      HostName: DSCTest
      IPAddress: '*'
      SSLFlags: 1
    - Protocol: http
      Port: 8080
      IPAddress: '*'
    AuthenticationInfo:
      Anonymous: false
      Basic: true
      Digest: false
      Windows: true
```



## Windows 性能

本文档提供了一些您可以应用于 Windows 主机的性能优化，以加快它们的速度，特别是在将 Ansible 与它们一起使用时以及通常情况下。

### 优化 PowerShell 性能以减少 Ansible 任务开销

要将 PowerShell 的启动速度提高大约 10 倍，请在管理员会话中运行以下 PowerShell 代码段。预计需要花费数十秒。

```
function Optimize-Assemblies {
    param (
        [string]$assemblyFilter = "Microsoft.PowerShell.",
        [string]$activity = "Native Image Installation"
    )

    try {
        # Get the path to the ngen executable dynamically
        $ngenPath = [System.IO.Path]::Combine([Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory(), "ngen.exe")

        # Check if ngen.exe exists
        if (-Not (Test-Path $ngenPath)) {
            Write-Host "Ngen.exe not found at $ngenPath. Make sure .NET Framework is installed."
            return
        }

        # Get a list of loaded assemblies
        $assemblies = [AppDomain]::CurrentDomain.GetAssemblies()

        # Filter assemblies based on the provided filter
        $filteredAssemblies = $assemblies | Where-Object { $_.FullName -ilike "$assemblyFilter*" }

        if ($filteredAssemblies.Count -eq 0) {
            Write-Host "No matching assemblies found for optimization."
            return
        }

        foreach ($assembly in $filteredAssemblies) {
            # Get the name of the assembly
            $name = [System.IO.Path]::GetFileName($assembly.Location)

            # Display progress
            Write-Progress -Activity $activity -Status "Optimizing $name"

            # Use Ngen to install the assembly
            Start-Process -FilePath $ngenPath -ArgumentList "install `"$($assembly.Location)`"" -Wait -WindowStyle Hidden
        }

        Write-Host "Optimization complete."
    } catch {
        Write-Host "An error occurred: $_"
    }
}

# Optimize PowerShell assemblies:
Optimize-Assemblies -assemblyFilter "Microsoft.PowerShell."
```



每个 Windows Ansible 模块都使用 PowerShell。此优化减少了 PowerShell 启动所需的时间，从而消除了每次调用中的开销。

此代码段使用 本机映像生成器 ngen 为 PowerShell 依赖的程序集抢先创建本机映像。

### 修复虚拟机/云实例启动时 CPU 占用率过高的问题

如果您正在创建用于生成实例的黄金映像，则可以通过在黄金映像创建过程中处理 ngen 队列来避免启动时出现破坏性的高 CPU 任务，前提是您知道 CPU 类型在黄金映像构建过程和运行时之间不会更改。

请将以下内容放在您的 playbook 末尾附近，同时考虑到可能导致本机映像失效的因素（[请参阅 MSDN](https://docs.microsoft.com/en-us/dotnet/framework/tools/ngen-exe-native-image-generator#native-images-and-jit-compilation)）。

```
- name: generate native .NET images for CPU
  win_dotnet_ngen:
```



## Windows 常见问题解答

以下是一些关于 Ansible 和 Windows 的常见问题及其答案。

### Ansible 是否适用于 Windows XP 或 Server 2003？

Ansible 不适用于 Windows XP 或 Server 2003 主机。Ansible 适用于以下 Windows 操作系统版本：

- Windows Server 2016
- Windows Server 2019
- Windows Server 2022
- Windows 10
- Windows 11

对 Windows Server 2008、2008 R2 和 Windows 7 的支持在 2.10 版本中结束。对 Windows Server 2012、2012 R2、Windows 8 和 8.1 的支持在 2.16 版本中结束。

Ansible 还有最低 PowerShell 版本要求 - 请参阅设置 Windows 主机以获取最新信息。

### 我可以使用 Ansible 管理 Windows Nano Server 吗？

Ansible 目前不适用于 Windows Nano Server，因为它无法访问大多数模块和内部组件使用的完整 .NET Framework。



### Ansible 可以在 Windows 上运行吗？

否，Ansible 只能管理 Windows 主机。Ansible 不能在 Windows 主机上本机运行，但可以在 Linux 的 Windows 子系统 (WSL) 下运行。

要在 WSL 上安装 Ansible，可以在 bash 终端中运行以下命令：

```
sudo apt-get update
sudo apt-get install python3-pip git libffi-dev libssl-dev -y
pip install --user ansible pywinrm
```



要从源代码而不是在 WSL 上发布 Ansible，只需卸载通过 pip 安装的版本，然后克隆 git 存储库。

```
pip uninstall ansible -y
git clone https://github.com/ansible/ansible.git
source ansible/hacking/env-setup

# To enable Ansible on login, run the following
echo ". ~/ansible/hacking/env-setup -q' >> ~/.bashrc
```



如果在 WSL 上运行 Ansible 时遇到超时错误，可能是由于 sleep 未正确返回导致的。以下解决方法可能会解决此问题：

```
mv /usr/bin/sleep /usr/bin/sleep.orig
ln -s /bin/true /usr/bin/sleep
```



如果运行 Windows 10 版本高于 2004，则另一种选择是使用 WSL 2。

```
wsl --set-default-version 2
```



### 我可以使用 SSH 密钥来验证 Windows 主机的身份吗？

您不能将 SSH 密钥与 WinRM 或 PSRP 连接插件一起使用。这些连接插件使用 X509 证书进行身份验证，而不是 SSH 使用的 SSH 密钥对。

X509 证书的生成方式以及其与用户的映射方式与 SSH 实现不同；有关详细信息，请参阅Windows 远程管理文档。

Ansible 2.8 添加了一个实验性选项，可以使用 SSH 连接插件（该插件使用 SSH 密钥进行身份验证）用于 Windows 服务器。有关详细信息，请参阅此问题。



### 为什么我可以在本地运行但在 Ansible 下无法运行的命令？

Ansible 通过 WinRM 执行命令。这些进程与在本地运行命令的方式不同，具体如下：

- 除非使用 CredSSP 或具有凭据委派的 Kerberos 等身份验证选项，否则 WinRM 进程无法将用户的凭据委派给网络资源，从而导致 Access is Denied 错误。
- 在 WinRM 下运行的所有进程都处于非交互式会话中。需要交互式会话的应用程序将无法工作。
- 通过 WinRM 运行时，Windows 限制对内部 Windows API 的访问，如 Windows Update API 和 DPAPI，某些安装程序和程序依赖于这些 API。

绕过这些限制的一些方法是：

- 使用 become，它会像在本地运行时一样运行命令。这将绕过大多数 WinRM 限制，因为当使用 become 时，Windows 不知道该进程是在 WinRM 下运行的。有关详细信息，请参阅了解特权提升：become文档。
- 使用计划任务，可以使用 win_scheduled_task 创建。像 become 一样，它将绕过所有 WinRM 限制，但只能用于运行命令，而不能用于运行模块。
- 使用 win_psexec 在主机上运行命令。PSExec 不使用 WinRM，因此会绕过任何限制。
- 要在不使用任何这些解决方法的情况下访问网络资源，可以使用 CredSSP 或启用了凭据委派的 Kerberos。

有关如何使用 become 的更多信息，请参阅了解特权提升：become。Windows 远程管理的限制部分有关于 WinRM 限制的更多详细信息。

### 这个程序无法在 Windows 上使用 Ansible 安装

有关 WinRM 限制的更多信息，请参阅此问题。

### 有哪些可用的 Windows 模块？

Ansible Core 中的大多数 Ansible 模块都是为 Linux/Unix 机器和任意 Web 服务的组合编写的。这些模块是用 Python 编写的，其中大多数在 Windows 上不起作用。

因此，有专门的 Windows 模块是用 PowerShell 编写的，旨在在 Windows 主机上运行。可以在此处找到这些模块的列表。

此外，以下 Ansible Core 模块/操作插件适用于 Windows：

- add_host
- assert
- async_status
- debug
- fail
- fetch
- group_by
- include
- include_role
- include_vars
- meta
- pause
- raw
- script
- set_fact
- set_stats
- setup
- slurp
- template (也: win_template)
- wait_for_connection

Ansible Windows 模块存在于Ansible.Windows、Community.Windows和Chocolatey.Chocolatey集合中。

### 我可以在 Windows 主机上运行 Python 模块吗？

否，WinRM 连接协议设置为使用 PowerShell 模块，因此 Python 模块将不起作用。绕过此问题的一种方法是使用 delegate_to: localhost 在 Ansible 控制节点上运行 Python 模块。如果在 playbook 期间需要联系外部服务，并且没有可用的等效 Windows 模块，则此方法非常有用。



### 我可以通过 SSH 连接到 Windows 主机吗？

Ansible 2.8 添加了一个实验性选项，可以使用 SSH 连接插件来管理 Windows 主机。要通过 SSH 连接到 Windows 主机，您必须在 Windows 主机上安装和配置 Microsoft 正在开发的Win32-OpenSSH 分支。虽然大多数基础知识应该适用于 SSH，但 Win32-OpenSSH 正在快速变化，每个版本都会添加新功能和修复错误。强烈建议您从 GitHub 发布页面安装最新版本的 Win32-OpenSSH，以便在 Windows 主机上使用 Ansible 时使用。

要使用 SSH 连接到 Windows 主机，请在清单中设置以下变量：

```
ansible_connection=ssh

# Set either cmd or powershell not both
ansible_shell_type=cmd
# ansible_shell_type=powershell
```



`ansible_shell_type` 的值应为 `cmd` 或 `powershell`。如果 SSH 服务上未配置 `DefaultShell`，请使用 `cmd`；如果已将其设置为 `DefaultShell`，则使用 `powershell`。

### 为什么通过 SSH 连接到 Windows 主机失败？

除非您使用如上所述的 Win32-OpenSSH，否则必须使用 Windows 远程管理连接到 Windows 主机。如果您的 Ansible 输出指示使用了 SSH，则要么您没有正确设置连接变量，要么主机没有正确继承这些变量。

请确保在 Windows 主机的清单中设置了 ansible_connection: winrm。

### 为什么我的凭据被拒绝？

这可能是由于与凭据不正确无关的多种原因造成的。

有关这可能意味着什么的更详细指南，请参阅 设置 Windows 主机中的 HTTP 401/凭据被拒绝。

### 为什么我会收到错误 SSL CERTIFICATE_VERIFY_FAILED？

当 Ansible 控制节点运行在 Python 2.7.9+ 或具有回移植 SSLContext 的旧版本 Python（如 RHEL 7 上的 Python 2.7.5）上时，控制节点将尝试验证 WinRM 用于 HTTPS 连接的证书。如果无法验证证书（例如自签名证书的情况），则验证过程将失败。

要忽略证书验证，请在 Windows 主机的清单中添加 `ansible_winrm_server_cert_validation: ignore`。



## 使用 Ansible 管理 BSD 主机

管理 BSD 计算机与管理其他类 Unix 计算机不同。如果您有运行 BSD 的托管节点，请查看以下主题。

- 连接到 BSD 节点
- 引导 BSD
- 设置 Python 解释器
  - FreeBSD 软件包和 Ports
  - INTERPRETER_PYTHON_FALLBACK
  - 调试 Python 的发现
  - 其他变量
- 哪些模块可用？
- 将 BSD 用作控制节点
- BSD facts
- BSD 的努力和贡献

### 连接到 BSD 节点

Ansible 默认使用 OpenSSH 连接到托管节点。如果您使用 SSH 密钥进行身份验证，这在 BSD 上有效。但是，如果您使用 SSH 密码进行身份验证，则 Ansible 依赖于 sshpass。大多数版本的 sshpass 不能很好地处理 BSD 登录提示，因此在使用 SSH 密码连接 BSD 计算机时，请使用 paramiko 而不是 OpenSSH 进行连接。您可以在 ansible.cfg 中全局执行此操作，也可以将其设置为清单/组/主机变量。例如

```
[freebsd]
mybsdhost1 ansible_connection=paramiko
```





### 引导 BSD

Ansible 默认是无代理的，但是，它需要在托管节点上安装 Python。只有 raw 模块才能在没有 Python 的情况下运行。尽管此模块可用于引导 Ansible 并在 BSD 变体上安装 Python（请参见下文），但它非常有限，并且需要使用 Python 才能充分利用 Ansible 的功能。

以下示例安装了 Python，其中包括 Ansible 全部功能所需的 json 库。在您的控制计算机上，您可以对大多数 FreeBSD 版本执行以下操作

```
ansible -m raw -a "pkg install -y python" mybsdhost1
```



或者对于 OpenBSD

```
ansible -m raw -a "pkg_add python%3.8"
```



完成此操作后，您现在可以使用 raw 模块之外的其他 Ansible 模块。

### 设置 Python 解释器

为了支持各种类 Unix 操作系统和发行版，Ansible 不能总是依赖于现有环境或 env 变量来定位正确的 Python 二进制文件。默认情况下，模块指向 /usr/bin/python，因为这是最常见的位置。在 BSD 变体上，此路径可能有所不同，因此建议通知 Ansible 二进制文件的位置。请参阅 INTERPRETER_PYTHON。例如，设置 ansible_python_interpreter 清单变量

```
[freebsd:vars]
ansible_python_interpreter=/usr/local/bin/python
[openbsd:vars]
ansible_python_interpreter=/usr/local/bin/python3.8
```



#### FreeBSD 软件包和 Ports

在 FreeBSD 中，不能保证默认情况下会安装 /usr/local/bin/python 可执行文件或指向可执行文件的链接。对于远程主机，关于 Ansible 的最佳实践是至少安装 Ansible 支持的 Python 版本，例如， lang/python38，以及元端口 lang/python3 和 lang/python。引用自 /usr/ports/lang/python3/pkg-descr

```
This is a meta port to the Python 3.x interpreter and provides symbolic links
to bin/python3, bin/pydoc3, bin/idle3 and so on to allow compatibility with
minor version agnostic Python scripts.
```



引用自 /usr/ports/lang/python/pkg-descr

```
This is a meta port to the Python interpreter and provides symbolic links
to bin/python, bin/pydoc, bin/idle and so on to allow compatibility with
version agnostic python scripts.
```



因此，安装以下软件包

```
shell> pkg info | grep python
python-3.8_3,2                 "meta-port" for the default version of Python interpreter
python3-3_3                    Meta-port for the Python interpreter 3.x
python38-3.8.12_1              Interpreted object-oriented programming language
```



和以下可执行文件和链接

```
shell> ll /usr/local/bin/ | grep python
lrwxr-xr-x  1 root  wheel       7 Jan 24 08:30 python@ -> python3
lrwxr-xr-x  1 root  wheel      14 Jan 24 08:30 python-config@ -> python3-config
lrwxr-xr-x  1 root  wheel       9 Jan 24 08:29 python3@ -> python3.8
lrwxr-xr-x  1 root  wheel      16 Jan 24 08:29 python3-config@ -> python3.8-config
-r-xr-xr-x  1 root  wheel    5248 Jan 13 01:12 python3.8*
-r-xr-xr-x  1 root  wheel    3153 Jan 13 01:12 python3.8-config*
```



#### INTERPRETER_PYTHON_FALLBACK

自 2.8 版本起，Ansible 提供了一个有用的变量 ansible_interpreter_python_fallback 来指定搜索 Python 的路径列表。请参阅 INTERPRETER_PYTHON_FALLBACK。将搜索此列表，并将使用找到的第一项。例如，以下配置将使上一节中元端口的安装变得多余，也就是说，如果您不安装 Python 元端口，则将跳过列表中的前两项，并且将发现 /usr/local/bin/python3.8。

```
ansible_interpreter_python_fallback=['/usr/local/bin/python', '/usr/local/bin/python3', '/usr/local/bin/python3.8']
```



您可以延长此变量，使用较低版本的 Python，并将其放置在例如 group_vars/all 中。然后，根据需要在 group_vars/{group1, group2, ...} 中覆盖特定组的变量，并在 host_vars/{host1, host2, ...} 中覆盖特定主机的变量。请参阅 变量优先级：我应该将变量放在哪里？。

#### 调试 Python 的发现

例如，给定清单

```
shell> cat hosts
[test]
test_11
test_12
test_13

[test:vars]
ansible_connection=ssh
ansible_user=admin
ansible_become=true
ansible_become_user=root
ansible_become_method=sudo
ansible_interpreter_python_fallback=['/usr/local/bin/python', '/usr/local/bin/python3', '/usr/local/bin/python3.8']
ansible_perl_interpreter=/usr/local/bin/perl
```



以下 Playbook

```
shell> cat playbook.yml
- hosts: test_11
  gather_facts: false
  tasks:
    - command: which python
      register: result
    - debug:
        var: result.stdout
    - debug:
        msg: |-
          {% for i in _vars %}
          {{ i }}:
            {{ lookup('vars', i)|to_nice_yaml|indent(2) }}
          {% endfor %}
      vars:
        _vars: "{{ query('varnames', '.*python.*') }}"
```



显示详细信息

```
shell> ansible-playbook -i hosts playbook.yml

PLAY [test_11] *******************************************************************************

TASK [command] *******************************************************************************
[WARNING]: Platform freebsd on host test_11 is using the discovered Python interpreter at
/usr/local/bin/python, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.org.cn/ansible-
core/2.12/reference_appendices/interpreter_discovery.html for more information.
changed: [test_11]

TASK [debug] *********************************************************************************
ok: [test_11] =>
  result.stdout: /usr/local/bin/python

TASK [debug] *********************************************************************************
ok: [test_11] =>
  msg: |-
    ansible_interpreter_python_fallback:
      - /usr/local/bin/python
      - /usr/local/bin/python3
      - /usr/local/bin/python3.8

    discovered_interpreter_python:
      /usr/local/bin/python

    ansible_playbook_python:
      /usr/bin/python3
```



您可以看到，从列表 ansible_interpreter_python_fallback 中的第一项是在 FreeBSD 远程主机上发现的。变量 ansible_playbook_python 保留了在运行 Playbook 的 Linux 控制节点上的 Python 路径。

关于警告，引用自 INTERPRETER_PYTHON

```
The fallback behavior will issue a warning that the interpreter
should be set explicitly (since interpreters installed later may
change which one is used). This warning behavior can be disabled by
setting auto_silent or auto_legacy_silent. ...
```



您可以忽略它，也可以通过设置变量 ansible_python_interpreter=auto_silent 来消除它，因为这实际上是您通过使用 /usr/local/bin/python（“稍后安装的解释器可能会更改使用的解释器”）想要实现的目标。例如

```
shell> cat hosts
[test]
test_11
test_12
test_13

[test:vars]
ansible_connection=ssh
ansible_user=admin
ansible_become=true
ansible_become_user=root
ansible_become_method=sudo
ansible_interpreter_python_fallback=['/usr/local/bin/python', '/usr/local/bin/python3', '/usr/local/bin/python3.8']
ansible_python_interpreter=auto_silent
ansible_perl_interpreter=/usr/local/bin/perl
```



#### 其他变量

如果您使用 Ansible 捆绑的插件之外的其他插件，则可以根据插件的编写方式为 bash、perl 或 ruby 设置类似的变量。例如

```
[freebsd:vars]
ansible_python_interpreter=/usr/local/bin/python
ansible_perl_interpreter=/usr/local/bin/perl
```



### 哪些模块可用？

大多数核心 Ansible 模块是为类 Unix 计算机和其他通用服务的组合编写的，因此大多数模块应该在 BSD 上运行良好，但明显例外的是那些针对仅限 Linux 技术的模块（例如 LVG）。

### 将 BSD 用作控制节点

将 BSD 用作控制计算机与为您的 BSD 变体安装 Ansible 软件包或按照 pip 或“从源代码”的说明进行安装一样简单。



### BSD facts

Ansible 从 BSD 系统收集信息的方式与 Linux 机器类似，但由于网络、磁盘和其他设备的数据、名称和结构可能有所不同，因此输出结果可能会略有不同，但对于 BSD 管理员来说仍然是熟悉的。



### BSD 贡献与努力

在 Ansible 中，对 BSD 的支持对我们来说非常重要。尽管我们的大部分贡献者使用并面向 Linux，但我们有一个活跃的 BSD 社区，并努力尽可能地对 BSD 友好。请随时报告您在使用 BSD 时发现的任何问题或不兼容之处；我们也欢迎包含修复的拉取请求！
