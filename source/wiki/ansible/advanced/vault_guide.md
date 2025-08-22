---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.3 Ansible vault 保护敏感数据
order: 2
---

# 使用 Ansible vault 保护敏感数据

欢迎使用 Ansible vault 文档。Ansible vault 提供了一种加密和管理敏感数据（如密码）的方法。本指南将向您介绍 Ansible vault，并涵盖以下主题：

- 管理 vault 密码。
- 使用 Ansible vault 加密内容和文件。
- 使用加密变量和文件。

- Ansible Vault
- 管理 vault 密码
  - 选择单一密码还是多个密码
  - 使用 vault ID 管理多个密码
  - 存储和访问 vault 密码
- 使用 Ansible Vault 加密内容
  - 使用 Ansible Vault 加密单个变量
  - 使用 Ansible Vault 加密文件
- 使用加密变量和文件
  - 传递单个密码
  - 传递 vault ID
  - 传递多个 vault 密码
  - 在没有 vault ID 的情况下使用 --vault-id
- 配置使用加密内容的默认设置
  - 设置默认 vault ID
  - 设置默认密码源
- 何时显示加密文件？
- 使用 Ansible Vault 加密文件的格式
  - Ansible Vault 负载格式 1.1 - 1.2



# Ansible Vault

Ansible Vault 加密变量和文件，以便您可以保护敏感内容（如密码或密钥），而不是将其以明文形式留在剧本或角色中。要使用 Ansible Vault，您需要一个或多个密码来加密和解密内容。如果将 Vault 密码存储在第三方工具（如密钥管理器）中，则需要脚本来访问它们。使用 ansible-vault 命令行工具将密码用于创建和查看加密变量、创建加密文件、加密现有文件，或者编辑、重新加密或解密文件。然后，您可以将加密内容置于源代码控制之下，并更安全地共享它。

您可以通过提供用于加密它们的密码，在临时命令和剧本中使用加密的变量和文件。您可以修改 ansible.cfg 文件以指定密码文件的位置，或者始终提示输入密码。



## 管理 Vault 密码

如果您制定了管理 Vault 密码的策略，则管理加密内容会更容易。Vault 密码可以是您选择的任何字符串。没有特殊的命令来创建 Vault 密码。但是，您需要跟踪您的 Vault 密码。每次使用 Ansible Vault 加密变量或文件时，都必须提供密码。当您在命令或 playbook 中使用加密的变量或文件时，您必须提供用于加密它的同一密码。要制定管理 Vault 密码的策略，请从两个问题开始

> - 您是想用相同的密码加密所有内容，还是为不同的需求使用不同的密码？
> - 您想在哪里存储密码或密码？

### 选择使用单个密码还是多个密码

如果您有一个小型团队或少量敏感值，则可以使用单个密码来加密您使用 Ansible Vault 加密的所有内容。如下所述，将您的 Vault 密码安全地存储在文件或秘密管理器中。

如果您有一个较大的团队或许多敏感值，则可以使用多个密码。例如，您可以为不同的用户或不同的访问级别使用不同的密码。根据您的需求，您可能需要为每个加密文件、每个目录或每个环境使用不同的密码。您可能有一个 playbook，其中包括两个 vars 文件，一个用于开发环境，另一个用于生产环境，使用两个不同的密码加密。当您运行 playbook 时，您可以使用 Vault ID 为您定位的环境选择正确的 Vault 密码。



### 使用 Vault ID 管理多个密码

如果您使用多个 Vault 密码，您可以使用 Vault ID 来区分密码。您可以通过三种方式使用 Vault ID

> - 在您创建加密内容时，将其与 --vault-id 一起传递给 ansible-vault 命令
> - 将其包含在您存储该 Vault ID 密码的任何位置（请参阅 存储和访问 Vault 密码）
> - 当您运行使用使用该 Vault ID 加密的内容的 playbook 时，将其与 --vault-id 一起传递给 ansible-playbook 命令

当您将 Vault ID 作为选项传递给 ansible-vault 命令时，您将标签（提示或昵称）添加到加密内容中。此标签记录了您用于加密它的密码。加密的变量或文件在其标头中以纯文本形式包含 Vault ID 标签。Vault ID 是加密内容之前的最后一个元素。例如

