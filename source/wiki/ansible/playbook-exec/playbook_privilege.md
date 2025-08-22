---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.3 权限提升
order: 2
---

# 理解权限提升：become

Ansible 使用现有的权限提升系统以 root 权限或其他用户的权限执行任务。由于此功能允许您“成为”另一个用户（与登录到机器的远程用户不同的用户），因此我们称之为become。become关键字使用现有的权限提升工具，例如sudo、su、pfexec、doas、pbrun、dzdo、ksu、runas、machinectl等。

- 使用 become
  - Become 指令
  - Become 连接变量
  - Become 命令行选项
- become 的风险和限制
  - 成为非特权用户的风险
  - 并非所有连接插件都支持
  - 每个主机只能启用一种方法
  - 权限提升必须是通用的
  - 可能无法访问由 pamd_systemd 填充的环境变量
  - 解决临时文件错误消息
- become 和网络自动化
  - 为所有任务设置启用模式
    - 启用模式的密码
  - authorize 和 auth_pass
- become 和 Windows
  - 管理员权限
  - 本地服务帐户
  - 无需设置密码即可使用 become
  - 没有密码的帐户
  - Windows 的 Become 标志
  - become 在 Windows 上的限制

## 使用 become

您可以使用 playbook 或任务指令、连接变量或命令行来控制become的使用。如果您以多种方式设置权限提升属性，请查看常规优先级规则以了解将使用哪些设置。

Ansible 中包含的所有 become 插件的完整列表可以在插件列表中找到。

### Become 指令

您可以在 play 或任务级别设置控制become的指令。您可以通过设置连接变量来覆盖这些指令，这些变量通常因主机而异。这些变量和指令是独立的。例如，设置become_user不会设置become。

- become

  设置为true以激活权限提升。

- become_user

  设置为具有所需权限的用户——您become的用户，而不是您登录的用户。不暗示become: true，以允许在主机级别设置它。默认值为root。

- become_method

  （在 play 或任务级别）覆盖ansible.cfg中设置的默认方法，设置为使用任何Become 插件。

- become_flags

  （在 play 或任务级别）允许为任务或角色使用特定标志。一个常见的用途是在 shell 设置为 nologin 时将用户更改为 nobody。在 Ansible 2.2 中添加。

例如，要管理系统服务（需要root权限）时，以非root用户连接，您可以使用become_user的默认值（root）

```
- name: Ensure the httpd service is running
  service:
    name: httpd
    state: started
  become: true
```



以apache用户身份运行命令

```
- name: Run a command as the apache user
  command: somecommand
  become: true
  become_user: apache
```



当 shell 为 nologin 时，以nobody用户身份执行某些操作

```
- name: Run a command as nobody
  command: somecommand
  become: true
  become_method: su
  become_user: nobody
  become_flags: '-s /bin/sh'
```



要为 sudo 指定密码，请使用--ask-become-pass（简写为-K）运行ansible-playbook。如果您运行使用become的 playbook 并且 playbook 似乎挂起，很可能它卡在了权限提升提示符处。使用CTRL-c停止它，然后使用-K和相应的密码执行 playbook。

### Become 连接变量

您可以为每个受管节点或组定义不同的become选项。您可以在清单中定义这些变量，或将它们用作普通变量。

- ansible_become

  覆盖become指令并决定是否使用权限提升。

- ansible_become_method

  应使用哪种权限提升方法

- ansible_become_user

  设置您通过权限提升成为的用户；不暗示ansible_become: true

- ansible_become_password

  设置权限提升密码。有关如何避免在纯文本中使用密钥的详细信息，请参见使用加密变量和文件

- ansible_common_remote_group

  确定如果setfacl和chown都失败，Ansible 是否应尝试将其临时文件chgrp到一个组。有关更多信息，请参见成为非特权用户的风险。在 2.10 版中添加。

例如，如果您想在名为webserver的服务器上以root身份运行所有任务，但您只能以manager用户身份连接，您可以使用如下所示的清单条目

```
webserver ansible_user=manager ansible_become=true
```



### Become 命令行选项

- --ask-become-pass, -K

  询问权限提升密码；不暗示将使用 become。请注意，此密码将用于所有主机。

- --become, -b

  使用 become 运行操作（不包含密码）。

