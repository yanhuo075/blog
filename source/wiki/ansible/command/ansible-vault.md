---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 4.9 ansible-vault
order: 8
---

# ansible-vault

Ansible 数据文件的加密/解密实用程序



## 概要

```
usage: ansible-vault [-h] [--version] [-v]
                  {create,decrypt,edit,view,encrypt,encrypt_string,rekey}
                  ...
```



## 描述

可以加密 Ansible 使用的任何结构化数据文件。这包括group_vars/ 或 host_vars/ 清单变量，由include_vars 或 vars_files 加载的变量，或者使用-e @file.yml 或 -e @file.json 在 ansible-playbook 命令行上传递的变量文件。角色变量和默认值也包括在内！

因为 Ansible 任务、处理程序和其他对象都是数据，所以这些也可以用 vault 加密。如果您不想公开正在使用的变量，可以将单个任务文件完全加密。

## 常用选项

- --version

  显示程序的版本号、配置文件位置、已配置的模块搜索路径、模块位置、可执行文件位置并退出

- -h, --help

  显示此帮助消息并退出

- -v, --verbose

  使 Ansible 打印更多调试消息。添加多个 -v 将增加详细程度，内置插件目前最多评估到 -vvvvvv。一个合理的起始级别是 -vvv，连接调试可能需要 -vvvv。此参数可以多次指定。

## 操作



### 创建

创建一个文件并在编辑器中打开，该文件将在关闭时使用提供的 vault 密钥进行加密

- --encrypt-vault-id <ENCRYPT_VAULT_ID>

  用于加密的 vault ID（如果提供多个 vault-id，则需要此选项）

- --skip-tty-check

  允许在没有 tty 连接时打开编辑器

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码



### 解密

使用提供的 vault 密钥解密提供的文件

- --output <OUTPUT_FILE>

  加密或解密的输出文件名；使用 - 表示标准输出

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码



### 编辑

在编辑器中打开并解密现有的已加密文件，该文件将在关闭时再次加密

- --encrypt-vault-id <ENCRYPT_VAULT_ID>

  用于加密的 vault ID（如果提供多个 vault-id，则需要此选项）

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码



### 查看

使用提供的 vault 密钥打开、解密并使用分页器查看现有的已加密文件

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码



### 加密

使用提供的 vault 密钥加密提供的文件

- --encrypt-vault-id <ENCRYPT_VAULT_ID>

  用于加密的 vault ID（如果提供多个 vault-id，则需要此选项）

- --output <OUTPUT_FILE>

  加密或解密的输出文件名；使用 - 表示标准输出

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码



### encrypt_string

使用提供的Vault密钥加密提供的字符串

- --encrypt-vault-id <ENCRYPT_VAULT_ID>

  用于加密的 vault ID（如果提供多个 vault-id，则需要此选项）

- --output <OUTPUT_FILE>

  加密或解密的输出文件名；使用 - 表示标准输出

- --show-input

  提示输入要加密的字符串时，不隐藏输入内容

- --stdin-name <ENCRYPT_STRING_STDIN_NAME>

  指定标准输入的变量名

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码

- -n, --name

  指定变量名。此参数可以多次指定。

- -p, --prompt

  提示输入要加密的字符串



### rekey

使用新的密钥重新加密已加密的文件，需要之前的密钥

- --encrypt-vault-id <ENCRYPT_VAULT_ID>

  用于加密的 vault ID（如果提供多个 vault-id，则需要此选项）

- --new-vault-id <NEW_VAULT_ID>

  重新加密时使用的新的Vault标识

- --new-vault-password-file <NEW_VAULT_PASSWORD_FILE>

  重新加密时使用的新的Vault密码文件

- --vault-id

  要使用的 vault 身份。此参数可以多次指定。

- --vault-password-file, --vault-pass-file

  vault 密码文件

- -J, --ask-vault-password, --ask-vault-pass

  询问 vault 密码

## 环境变量

可以指定以下环境变量。

ANSIBLE_CONFIG – 覆盖默认的Ansible配置文件

Ansible.cfg 中还有许多其他选项。

## 文件

/etc/ansible/ansible.cfg – 如果存在，则使用配置文件

~/.ansible.cfg – 用户配置文件，如果存在，则覆盖默认配置