> ```
> my_encrypted_var: !vault |
>           $ANSIBLE_VAULT;1.2;AES256;dev
>           30613233633461343837653833666333643061636561303338373661313838333565653635353162
>           3263363434623733343538653462613064333634333464660a663633623939393439316636633863
>           61636237636537333938306331383339353265363239643939666639386530626330633337633833
>           6664656334373166630a363736393262666465663432613932613036303963343263623137386239
>           6330
> ```

除了标签之外，您还必须为相关密码提供来源。来源可以是提示、文件或脚本，具体取决于您存储 Vault 密码的方式。该模式如下所示

```
--vault-id label@source
```



如果您的 playbook 使用多个使用不同密码加密的加密变量或文件，则在运行该 playbook 时必须传递 Vault ID。您可以单独使用 --vault-id，或与 --vault-password-file 或 --ask-vault-pass 一起使用。该模式与创建加密内容时相同：包括标签和匹配密码的来源。

有关使用 Vault ID 加密内容以及使用使用 Vault ID 加密的内容的示例，请参见下文。 --vault-id 选项适用于与 Vault 交互的任何 Ansible 命令，包括 ansible-vault、ansible-playbook 等。

### Vault ID 的限制

Ansible 不会强制您每次使用特定的 Vault ID 标签时都使用相同的密码。您可以使用相同的 Vault ID 标签但不同的密码加密不同的变量或文件。当您在提示符下输入密码并犯错时，通常会发生这种情况。您可以故意使用具有相同 Vault ID 标签的不同密码。例如，您可以将每个标签用作对一类密码的引用，而不是单个密码。在这种情况下，您必须始终知道在上下文中要使用哪个特定的密码或文件。但是，您更有可能错误地使用相同的 Vault ID 标签和不同的密码加密两个文件。如果您意外地使用相同的标签但不同的密码加密了两个文件，则可以 重新加密 一个文件来解决该问题。

### 强制 Vault ID 匹配

默认情况下，Vault ID 标签只是一个提示，用于提醒您用于加密变量或文件的密码。Ansible 不会检查加密内容标头中的 Vault ID 是否与您在使用内容时提供的 Vault ID 匹配。Ansible 会解密您的命令或 playbook 调用的所有文件和变量，这些文件和变量使用您提供的密码加密。要检查加密内容并仅当其包含的 Vault ID 与您使用 --vault-id 提供的 Vault ID 匹配时才解密，请设置配置选项 DEFAULT_VAULT_ID_MATCH。当您设置 DEFAULT_VAULT_ID_MATCH 时，每个密码仅用于解密使用相同标签加密的数据。这既高效又可预测，并且可以减少使用不同密码加密不同值时出现的错误。



### 存储和访问 Vault 密码

您可以记住您的 Vault 密码或手动从任何来源复制 Vault 密码并将其粘贴到命令行提示符下，但大多数用户会安全地存储它们，并根据需要在 Ansible 中访问它们。您有两个用于在 Ansible 中存储 Vault 密码的选项：文件或第三方工具（例如系统密钥环或秘密管理器）。如果将密码存储在第三方工具中，则需要一个 Vault 密码客户端脚本才能从 Ansible 中检索它们。

### 将密码存储在文件中

要将 Vault 密码存储在文件中，请将密码作为字符串输入到文件中的单行上。确保文件的权限适当。请勿将密码文件添加到源代码控制中。

当您运行使用文件中存储的 Vault 密码的 playbook 时，请在 --vault-password-file 标志中指定该文件。例如

```
ansible-playbook --extra-vars @secrets.enc --vault-password-file secrets.pass
```





### 使用 Vault 密码客户端脚本将密码存储在第三方工具中

您可以将您的 Vault 密码存储在系统密钥环、数据库或秘密管理器中，并使用 Vault 密码客户端脚本从 Ansible 中检索它们。将密码作为字符串输入到单行上。如果您的密码具有 Vault ID，请以与您的密码存储工具兼容的方式存储它。

要创建 Vault 密码客户端脚本

> - 创建一个名称以 -client 或 -client.EXTENSION 结尾的文件
>
> - 使文件可执行
>
> - - 在脚本本身中
>
>     将密码打印到标准输出接受一个 --vault-id 选项如果脚本提示输入数据（例如数据库密码），则将提示显示到 TTY。