- --become-method=BECOME_METHOD

  要使用的权限提升方法（默认为 sudo），有效选择：[sudo | su | pbrun | pfexec | doas | dzdo | ksu | runas | machinectl]

- --become-user=BECOME_USER

  以该用户身份运行操作（默认为 root），不暗示--become/-b

## become 的风险和限制

尽管权限提升大多很直观，但其工作方式有一些限制。用户应注意这些限制，以免出现意外。

### 成为非特权用户的风险

Ansible 模块在远程机器上执行，方法是首先将参数替换到模块文件中，然后将文件复制到远程机器，最后在那里执行它。

如果模块文件执行时未使用become，或者become_user为root用户，或者以root用户连接到远程机器，则一切正常。在这些情况下，Ansible 会创建权限仅允许用户和root读取，或仅允许切换到的非特权用户读取的模块文件。

但是，当连接用户和become_user都是非特权用户时，模块文件将以Ansible连接的用户（remote_user）身份写入，但文件需要可被Ansible设置为become的用户读取。Ansible 如何解决这个问题的细节可能因平台而异。但在POSIX系统上，Ansible 通过以下方式解决此问题：

首先，如果setfacl已安装并在远程PATH中可用，并且远程主机的临时目录已挂载了POSIX.1e文件系统ACL支持，则Ansible将使用POSIX ACL与第二个非特权用户共享模块文件。

接下来，如果POSIX ACL不可用或无法运行setfacl，Ansible将尝试使用chown更改模块文件的所属者，前提是系统支持非特权用户这样做。

Ansible 2.11新增功能：此时，Ansible将尝试使用chmod +a，这是一种在macOS上设置文件ACL的特定方法。

Ansible 2.10新增功能：如果上述所有方法都失败，Ansible将检查配置设置ansible_common_remote_group的值。许多系统允许给定用户将文件的组所有权更改为用户所属的组。因此，如果第二个非特权用户（become_user）与Ansible连接的用户（remote_user）拥有共同的UNIX组，并且ansible_common_remote_group被定义为该组，则Ansible可以使用chgrp尝试将模块文件的组所有权更改为该组，从而使其可能可被become_user读取。

此时，如果已定义ansible_common_remote_group并尝试了chgrp并成功返回，Ansible假设（但重要的是，不检查）新的组所有权就足够了，并且不会进一步回退。也就是说，Ansible不检查become_user是否确实与remote_user共享一个组；只要命令成功退出，Ansible就认为结果成功，并且不会继续检查下面的world_readable_temp。

如果ansible_common_remote_group未设置且上面的chown失败，或者ansible_common_remote_group已设置但chgrp（或后续的组权限chmod）返回非成功的退出代码，Ansible将最后检查world_readable_temp选项。如果设置了此选项，Ansible将把模块文件放在一个世界可读的临时目录中，并具有世界可读的权限，以允许become_user（顺便说一句，系统上的任何其他用户）读取文件的内容。如果传递给模块的任何参数本质上是敏感的，并且您不信任远程机器，那么这可能是一个潜在的安全风险。

模块执行完毕后，Ansible会删除临时文件。

存在几种方法可以完全避免上述逻辑流程：

- 使用管道传输。启用管道传输时，Ansible不会将模块保存到客户端的临时文件。相反，它会将模块传递到远程Python解释器的stdin。管道传输不适用于涉及文件传输的Python模块（例如：copy，fetch，template），也不适用于非Python模块。
- 避免成为非特权用户。当您become root或不使用become时，临时文件受UNIX文件权限保护。在Ansible 2.1及以上版本中，如果您以root用户连接到被管理的机器，然后使用become访问非特权帐户，则UNIX文件权限也是安全的。

2.1版本中的更改。

Ansible使得难以无意中不安全地使用become。从Ansible 2.1开始，如果Ansible无法安全地使用become执行，则默认情况下会发出错误。如果您无法使用管道传输或POSIX ACL，必须以非特权用户连接，必须使用become以不同的非特权用户身份执行，并且确定您的被管理节点足够安全，以允许您想要在那里运行的模块是世界可读的，您可以打开world_readable_temp选项，这会将此错误更改为警告并允许任务像2.1之前的版本那样运行。

2.10版本中的更改。

