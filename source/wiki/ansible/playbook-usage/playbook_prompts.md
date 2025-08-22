---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 5.17 交互提示
order: 16
---

# 交互式输入：提示

如果希望 Playbook 提示用户输入某些内容，请添加一个“vars_prompt”部分。提示用户输入变量可以避免记录诸如密码之类的敏感数据。除了安全性之外，提示还支持灵活性。例如，如果您在多个软件版本中使用一个 Playbook，则可以提示输入特定的发布版本。

- 哈希 vars_prompt 提供的值
- 允许在 vars_prompt 值中使用特殊字符

这是一个最基本的示例

```
---
- hosts: all
  vars_prompt:

    - name: username
      prompt: What is your username?
      private: false

    - name: password
      prompt: What is your password?

  tasks:

    - name: Print a message
      ansible.builtin.debug:
        msg: 'Logging in as {{ username }}'
```



默认情况下，用户输入是隐藏的，但可以通过设置 private: false 使其可见。

如果您有一个不经常更改的变量，您可以提供一个可以被覆盖的默认值。

```
vars_prompt:

  - name: release_version
    prompt: Product release version
    default: "1.0"
```



## 哈希 vars_prompt 提供的值

您可以哈希输入的值，以便您可以将其与用户模块一起使用以定义密码，例如

```
vars_prompt:

  - name: my_password2
    prompt: Enter password2
    private: true
    encrypt: sha512_crypt
    confirm: true
    salt_size: 7
```



如果您安装了 Passlib，您可以使用该库支持的任何加密方案

- des_crypt - DES 加密
- bsdi_crypt - BSDi 加密
- bigcrypt - BigCrypt
- crypt16 - Crypt16
- md5_crypt - MD5 加密
- bcrypt - BCrypt
- sha1_crypt - SHA-1 加密
- sun_md5_crypt - Sun MD5 加密
- sha256_crypt - SHA-256 加密
- sha512_crypt - SHA-512 加密
- apr_md5_crypt - Apache 的 MD5-Crypt 变体
- phpass - PHPass 的可移植哈希
- pbkdf2_digest - 通用 PBKDF2 哈希
- cta_pbkdf2_sha1 - Cryptacular 的 PBKDF2 哈希
- dlitz_pbkdf2_sha1 - Dwayne Litzenberger 的 PBKDF2 哈希
- scram - SCRAM 哈希
- bsd_nthash - FreeBSD 的 MCF 兼容 nthash 编码

唯一接受的参数是“salt”或“salt_size”。您可以通过定义“salt”来使用自己的 salt，或者使用“salt_size”自动生成一个 salt。默认情况下，Ansible 生成大小为 8 的 salt。

2.7 版本中的新增功能。

如果您没有安装 Passlib，Ansible 将使用 crypt 库作为后备。Ansible 最多支持四种加密方案，具体取决于您的平台，最多支持以下加密方案

- bcrypt - BCrypt
- md5_crypt - MD5 加密
- sha256_crypt - SHA-256 加密
- sha512_crypt - SHA-512 加密

2.8 版本中的新增功能。



## 允许在 vars_prompt 值中使用特殊字符

某些特殊字符，例如 { 和 %，可能会创建模板错误。如果您需要接受特殊字符，请使用 unsafe 选项

```
vars_prompt:
  - name: my_password_with_weird_chars
    prompt: Enter password
    unsafe: true
    private: true
```