当你运行使用存储在第三方工具中的 vault 密码的 playbook 时，请在 --vault-id 标志中指定脚本作为源。例如

```
ansible-playbook --vault-id dev@contrib-scripts/vault/vault-keyring-client.py
```



Ansible 执行客户端脚本时会带上 --vault-id 选项，以便脚本知道你指定的 vault ID 标签。例如，从密钥管理器加载密码的脚本可以使用 vault ID 标签来选择 'dev' 或 'prod' 密码。上面的示例命令会导致以下客户端脚本的执行

```
contrib-scripts/vault/vault-keyring-client.py --vault-id dev
```



有关从系统密钥环加载密码的客户端脚本示例，请参阅 [vault-keyring-client 脚本](https://github.com/ansible-community/contrib-scripts/blob/main/vault/vault-keyring-client.py)。



## 使用 Ansible Vault 加密内容

一旦你制定了管理和存储 vault 密码的策略，就可以开始加密内容了。你可以使用 Ansible Vault 加密两种类型的内容：变量和文件。加密内容始终包含 !vault 标记，该标记告诉 Ansible 和 YAML 需要解密内容，以及一个 | 字符，允许多行字符串。使用 --vault-id 创建的加密内容还包含 vault ID 标签。有关加密过程以及使用 Ansible Vault 加密的内容格式的更多详细信息，请参阅 使用 Ansible Vault 加密文件的格式。此表显示了加密变量和加密文件的主要区别

|                  | 加密变量           | 加密文件           |
| ---------------- | ------------------ | ------------------ |
| 加密了多少内容？ | 纯文本文件中的变量 | 整个文件           |
| 何时解密？       | 按需，仅在需要时   | 加载或引用时1      |
| 可以加密什么？   | 仅变量             | 任何结构化数据文件 |

[1]

除非 Ansible 解密文件，否则 Ansible 无法知道它是否需要加密文件中的内容，因此它会解密 playbook 和角色中引用的所有加密文件。



### 使用 Ansible Vault 加密单个变量

您可以使用 ansible-vault encrypt_string 命令加密 YAML 文件内的单个值。有关安全查看加密变量的一种方法，请参阅 安全查看加密变量。

#### 加密变量的优缺点

使用变量级加密，您的文件仍然易于阅读。您可以混合使用纯文本和加密变量，甚至可以在 playbook 或角色中内联使用。但是，密码轮换不像文件级加密那样简单。您无法 重新加密 加密变量。此外，变量级加密仅适用于变量。如果要加密任务或其他内容，则必须加密整个文件。



#### 创建加密变量

ansible-vault encrypt_string 命令会加密并格式化您键入（或复制或生成）的任何字符串，使其可以包含在 playbook、角色或变量文件中。要创建一个基本的加密变量，请向 ansible-vault encrypt_string 命令传递三个选项

> - vault 密码的来源（提示、文件或脚本，带或不带 vault ID）
> - 要加密的字符串
> - 字符串名称（变量的名称）

模式如下所示

```
ansible-vault encrypt_string <password_source> '<string_to_encrypt>' --name '<string_name_of_variable>'
```



例如，要使用仅存储在“a_password_file”中的密码加密字符串“foobar”，并将变量命名为“the_secret”

```
ansible-vault encrypt_string --vault-password-file a_password_file 'foobar' --name 'the_secret'
```



上述命令创建以下内容

> ```
> the_secret: !vault |
>       $ANSIBLE_VAULT;1.1;AES256
>       62313365396662343061393464336163383764373764613633653634306231386433626436623361
>       6134333665353966363534333632666535333761666131620a663537646436643839616531643561
>       63396265333966386166373632626539326166353965363262633030333630313338646335303630
>       3438626666666137650a353638643435666633633964366338633066623234616432373231333331
>       6564
> ```

要加密字符串“foooodev”，请使用存储在“a_password_file”中的“dev” vault 密码添加 vault ID 标签“dev”，并将加密变量称为“the_dev_secret”

```
ansible-vault encrypt_string --vault-id dev@a_password_file 'foooodev' --name 'the_dev_secret'
```



上述命令创建以下内容

> ```
> the_dev_secret: !vault |
>           $ANSIBLE_VAULT;1.2;AES256;dev
>           30613233633461343837653833666333643061636561303338373661313838333565653635353162
>           3263363434623733343538653462613064333634333464660a663633623939393439316636633863
>           61636237636537333938306331383339353265363239643939666639386530626330633337633833
>           6664656334373166630a363736393262666465663432613932613036303963343263623137386239
>           6330
> ```

要加密从 stdin 读取的字符串“letmein”，使用存储在 a_password_file 中的“dev” vault 密码添加 vault ID“dev”，并将变量命名为“db_password”

```
echo -n 'letmein' | ansible-vault encrypt_string --vault-id dev@a_password_file --stdin-name 'db_password'
```



上述命令创建以下输出

> ```
> Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a new line)
> db_password: !vault |
> ANSIBLE_VAULT;1.2;AES256;dev
>           61323931353866666336306139373937316366366138656131323863373866376666353364373761
>           3539633234313836346435323766306164626134376564330a373530313635343535343133316133
>           36643666306434616266376434363239346433643238336464643566386135356334303736353136
>           6565633133366366360a326566323363363936613664616364623437336130623133343530333739
>           3039
> ```

要提示输入要加密的字符串，请使用来自“a_password_file”的“dev” vault 密码对其进行加密，将变量命名为“new_user_password”并赋予其 vault ID 标签“dev”

```
ansible-vault encrypt_string --vault-id dev@a_password_file --stdin-name 'new_user_password'
```



上述命令会触发此提示

```
Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a new line)
```



键入要加密的字符串（例如，“hunter2”），按 ctrl-d，然后等待。

上述序列创建以下输出

> ```
> new_user_password: !vault |
>           $ANSIBLE_VAULT;1.2;AES256;dev
>           37636561366636643464376336303466613062633537323632306566653533383833366462366662
>           6565353063303065303831323539656138653863353230620a653638643639333133306331336365
>           62373737623337616130386137373461306535383538373162316263386165376131623631323434
>           3866363862363335620a376466656164383032633338306162326639643635663936623939666238
>           3161
> ```

您可以将上述任何示例中的输出添加到任何 playbook、变量文件或角色以供将来使用。加密变量比纯文本变量更大，但它们可以保护您的敏感内容，同时使 playbook、变量文件或角色的其余部分保持纯文本，以便您可以轻松阅读。

### 查看加密变量

您可以使用 debug 模块查看加密变量的原始值。您必须传递用于加密变量的密码。例如，如果您将上述最后一个示例创建的变量存储在名为“vars.yml”的文件中，则可以像这样查看该变量的未加密值

```
ansible localhost -m ansible.builtin.debug -a var="new_user_password" -e "@vars.yml" --vault-id dev@a_password_file

localhost | SUCCESS => {
    "new_user_password": "hunter2"
}
```



## 使用 Ansible Vault 加密文件

Ansible Vault 可以加密 Ansible 使用的任何结构化数据文件，包括

> - 清单中的组变量文件
> - 清单中的主机变量文件
> - 使用 -e @file.yml 或 -e @file.json 传递给 ansible-playbook 的变量文件
> - 由 include_vars 或 vars_files 加载的变量文件
> - 角色中的变量文件
> - 角色中的默认文件
> - 任务文件
> - 处理程序文件
> - 二进制文件或其他任意文件

整个文件都加密在 vault 中。

### 加密文件的优缺点

文件级加密易于使用。使用 重新加密 命令，加密文件的密码轮换非常简单。加密文件不仅可以隐藏敏感值，还可以隐藏您使用的变量的名称。但是，使用文件级加密，文件的内容不再易于访问和读取。这在加密任务文件时可能是一个问题。加密变量文件时，请参阅 安全查看加密变量，了解一种在非加密文件中保留对这些变量的引用的方法。Ansible 始终会在加载或引用加密文件时解密整个加密文件，因为除非 Ansible 解密它，否则 Ansible 无法知道它是否需要该内容。



### 创建加密文件

要使用来自“multi_password_file”的“test” vault 密码创建一个名为“foo.yml”的新加密数据文件

```
ansible-vault create --vault-id test@multi_password_file foo.yml
```



该工具将启动一个编辑器（您使用 $EDITOR 定义的任何编辑器，默认编辑器为 vi）。添加内容。关闭编辑器会话时，文件将保存为加密数据。文件标题反映了用于创建它的 vault ID

```
``$ANSIBLE_VAULT;1.2;AES256;test``
```



要创建一个新的加密数据文件，并为其分配 vault ID“my_new_password”，并提示输入密码

```
ansible-vault create --vault-id my_new_password@prompt foo.yml
```



同样，在编辑器中向文件添加内容并保存。请务必存储您在提示中创建的新密码，以便您在想要解密该文件时可以找到它。



### 加密现有文件

要加密现有文件，请使用ansible-vault encrypt命令。此命令可以同时操作多个文件。例如

```
ansible-vault encrypt foo.yml bar.yml baz.yml
```



使用“project”ID加密现有文件，并提示输入密码

```
ansible-vault encrypt --vault-id project@prompt foo.yml bar.yml baz.yml
```





### 查看加密文件

要查看加密文件的内容而不进行编辑，可以使用ansible-vault view命令

```
ansible-vault view foo.yml bar.yml baz.yml
```





### 编辑加密文件

要就地编辑加密文件，请使用ansible-vault edit命令。此命令会将文件解密到临时文件，允许您编辑内容，然后保存并重新加密内容，并在关闭编辑器时删除临时文件。例如

```
ansible-vault edit foo.yml
```



编辑使用vault2密码文件加密并分配了vault ID pass2的文件

```
ansible-vault edit --vault-id pass2@vault2 foo.yml
```





### 更改加密文件的密码和/或vault ID

要更改加密文件或文件的密码，请使用rekey命令

```
ansible-vault rekey foo.yml bar.yml baz.yml
```



此命令可以一次重新加密多个数据文件，并将询问原始密码和新密码。要为重新加密的文件设置不同的ID，请将新ID传递给--new-vault-id。例如，要将使用“preprod1”vault ID加密的文件列表从“ppold”文件重新加密到“preprod2”vault ID，并提示输入新密码

```
ansible-vault rekey --vault-id preprod1@ppold --new-vault-id preprod2@prompt foo.yml bar.yml baz.yml
```





### 解密加密文件

如果您有一个不再需要加密的加密文件，您可以通过运行ansible-vault decrypt命令永久解密它。此命令会将文件未加密地保存到磁盘，因此请确保您不想edit它。

```
ansible-vault decrypt foo.yml bar.yml baz.yml
```





### 解密加密字符串

如果您只想检查加密字符串的内容，也可以通过stdin传递它来查看。

```
echo -e '$ANSIBLE_VAULT;1.1;V2\neyJrZXkiOiAiZ0FBQUFBQm5UYzlPUVgzeUc5NFo3R2pzYVNMSXVsdXA3Z0paMmczNVRtS0NqMUcwMTVx\nSU1JVDlJZlRrSXBkVThmLXhKS00xZGl6X3F3YXZmWWUteGJWaHNZZXZNWl9hMWZvLVRYM3ZUZDRvaHRR\nWkhIdkJmZEZWNlBwVjhNVjJFT05QbDFwandaazAiLCAiY2lwaGVydGV4dCI6ICJnQUFBQUFCblRjOU9u\nWmM2dDh2VEN5c3NTQVlyV0hMclNEOFZfSGd2eEVHdERCdkJfakFpcUpaWWNTV19sR2hPY0VsWEVweS0z\nQ0NBcmJfdUdsUEt0NzJuSmFxVVVmRFIzdz09In0=' | ansible-vault decrypt
```



或者，您可以使用以下命令让Ansible提示您输入（使用Ctrl+D两次结束输入），就像使用encrypt_string一样

```
ansible-vault decrypt
Reading ciphertext input from stdin
```





### 保护编辑器的步骤

Ansible Vault依赖于您配置的编辑器，这可能是信息泄露的来源。大多数编辑器都有防止数据丢失的方法，但这些方法通常依赖于额外的纯文本文件，这些文件可能包含您的密钥的明文副本。请参阅您的编辑器文档以配置编辑器，避免泄露安全数据。以下部分提供了一些关于常用编辑器的指导，但不应将其视为保护编辑器的完整指南。

#### vim

您可以在命令模式下设置以下vim选项以避免信息泄露的情况。您可能需要修改更多设置以确保安全，尤其是在使用插件时，因此请查阅vim文档。

1. 禁用充当崩溃或中断时自动保存的交换文件。

```
set noswapfile
```



1. 禁用备份文件的创建。

```
set nobackup
set nowritebackup
```



1. 禁用viminfo文件复制当前会话中的数据。

```
set viminfo=
```



1. 禁用复制到系统剪贴板。

```
set clipboard=
```



您可以选择在.vimrc中为所有文件或特定路径或扩展名添加这些设置。有关详细信息，请参阅vim手册。

#### Emacs

您可以设置以下Emacs选项以避免信息泄露的情况。您可能需要修改更多设置以确保安全，尤其是在使用插件时，因此请查阅Emacs文档。

1. 不要将数据复制到系统剪贴板。

```
(setq x-select-enable-clipboard nil)
```



1. 禁用备份文件的创建。

```
(setq make-backup-files nil)
```



1. 禁用自动保存文件。

```
(setq auto-save-default nil)
```



## 使用加密变量和文件

运行使用加密变量或文件的任务或 playbook 时，必须提供用于解密变量或文件的密码。这可以通过命令行完成，也可以通过在配置选项或环境变量中设置默认密码来源来完成。

### 传递单个密码

如果任务或 playbook 中所有加密变量和文件都需要使用单个密码，则可以使用 --ask-vault-pass 或 --vault-password-file 命令行选项。

提示输入密码

```
ansible-playbook --ask-vault-pass site.yml
```



从 /path/to/my/vault-password-file 文件中检索密码

```
ansible-playbook --vault-password-file /path/to/my/vault-password-file site.yml
```



从 vault 密码客户端脚本 my-vault-password-client.py 获取密码

```
ansible-playbook --vault-password-file my-vault-password-client.py
```



### 传递 vault ID

也可以使用 --vault-id 选项来传递带有其 vault 标签的单个密码。当单个清单中使用多个 vault 时，这种方法更清晰。

提示输入 “dev” vault ID 的密码

```
ansible-playbook --vault-id dev@prompt site.yml
```



从 dev-password 文件中检索 “dev” vault ID 的密码

```
ansible-playbook --vault-id dev@dev-password site.yml
```



从 vault 密码客户端脚本 my-vault-password-client.py 获取 “dev” vault ID 的密码

```
ansible-playbook --vault-id [email protected]
```



### 传递多个 vault 密码

如果任务或 playbook 需要多个使用不同 vault ID 加密的加密变量或文件，则必须使用 --vault-id 选项，传递多个 --vault-id 选项来指定 vault ID（“dev”、“prod”、“cloud”、“db”）和密码来源（提示、文件、脚本）。例如，要使用从文件中读取的 “dev” 密码并提示输入 “prod” 密码

```
ansible-playbook --vault-id dev@dev-password --vault-id prod@prompt site.yml
```



默认情况下，vault ID 标签（dev、prod 等）只是提示。Ansible 会尝试使用每个密码解密 vault 内容。与加密数据标签相同的密码将首先尝试，之后，每个 vault 密钥将按它们在命令行中提供的顺序尝试。

如果加密数据没有标签，或者标签与任何提供的标签都不匹配，则将按指定的顺序尝试密码。在上面的示例中，将首先尝试 “dev” 密码，然后尝试 “prod” 密码，用于 Ansible 不知道使用哪个 vault ID 加密某些内容的情况。

### 使用 --vault-id 但不指定 vault ID

--vault-id 选项也可以不指定 vault-id 使用。此行为等效于 --ask-vault-pass 或 --vault-password-file，因此很少使用。

例如，使用密码文件 dev-password

```
ansible-playbook --vault-id dev-password site.yml
```



提示输入密码

```
ansible-playbook --vault-id @prompt site.yml
```



从可执行脚本 my-vault-password-client.py 获取密码

```
ansible-playbook --vault-id my-vault-password-client.py
```



## 配置使用加密内容的默认值

### 设置默认 vault ID

如果比其他任何 vault ID 更频繁地使用一个 vault ID，则可以将配置选项 DEFAULT_VAULT_IDENTITY_LIST 设置为指定默认 vault ID 和密码来源。在未指定 --vault-id 时，Ansible 将使用默认 vault ID 和来源。可以为此选项设置多个值。设置多个值等效于传递多个 --vault-id 命令行选项。

### 设置默认密码来源

如果不想在命令行上提供密码文件，或者比其他任何 vault 密码文件更频繁地使用一个 vault 密码文件，则可以设置 DEFAULT_VAULT_PASSWORD_FILE 配置选项或 ANSIBLE_VAULT_PASSWORD_FILE 环境变量来指定要使用的默认文件。例如，如果设置 ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt，Ansible 将自动在该文件中搜索密码。这在例如从 Jenkins 等持续集成系统使用 Ansible 时非常有用。

引用的文件可以是包含密码（纯文本）的文件，也可以是返回密码的脚本（已设置可执行权限）。

### 何时显示加密文件？

通常情况下，使用 Ansible Vault 加密的内容在执行后仍然保持加密状态。但是，有一个例外。如果将加密文件作为 src 参数传递给 copy、template、unarchive、script 或 assemble 模块，则该文件不会在目标主机上加密（假设在运行 play 时提供了正确的 vault 密码）。此行为是预期的且有用的。可以加密配置文件或模板以避免共享配置详细信息，但在将该配置复制到环境中的服务器时，需要对其进行解密，以便本地用户和进程可以访问它。



## 使用 Ansible Vault 加密的文件格式

Ansible Vault 创建 UTF-8 编码的 txt 文件。文件格式包括以换行符结尾的标题。例如

> ```
> $ANSIBLE_VAULT;1.1;AES256
> ```

或

> ```
> $ANSIBLE_VAULT;1.2;AES256;vault-id-label
> ```

标题包含最多四个元素，用分号 (;) 分隔。

> 1. 格式 ID ($ANSIBLE_VAULT)。目前 $ANSIBLE_VAULT 是唯一有效的格式 ID。格式 ID 用于标识使用 Ansible Vault 加密的内容 (使用 vault.is_encrypted_file())。
> 2. 保管库格式版本（1.X）。所有受支持的 Ansible 版本目前在提供标记的保管库 ID 时将默认为“1.1”或“1.2”。“1.0”格式仅支持读取（写入时会自动转换为“1.1”格式）。格式版本目前仅用作精确字符串比较（版本号目前不进行“比较”）。
> 3. 用于加密数据的密码算法（AES256）。目前AES256是唯一支持的密码算法。保管库格式 1.0 使用“AES”，但当前代码始终使用“AES256”。
> 4. 用于加密数据的保管库 ID 标签（可选，vault-id-label）例如，如果使用--vault-id dev@prompt加密文件，则保管库 ID 标签为dev。

注意：将来，标题可能会更改。格式 ID 和格式版本之后的字段取决于格式版本。未来的保管库格式版本可能会添加更多密码算法选项和/或其他字段。

文件的其余内容是“vaulttext”。vaulttext 是加密密文的文本装甲版本。每行 80 个字符宽，最后一行的长度可能较短。

### Ansible 保管库有效负载格式 1.1 - 1.2

vaulttext 是密文和 SHA256 摘要的串联，结果“进行了十六进制转换”。

“十六进制转换”指的是 Python 标准库的binascii 模块的hexlify()方法。

经过十六进制转换的结果为：

- 盐的十六进制转换字符串，后跟换行符（0x0a）
- 加密 HMAC 的十六进制转换字符串，后跟换行符。HMAC 是
  - 一个RFC2104 风格的 HMAC
    - 输入是：
      - AES256 加密的密文
      - PBKDF2 密钥。此密钥、密码密钥和密码 IV 由以下内容生成：
        - 盐（以字节为单位）
        - 10000 次迭代
        - SHA256() 算法
        - 前 32 个字节是密码密钥
        - 接下来的 32 个字节是 HMAC 密钥
        - 其余 16 个字节是密码 IV
- 密文的十六进制转换字符串。密文是

> - AES256 加密的数据。数据使用以下方式加密：
>   - AES-CTR 流密码
>   - 密码密钥
>   - IV
>   - 一个从整数 IV 播种的 128 位计数器块
>   - 明文
>     - 原始明文
>     - 填充到 AES256 块大小。（用于填充的数据基于RFC5652）