Ansible 2.10引入了上述ansible_common_remote_group回退机制。如上所述，如果启用，则在remote_user和become_user都是非特权用户时使用。有关此回退何时发生的信息，请参考上面的文本。

### 并非所有连接插件都支持

特权提升方法也必须受所使用的连接插件支持。大多数连接插件如果不支持become，都会发出警告。有些插件会忽略它，因为它们总是以root用户身份运行（jail、chroot等）。

### 每个主机只能启用一种方法

方法不能串联。您不能使用sudo /bin/su -成为用户，您需要具有以该用户身份运行命令的特权，或者能够直接su到它（pbrun、pfexec或其他支持的方法也是如此）。

### 特权提升必须是通用的

您不能将特权提升权限限制为某些命令。Ansible并不总是使用特定命令来执行某些操作，而是从每次都会更改的临时文件名运行模块（代码）。如果您将‘/sbin/service’或‘/bin/chmod’作为允许的命令，这将与Ansible一起失败，因为这些路径与Ansible创建以运行模块的临时文件不匹配。如果您有安全规则将您的sudo/pbrun/doas环境限制为仅运行特定的命令路径，请从没有此限制的特殊帐户使用Ansible，或使用AWX或Red Hat Ansible Automation Platform来管理对SSH凭据的间接访问。

### 可能无法访问由pam_systemd填充的环境变量

对于大多数使用systemd作为其init的Linux发行版，become使用的默认方法不会打开新的“会话”（systemd意义上的）。因为pam_systemd模块不会完全初始化新的会话，所以与通过ssh打开的普通会话相比，您可能会遇到意外情况：pam_systemd设置的一些环境变量，最值得注意的是XDG_RUNTIME_DIR，不会为新用户填充，而是继承或只是清空。

这可能会在尝试调用依赖XDG_RUNTIME_DIR访问总线的systemd命令时造成麻烦。

```
echo $XDG_RUNTIME_DIR

systemctl --user status
Failed to connect to bus: Permission denied
```



为了强制become打开一个通过pam_systemd的新systemd会话，可以使用become_method: machinectl。

更多信息，请参见这个systemd问题。

### 解决临时文件错误消息

- 无法设置Ansible在成为非特权用户时需要创建的临时文件的权限”
- 此错误可以通过安装提供setfacl命令的软件包来解决。（这通常是acl软件包，但请检查您的操作系统文档。）



## Become和网络自动化

从2.6版本开始，Ansible支持在所有支持enable模式的Ansible维护的网络平台上使用become进行权限提升（进入enable模式或特权EXEC模式）。使用become替换了provider字典中的authorize和auth_pass选项。

必须将连接类型设置为connection: ansible.netcommon.network_cli或connection: ansible.netcommon.httpapi才能在网络设备上使用become进行权限提升。有关详细信息，请查看平台选项文档。

您可以在只需要提升权限的特定任务、整个剧本或所有剧本上使用提升的权限。添加become: true和become_method: enable将指示Ansible在执行设置这些参数的任务、剧本或剧本集之前进入enable模式。

如果您看到此错误消息，则生成它的任务需要enable模式才能成功。

```
Invalid input (privileged mode required)
```



要为特定任务设置enable模式，请在任务级别添加become。

```
- name: Gather facts (eos)
  arista.eos.eos_facts:
    gather_subset:
      - "!hardware"
  become: true
  become_method: enable
```



要为单个剧本中的所有任务设置enable模式，请在剧本级别添加become。

```
- hosts: eos-switches
  become: true
  become_method: enable
  tasks:
    - name: Gather facts (eos)
      arista.eos.eos_facts:
        gather_subset:
          - "!hardware"
```



### 为所有任务设置enable模式

通常，您希望所有剧本中的所有任务都使用特权模式运行，这最好通过使用group_vars来实现。

group_vars/eos.yml

```
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: arista.eos.eos
ansible_user: myuser
ansible_become: true
ansible_become_method: enable
```



#### enable模式的密码

如果您需要密码才能进入enable模式，您可以通过两种方式之一指定它。

- 提供--ask-become-pass命令行选项。
- 设置ansible_become_password连接变量。

### authorize和auth_pass

Ansible仍然支持使用connection: local的旧版网络剧本的enable模式。要使用connection: local进入enable模式，请使用模块选项authorize和auth_pass。

```
- hosts: eos-switches
  ansible_connection: local
  tasks:
    - name: Gather facts (eos)
      eos_facts:
        gather_subset:
          - "!hardware"
      provider:
        authorize: true
        auth_pass: " {{ secret_auth_pass }}"
```



我们建议您将您的剧本更新为始终使用become用于网络设备的enable模式。将来将弃用使用authorize和provider字典。有关详细信息，请查看平台选项文档。



## Become和Windows

从Ansible 2.3开始，可以通过runas方法在Windows主机上使用become。Windows上的Become使用与非Windows主机上的become相同的清单设置和调用参数，因此设置和变量名称与本文档中定义的相同，但become_user除外。由于Windows上become_user没有合理的默认值，因此在使用become时是必需的。有关详细信息，请参见ansible.builtin.runas become插件。

虽然become可用于假定另一个用户的身份，但它在Windows主机上还有其他用途。一个重要的用途是绕过在WinRM上运行时施加的一些限制，例如受约束的网络委派或访问禁止的系统调用（如WUA API）。您可以将become与与ansible_user相同的用户一起使用，以绕过这些限制并运行在WinRM会话中通常无法访问的命令。

### 管理权限

Windows中的许多任务都需要管理权限才能完成。使用runas become方法时，Ansible将尝试使用可用于become用户的全部权限运行模块。如果它未能提升用户令牌，它将在执行期间继续使用受限令牌。

用户必须拥有SeDebugPrivilege才能以提升的权限运行become进程。此权限默认分配给管理员。如果调试权限不可用，则become进程将以有限的权限和组运行。

要确定Ansible能够获得的令牌类型，请运行以下任务。

```
- name: Check my username
  ansible.windows.win_whoami:
  become: true
```



输出将类似于以下内容。

```
ok: [windows] => {
    "account": {
        "account_name": "vagrant-domain",
        "domain_name": "DOMAIN",
        "sid": "S-1-5-21-3088887838-4058132883-1884671576-1105",
        "type": "User"
    },
    "authentication_package": "Kerberos",
    "changed": false,
    "dns_domain_name": "DOMAIN.LOCAL",
    "groups": [
        {
            "account_name": "Administrators",
            "attributes": [
                "Mandatory",
                "Enabled by default",
                "Enabled",
                "Owner"
            ],
            "domain_name": "BUILTIN",
            "sid": "S-1-5-32-544",
            "type": "Alias"
        },
        {
            "account_name": "INTERACTIVE",
            "attributes": [
                "Mandatory",
                "Enabled by default",
                "Enabled"
            ],
            "domain_name": "NT AUTHORITY",
            "sid": "S-1-5-4",
            "type": "WellKnownGroup"
        },
    ],
    "impersonation_level": "SecurityAnonymous",
    "label": {
        "account_name": "High Mandatory Level",
        "domain_name": "Mandatory Label",
        "sid": "S-1-16-12288",
        "type": "Label"
    },
    "login_domain": "DOMAIN",
    "login_time": "2018-11-18T20:35:01.9696884+00:00",
    "logon_id": 114196830,
    "logon_server": "DC01",
    "logon_type": "Interactive",
    "privileges": {
        "SeBackupPrivilege": "disabled",
        "SeChangeNotifyPrivilege": "enabled-by-default",
        "SeCreateGlobalPrivilege": "enabled-by-default",
        "SeCreatePagefilePrivilege": "disabled",
        "SeCreateSymbolicLinkPrivilege": "disabled",
        "SeDebugPrivilege": "enabled",
        "SeDelegateSessionUserImpersonatePrivilege": "disabled",
        "SeImpersonatePrivilege": "enabled-by-default",
        "SeIncreaseBasePriorityPrivilege": "disabled",
        "SeIncreaseQuotaPrivilege": "disabled",
        "SeIncreaseWorkingSetPrivilege": "disabled",
        "SeLoadDriverPrivilege": "disabled",
        "SeManageVolumePrivilege": "disabled",
        "SeProfileSingleProcessPrivilege": "disabled",
        "SeRemoteShutdownPrivilege": "disabled",
        "SeRestorePrivilege": "disabled",
        "SeSecurityPrivilege": "disabled",
        "SeShutdownPrivilege": "disabled",
        "SeSystemEnvironmentPrivilege": "disabled",
        "SeSystemProfilePrivilege": "disabled",
        "SeSystemtimePrivilege": "disabled",
        "SeTakeOwnershipPrivilege": "disabled",
        "SeTimeZonePrivilege": "disabled",
        "SeUndockPrivilege": "disabled"
    },
    "rights": [
        "SeNetworkLogonRight",
        "SeBatchLogonRight",
        "SeInteractiveLogonRight",
        "SeRemoteInteractiveLogonRight"
    ],
    "token_type": "TokenPrimary",
    "upn": "[email protected]",
    "user_flags": []
}
```



在label键下，account_name条目确定用户是否具有管理权限。以下是可返回的标签及其含义。

- Medium：Ansible未能获取提升的令牌，并在受限令牌下运行。在模块执行期间，只有分配给用户的权限子集可用，并且用户没有管理权限。
- High：使用了提升的令牌，并且在模块执行期间可以使用分配给用户的所有权限。
- System：使用NT AUTHORITY\System帐户，并具有最高的可用权限级别。

输出还将显示已授予用户的权限列表。当权限值为disabled时，权限已分配给登录令牌，但尚未启用。在大多数情况下，这些权限在需要时会自动启用。

如果在低于2.5版本的Ansible上运行，或者正常的runas升级过程失败，则可以通过以下方式检索提升的令牌：

- 将become_user设置为System，它对操作系统具有完全控制权。

- 将SeTcbPrivilege授予Ansible在WinRM上连接的用户。SeTcbPrivilege是一个高级权限，它授予对操作系统的完全控制权。默认情况下，没有用户获得此权限，如果您将此权限授予用户或组，则应谨慎操作。有关此权限的更多信息，请参见充当操作系统的一部分。您可以使用以下任务在Windows主机上设置此权限。

  ```
  - name: grant the ansible user the SeTcbPrivilege right
    ansible.windows.win_user_right:
      name: SeTcbPrivilege
      users: '{{ansible_user}}'
      action: add
  ```

  

- 在尝试成为用户之前，关闭主机的UAC并重新启动。UAC是一种旨在使用最小权限原则运行帐户的安全协议。您可以通过运行以下任务来关闭UAC。

  ```
  - name: turn UAC off
    win_regedit:
      path: HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system
      name: EnableLUA
      data: 0
      type: dword
      state: present
    register: uac_result
  
  - name: reboot after disabling UAC
    win_reboot:
    when: uac_result is changed
  ```

  

### 本地服务帐户

在Ansible 2.5版本之前，become仅适用于具有本地或域用户帐户的Windows。在这些旧版本中，不能将诸如System或NetworkService之类的本地服务帐户用作become_user。此限制已从Ansible 2.5版本开始解除。可以在become_user下设置的三个服务帐户是：

- System
- NetworkService
- LocalService

由于本地服务帐户没有密码，因此ansible_become_password参数不是必需的，如果指定则会被忽略。

### 无需设置密码即可提升权限

从 Ansible 2.8 开始，become 可用于提升到 Windows 本地或域帐户的权限，而无需该帐户的密码。要使用此方法，必须满足以下要求：

- 连接用户具有分配的SeDebugPrivilege权限
- 连接用户是BUILTIN\Administrators组的成员
- become_user拥有SeBatchLogonRight或SeNetworkLogonRight用户权限

无需密码即可提升权限的方法有两种：

- 如果帐户已登录，则复制现有登录会话的令牌
- 使用 S4U 生成仅在远程主机上有效的登录令牌

在第一种情况下，提升权限进程是从该用户帐户的另一个登录会话生成的。这可能是现有的 RDP 登录、控制台登录，但这并非总是保证发生的。这类似于计划任务的Run only when user is logged on选项。

如果不存在提升权限帐户的另一个登录会话，则使用 S4U 创建新的登录会话，并通过该会话运行模块。这类似于计划任务的Run whether user is logged on or not以及Do not store password选项。在这种情况下，提升权限进程将无法像普通的 WinRM 进程一样访问任何网络资源。

为了区分使用无密码提升权限和提升没有密码的帐户，请确保将ansible_become_password设置为未定义或设置为ansible_become_password:。

### 没有密码的帐户

Ansible 可用于提升到没有密码的 Windows 帐户（如Guest帐户）。要提升到没有密码的帐户，请像往常一样设置变量，但将ansible_become_password: ''。

在提升权限能够在此类帐户上工作之前，必须禁用本地策略帐户：仅限本地帐户使用空密码进行控制台登录。这可以通过组策略对象 (GPO) 或使用此 Ansible 任务来完成。

```
- name: allow blank password on become
  ansible.windows.win_regedit:
    path: HKLM:\SYSTEM\CurrentControlSet\Control\Lsa
    name: LimitBlankPasswordUse
    data: 0
    type: dword
    state: present
```



### Windows 的提升权限标志

Ansible 2.5 为runas提升权限方法添加了become_flags参数。可以使用become_flags任务指令设置此参数，也可以使用 Ansible 的配置中的ansible_become_flags设置。此参数最初支持的两个有效值为logon_type和logon_flags。

键logon_type设置要执行的登录操作的类型。该值可以设置为以下值之一：

- interactive：默认登录类型。该进程将在与在本地运行进程时相同的上下文中运行。这将绕过所有 WinRM 限制，并且是推荐使用的方法。
- batch：在类似于设置了密码的计划任务的批处理上下文中运行进程。这应该绕过大多数 WinRM 限制，如果become_user不允许交互式登录，则很有用。
- new_credentials：在与调用用户相同的凭据下运行，但出站连接在become_user和become_password上下文中运行，类似于runas.exe /netonly。logon_flags标志也应设置为netcredentials_only。如果进程需要使用不同的凭据访问网络资源（如 SMB 共享），请使用此标志。
- network：在没有任何缓存凭据的网络上下文中运行进程。这将导致与运行没有凭据委派的普通 WinRM 进程相同的登录会话类型，并受相同的限制。
- network_cleartext：类似于network登录类型，但会缓存凭据以便它可以访问网络资源。这与运行具有凭据委派的普通 WinRM 进程的登录会话类型相同。

有关更多信息，请参阅dwLogonType。

logon_flags键指定在创建新进程时 Windows 如何登录用户。该值可以设置为以下值中的一个或多个：

- with_profile：设置的默认登录标志。该进程将在HKEY_USERS注册表键中将用户的配置文件加载到HKEY_CURRENT_USER。
- netcredentials_only：该进程将使用与调用者相同的令牌，但在访问远程资源时将使用become_user和become_password。这在不存在信任关系的跨域场景中很有用，应与new_credentials logon_type一起使用。

默认情况下，设置logon_flags=with_profile，如果不需要加载配置文件，则设置logon_flags=；如果要使用netcredentials_only加载配置文件，则设置logon_flags=with_profile,netcredentials_only。

有关更多信息，请参阅dwLogonFlags。

以下是一些关于如何在 Windows 任务中使用become_flags的示例。

```
- name: copy a file from a fileshare with custom credentials
  ansible.windows.win_copy:
    src: \\server\share\data\file.txt
    dest: C:\temp\file.txt
    remote_src: true
  vars:
    ansible_become: true
    ansible_become_method: runas
    ansible_become_user: DOMAIN\user
    ansible_become_password: Password01
    ansible_become_flags: logon_type=new_credentials logon_flags=netcredentials_only

- name: run a command under a batch logon
  ansible.windows.win_whoami:
  become: true
  become_flags: logon_type=batch

- name: run a command and not load the user profile
  ansible.windows.win_whomai:
  become: true
  become_flags: logon_flags=
```



### Windows 上提升权限的限制

- 在 Windows Server 2008、2008 R2 和 Windows 7 上使用async和become运行任务仅在使用 Ansible 2.7 或更高版本时有效。
- 默认情况下，提升权限用户将以交互式会话登录，因此它必须具有在 Windows 主机上执行此操作的权限。如果它没有继承SeAllowLogOnLocally权限或继承了SeDenyLogOnLocally权限，则提升权限进程将失败。添加权限或设置logon_type标志以更改使用的登录类型。
- 在 Ansible 2.3 版本之前，只有当ansible_winrm_transport为basic或credssp时，提升权限才有效。自 Ansible 2.4 版本起，此限制已对除 Windows Server 2008（非 R2 版本）以外的所有主机解除。
- 必须运行辅助登录服务seclogon才能使用ansible_become_method: runas
- 连接用户必须已经是 Windows 主机上的管理员才能使用runas。但是目标提升权限用户不必是管理员。
