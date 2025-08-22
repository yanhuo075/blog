---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 7.6 Ansible 集合使用
order: 5
---

# 使用 Ansible 集合

欢迎来到 Ansible 使用集合指南。

集合是 Ansible 内容的一种分发格式，其中可以包含剧本、角色、模块和插件。您可以通过分发服务器（例如 Ansible Galaxy 或 Pulp 3 Galaxy 服务器）安装和使用集合。

- 安装集合
  - 在容器中安装集合
  - 使用 ansible-galaxy 安装集合
  - 使用签名验证安装集合
  - 安装旧版本的集合
  - 使用需求文件安装多个集合
  - 下载集合以供离线使用
  - 在剧本旁边安装集合
  - 从源文件安装集合
  - 从 Git 仓库安装集合
  - 配置 ansible-galaxy 客户端
- 删除集合
- 下载集合
- 列出集合
- 验证集合
  - 使用 ansible-galaxy 验证集合
  - 验证已签名的集合
- 在剧本中使用集合
  - 使用 collections 关键字简化模块名称
  - 在角色中使用 collections
  - 在剧本中使用 collections
  - 使用集合中的剧本



## 安装集合

### 在容器中安装集合

您可以在称为执行环境的容器中安装集合及其依赖项。有关详细信息，请参阅 执行环境入门指南。

#### 使用 ansible-galaxy 安装集合

默认情况下，ansible-galaxy collection install 使用 https://galaxy.ansible.com 作为 Galaxy 服务器（如 ansible.cfg 文件中 GALAXY_SERVER 中所列）。您不需要任何其他配置。默认情况下，Ansible 将集合安装在 ~/.ansible/collections 下的 ansible_collections 目录中。

如果您使用的是任何其他 Galaxy 服务器（例如 Red Hat Automation Hub），请参阅 配置 ansible-galaxy 客户端。

要安装在 Galaxy 中托管的集合：

```
ansible-galaxy collection install my_namespace.my_collection
```



要将集合升级到 Galaxy 服务器上最新的可用版本，您可以使用 --upgrade 选项

```
ansible-galaxy collection install my_namespace.my_collection --upgrade
```



您也可以直接使用您构建的 tarball 包

```
ansible-galaxy collection install my_namespace-my_collection-1.0.0.tar.gz -p ./collections
```



您可以从本地源目录构建和安装集合。 ansible-galaxy 实用程序使用目录中的 MANIFEST.json 或 galaxy.yml 元数据来构建集合。

```
ansible-galaxy collection install /path/to/collection -p ./collections
```



您也可以在命名空间目录中安装多个集合。

```
ns/
├── collection1/
│   ├── MANIFEST.json
│   └── plugins/
└── collection2/
    ├── galaxy.yml
    └── plugins/
```



```
ansible-galaxy collection install /path/to/ns -p ./collections
```



使用 -p 选项指定安装路径时，请使用 COLLECTIONS_PATHS 中配置的值之一，因为 Ansible 本身将在此处查找集合。如果您未指定路径，ansible-galaxy collection install 将集合安装到 COLLECTIONS_PATHS 中定义的第一个路径，默认情况下为 ~/.ansible/collections



#### 使用签名验证安装集合

如果集合已由 分发服务器 签名，则服务器将提供 ASCII 编码的独立签名，以验证 MANIFEST.json 的真实性，然后再使用它来验证集合的内容。并非所有分发服务器都提供此选项。有关支持集合签名的服务器列表，请参阅 分发集合。

要对已签名的集合使用签名验证：

1. 配置 GnuPG 密钥环 用于 ansible-galaxy，或者在安装已签名的集合时使用 --keyring 选项提供密钥环的路径。

2. 将公钥从分发服务器导入到该密钥环。

   ```
   gpg --import --no-default-keyring --keyring ~/.ansible/pubring.kbx my-public-key.asc
   ```

   

3. 安装集合时验证签名。

   ```
   ansible-galaxy collection install my_namespace.my_collection --keyring ~/.ansible/pubring.kbx
   ```

   

   如果您已 配置 GnuPG 密钥环，则无需使用 --keyring 选项。

4. 或者，在安装后随时验证签名，以证明集合未被篡改。有关详细信息，请参阅 验证已签名的集合。

您还可以包含除分发服务器提供的签名之外的其他签名。使用 --signature 选项使用这些附加签名来验证集合的 MANIFEST.json。补充签名应作为 URI 提供。

```
ansible-galaxy collection install my_namespace.my_collection --signature https://examplehost.com/detached_signature.asc --keyring ~/.ansible/pubring.kbx
```



GnuPG 验证仅对从分发服务器安装的集合进行。用户提供的签名不用于验证从 Git 仓库、源目录或 URL/tar.gz 文件路径安装的集合。

您还可以将附加签名包含在集合的 requirements.yml 文件的 signatures 密钥下。

```
# requirements.yml
collections:
  - name: ns.coll
    version: 1.0.0
    signatures:
      - https://examplehost.com/detached_signature.asc
      - file:///path/to/local/detached_signature.asc
```



有关如何使用此文件安装集合的详细信息，请参阅 集合 requirements 文件。

默认情况下，如果至少 1 个签名成功验证了集合，则验证被认为成功。所需签名的数量可以使用 --required-valid-signature-count 或 GALAXY_REQUIRED_VALID_SIGNATURE_COUNT 进行配置。可以通过将选项设置为 all 来要求所有签名。如果未找到有效签名，则要使签名验证失败，请在值前加上 +，例如 +all 或 +1。

```
export ANSIBLE_GALAXY_GPG_KEYRING=~/.ansible/pubring.kbx
export ANSIBLE_GALAXY_REQUIRED_VALID_SIGNATURE_COUNT=2
ansible-galaxy collection install my_namespace.my_collection --signature https://examplehost.com/detached_signature.asc --signature file:///path/to/local/detached_signature.asc
```



可以使用 --ignore-signature-status-code 或 GALAXY_REQUIRED_VALID_SIGNATURE_COUNT 忽略某些 GnuPG 错误。GALAXY_REQUIRED_VALID_SIGNATURE_COUNT 应为列表，并且可以多次提供 --ignore-signature-status-code 以忽略多个其他错误状态代码。

此示例要求分发服务器提供的任何签名来验证集合，除非它们由于 NO_PUBKEY 而失败

```
export ANSIBLE_GALAXY_GPG_KEYRING=~/.ansible/pubring.kbx
export ANSIBLE_GALAXY_REQUIRED_VALID_SIGNATURE_COUNT=all
ansible-galaxy collection install my_namespace.my_collection --ignore-signature-status-code NO_PUBKEY
```



如果上述示例的验证失败，则只会显示除 NO_PUBKEY 之外的错误。

如果验证不成功，则不会安装集合。可以使用 --disable-gpg-verify 或通过配置 GALAXY_DISABLE_GPG_VERIFY 来禁用 GnuPG 签名验证。



#### 安装旧版本的集合

您一次只能安装一个版本的集合。默认情况下，ansible-galaxy 安装最新的可用版本。如果要安装特定版本，可以添加版本范围标识符。例如，要安装集合的 1.0.0-beta.1 版本：

```
ansible-galaxy collection install my_namespace.my_collection:==1.0.0-beta.1
```



您可以指定多个用 , 分隔的范围标识符。使用单引号，以便 shell 传递整个命令，包括 >、! 和其他运算符。例如，要安装大于或等于 1.0.0 且小于 2.0.0 的最新版本：

```
ansible-galaxy collection install 'my_namespace.my_collection:>=1.0.0,<2.0.0'
```



Ansible 将始终安装满足您指定的范围标识符的最新版本。您可以使用以下范围标识符：

- *：最新版本。这是默认值。
- !=：不等于指定的版本。
- ==：与指定的版本完全相同。
- >=：大于或等于指定的版本。
- >：大于指定的版本。
- <=：小于或等于指定的版本。
- <：小于指定的版本。



### 使用 requirements 文件安装多个集合

您可以设置一个 requirements.yml 文件，以便使用一条命令安装多个集合。此文件是一个 YAML 文件，格式如下：

```
---
collections:
# With just the collection name
- my_namespace.my_collection

# With the collection name, version, and source options
- name: my_namespace.my_other_collection
  version: ">=1.2.0" # Version range identifiers (default: ``*``)
  source: ... # The Galaxy URL to pull the collection from (default: ``--api-server`` from cmdline)
```



您可以为每个集合条目指定以下键：

> - name
> - version
> - signatures
> - source
> - type

version 键使用与安装旧版本的集合中记录的相同的范围标识符格式。

signatures 键接受一个签名源列表，这些源用于补充在集合安装期间和 ansible-galaxy collection verify 过程中在 Galaxy 服务器上找到的签名源。签名源应为包含分离签名的 URI。--keyring CLI 选项必须在指定签名时提供。

签名仅用于验证 Galaxy 服务器上的集合。用户提供的签名不用于验证从 git 仓库、源目录或 tar.gz 文件的 URL/路径安装的集合。

```
collections:
  - name: namespace.name
    version: 1.0.0
    type: galaxy
    signatures:
      - https://examplehost.com/detached_signature.asc
      - file:///path/to/local/detached_signature.asc
```



type 键可以设置为 file、galaxy、git、url、dir 或 subdirs。如果省略 type，则 name 键用于隐式确定集合的来源。

使用 type: git 安装集合时，version 键可以引用分支或git commit-ish 对象（提交或标签）。例如：

```
collections:
  - name: https://github.com/organization/repo_name.git
    type: git
    version: devel
```



您还可以将角色添加到 requirements.yml 文件中，位于 roles 键下。其值遵循旧版 Ansible 版本中使用的 requirements 文件的相同格式。

```
---
roles:
  # Install a role from Ansible Galaxy.
  - name: geerlingguy.java
    version: "1.9.6" # note that ranges are not supported for roles


collections:
  # Install a collection from Ansible Galaxy.
  - name: geerlingguy.php_roles
    version: ">=0.9.3"
    source: https://galaxy.ansible.com
```



要同时使用一条命令安装角色和集合，请运行以下命令：

```
$ ansible-galaxy install -r requirements.yml
```



运行 ansible-galaxy collection install -r 或 ansible-galaxy role install -r 将只分别安装集合或角色。



### 下载集合以供离线使用

要下载 Galaxy 中的集合 tarball 以供离线使用：

1. 导航到集合页面。
2. 点击下载 tarball。

您可能还需要手动下载任何依赖的集合。



### 在剧本旁边安装集合

您可以将集合在本地安装到项目中的剧本旁边，而不是安装到系统或 AWX 上的全局位置。

在剧本旁边使用本地安装的集合有一些好处，例如：

- 确保项目的所有用户都使用相同的集合版本。
- 使用自包含项目可以轻松地在不同的环境之间移动。更高的可移植性还可以减少设置新环境时的开销。在云环境中部署 Ansible 剧本时，这是一个优势。
- 本地管理集合可以让您将它们与剧本一起版本化。
- 本地安装集合可以将它们与具有多个项目的环境中的全局安装隔离开来。

以下是如何在 collections/ansible_collections/ 目录结构下将集合保存在当前剧本旁边的示例。

```
./
├── play.yml
├── collections/
│   └── ansible_collections/
│               └── my_namespace/
│                   └── my_collection/<collection structure lives here>
```



有关集合目录结构的详细信息，请参阅集合结构。

### 从源文件安装集合

Ansible 还可以通过多种方式从源目录安装：

```
collections:
  # directory containing the collection
  - name: ./my_namespace/my_collection/
    type: dir

  # directory containing a namespace, with collections as subdirectories
  - name: ./my_namespace/
    type: subdirs
```



Ansible 还可以通过直接指定输出文件来安装使用 ansible-galaxy collection build 收集或从 Galaxy 下载以供离线使用的集合。

```
collections:
  - name: /tmp/my_namespace-my_collection-1.0.0.tar.gz
    type: file
```



### 从 Git 仓库安装集合

您可以从 Git 仓库安装集合，而不是从 Galaxy 或 Automation Hub 安装。作为开发人员，从 Git 仓库安装可以让您在创建 tarball 并发布集合之前查看您的集合。作为用户，从 Git 仓库安装可以让您使用 Galaxy 或 Automation Hub 中尚不存在的集合或版本。此功能旨在作为前面所述的内容开发人员的最小快捷方式，并且 Git 仓库可能不支持 ansible-galaxy CLI 的全部功能。在复杂情况下，更灵活的选择可能是将 git clone 仓库到集合安装目录的正确文件结构中。

仓库必须包含 galaxy.yml 或 MANIFEST.json 文件。此文件提供元数据，例如集合的版本号和命名空间。

#### 在命令行中从 Git 仓库安装集合

要在命令行中从 Git 仓库安装集合，请使用仓库的 URI，而不是集合名称或 tar.gz 文件的路径。使用前缀 git+，除非您使用具有用户 git 的 SSH 身份验证（例如，git@github.com:ansible-collections/ansible.windows.git）。您可以使用逗号分隔的git commit-ish语法指定分支、提交或标签。

例如：

```
# Install a collection in a repository using the latest commit on the branch 'devel'
ansible-galaxy collection install git+https://github.com/organization/repo_name.git,devel

# Install a collection from a private GitHub repository
ansible-galaxy collection install [email protected]:organization/repo_name.git

# Install a collection from a local git repository
ansible-galaxy collection install git+file:///home/user/path/to/repo_name.git
```



#### 指定 Git 仓库中的集合位置

从 Git 仓库安装集合时，Ansible 使用集合 galaxy.yml 或 MANIFEST.json 元数据文件来构建集合。默认情况下，Ansible 会搜索两条路径以查找集合 galaxy.yml 或 MANIFEST.json 元数据文件：

- 仓库的顶层。
- 仓库路径中的每个目录（一层深）。

如果在仓库的顶层存在 galaxy.yml 或 MANIFEST.json 文件，则 Ansible 使用该文件中的集合元数据来安装单个集合。

```
├── galaxy.yml
├── plugins/
│   ├── lookup/
│   ├── modules/
│   └── module_utils/
└─── README.md
```



如果在仓库路径中的一个或多个目录（一层深）中存在 galaxy.yml 或 MANIFEST.json 文件，则 Ansible 会将每个具有元数据文件的目录作为集合安装。例如，Ansible 默认情况下会从此仓库结构安装集合 1 和集合 2：

```
├── collection1
│   ├── docs/
│   ├── galaxy.yml
│   └── plugins/
│       ├── inventory/
│       └── modules/
└── collection2
    ├── docs/
    ├── galaxy.yml
    ├── plugins/
    |   ├── filter/
    |   └── modules/
    └── roles/
```



如果您有不同的仓库结构，或者只想安装一部分集合，您可以将一个片段添加到 URI 的末尾（在可选的逗号分隔版本之前），以指示元数据文件或文件的位置。路径应为目录，而不是元数据文件本身。例如，要仅从具有两个集合的示例仓库安装集合 2：

```
ansible-galaxy collection install git+https://github.com/organization/repo_name.git#/collection2/
```



在某些仓库中，主目录对应于命名空间。

```
namespace/
├── collectionA/
|   ├── docs/
|   ├── galaxy.yml
|   ├── plugins/
|   │   ├── README.md
|   │   └── modules/
|   ├── README.md
|   └── roles/
└── collectionB/
    ├── docs/
    ├── galaxy.yml
    ├── plugins/
    │   ├── connection/
    │   └── modules/
    ├── README.md
    └── roles/
```



您可以安装此仓库中的所有集合，或从特定提交安装一个集合。

```
# Install all collections in the namespace
ansible-galaxy collection install git+https://github.com/organization/repo_name.git#/namespace/

# Install an individual collection using a specific commit
ansible-galaxy collection install git+https://github.com/organization/repo_name.git#/namespace/collectionA/,7b60ddc245bc416b72d8ea6ed7b799885110f5e5
```





### 配置 ansible-galaxy 客户端

默认情况下，ansible-galaxy 使用 https://galaxy.ansible.com 作为 Galaxy 服务器（如 ansible.cfg 文件中 GALAXY_SERVER 所列）。

您可以使用以下任一选项来配置 ansible-galaxy collection 以使用其他服务器（例如自定义 Galaxy 服务器）。

- 在 GALAXY_SERVER_LIST 配置选项（位于 配置文件 中）中设置服务器列表。
- 使用 --server 命令行参数限制到单个服务器。

要在 ansible.cfg 中配置 Galaxy 服务器列表：

1. 在 [galaxy] 部分下添加 server_list 选项，其中包含一个或多个服务器名称。
2. 为每个服务器名称创建一个新部分。
3. 为每个服务器名称设置 url 选项。
4. 可选：为每个服务器名称设置 API 令牌。访问 https://galaxy.ansible.com/me/preferences 并单击 显示 API 密钥。

以下示例显示如何配置多个服务器：

```
[galaxy]
server_list = my_org_hub, release_galaxy, test_galaxy, my_galaxy_ng

[galaxy_server.my_org_hub]
url=https://automation.my_org/
username=my_user
password=my_pass

[galaxy_server.release_galaxy]
url=https://galaxy.ansible.com/
token=my_token

[galaxy_server.test_galaxy]
url=https://galaxy-dev.ansible.com/
token=my_test_token

[galaxy_server.my_galaxy_ng]
url=http://my_galaxy_ng:8000/api/automation-hub/
auth_url=http://my_keycloak:8080/auth/realms/myco/protocol/openid-connect/token
client_id=galaxy-ng
token=my_keycloak_access_token
```



Galaxy 服务器列表配置选项

GALAXY_SERVER_LIST 选项是一个按优先级排序的服务器标识符列表。搜索集合时，安装过程将按此顺序搜索，例如，首先搜索 automation_hub，然后搜索 my_org_hub、release_galaxy，最后搜索 test_galaxy，直到找到集合为止。然后，实际的 Galaxy 实例在 [galaxy_server.{{ id }}] 部分下定义，其中 {{ id }} 是列表中定义的服务器标识符。此部分可以定义以下键：

- url：要连接到的 Galaxy 实例的 URL。必需。
- token：用于针对 Galaxy 实例进行身份验证的 API 令牌密钥。与 username 互斥。
- username：用于针对 Galaxy 实例进行基本身份验证的用户名。与 token 互斥。
- password：与 username 结合使用，用于基本身份验证的密码。
- auth_url：如果使用 SSO 身份验证（例如 galaxyNG），则为 Keycloak 服务器“token_endpoint”的 URL。与 username 互斥。需要 token。
- validate_certs：是否验证 Galaxy 服务器的 TLS 证书。默认为 True，除非提供 --ignore-certs 选项或将 GALAXY_IGNORE_CERTS 配置为 True。
- client_id：用于身份验证的 Keycloak 令牌的 client_id。需要 auth_url 和 token。默认的 client_id 是 cloud-services，用于与 Red Hat SSO 协同工作。
- timeout：等待 Galaxy 服务器响应的最大秒数。

除了在 ansible.cfg 文件中定义这些服务器选项外，您还可以将其定义为环境变量。环境变量的格式为 ANSIBLE_GALAXY_SERVER_{{ id }}_{{ key }}，其中 {{ id }} 是服务器标识符的大写形式，{{ key }} 是要定义的键。例如，您可以通过设置 ANSIBLE_GALAXY_SERVER_RELEASE_GALAXY_TOKEN=secret_token 来为 release_galaxy 定义 token。

对于仅使用一个 Galaxy 服务器的操作（例如，publish、info 或 install 命令），ansible-galaxy collection 命令使用 server_list 中的第一个条目，除非您使用 --server 参数传入明确的服务器。



## 删除集合

如果您不再需要某个集合，只需从您的文件系统中删除安装目录即可。路径可能因操作系统而异。

```
rm -rf ~/.ansible/collections/ansible_collections/community/general
rm -rf ./venv/lib/python3.9/site-packages/ansible_collections/community/general
```



## 下载集合

要下载集合及其依赖项以进行脱机安装，请运行 ansible-galaxy collection download。这会将指定的集合及其依赖项下载到指定的文件夹，并创建一个 requirements.yml 文件，该文件可用于在无法访问 Galaxy 服务器的主机上安装这些集合。默认情况下，所有集合都下载到 ./collections 文件夹。

就像 install 命令一样，集合是根据 配置的 Galaxy 服务器配置 获取的。即使通过 URL 或 tarball 的路径指定了要下载的集合，该集合也将从配置的 Galaxy 服务器重新下载。

集合可以指定为一个或多个集合，也可以使用 requirements.yml 文件，就像 ansible-galaxy collection install 一样。

下载单个集合及其依赖项

```
ansible-galaxy collection download my_namespace.my_collection
```



下载特定版本的单个集合

```
ansible-galaxy collection download my_namespace.my_collection:1.0.0
```



要下载多个集合，可以像上面所示那样指定多个集合作为命令行参数，或者使用 使用 requirements 文件安装多个集合 中记录的格式的 requirements 文件。

```
ansible-galaxy collection download -r requirements.yml
```



您还可以下载源集合目录。集合使用必需的 galaxy.yml 文件构建。

```
ansible-galaxy collection download /path/to/collection

ansible-galaxy collection download git+file:///path/to/collection/.git
```



您可以通过提供命名空间的路径来下载单个命名空间下的多个源集合。

```
ns/
├── collection1/
│   ├── galaxy.yml
│   └── plugins/
└── collection2/
    ├── galaxy.yml
    └── plugins/
```



```
ansible-galaxy collection install /path/to/ns
```



所有集合默认下载到 ./collections 文件夹，但您可以使用 -p 或 --download-path 指定另一个路径。

```
ansible-galaxy collection download my_namespace.my_collection -p ~/offline-collections
```



下载集合后，该文件夹包含指定的集合、它们的依赖项和一个 requirements.yml 文件。您可以使用此文件夹与 ansible-galaxy collection install 一起在无法访问 Galaxy 服务器的主机上安装集合。

```
# This must be run from the folder that contains the offline collections and requirements.yml file downloaded
# by the internet-connected host
cd ~/offline-collections
ansible-galaxy collection install -r requirements.yml
```



## 列出集合

要列出已安装的集合，请运行 ansible-galaxy collection list。这将显示在已配置的集合搜索路径中找到的所有已安装的集合。它还将显示包含 galaxy.yml 文件（而不是 MANIFEST.json 文件）的开发中集合。集合所在的路径以及版本信息也将显示。如果版本信息不可用，则版本号将显示为 *。

```
# /home/astark/.ansible/collections/ansible_collections
Collection                 Version
-------------------------- -------
cisco.aci                  0.0.5
cisco.mso                  0.0.4
sandwiches.ham             *
splunk.es                  0.0.5

# /usr/share/ansible/collections/ansible_collections
Collection        Version
----------------- -------
fortinet.fortios  1.0.6
pureport.pureport 0.0.8
sensu.sensu_go    1.3.0
```



使用 `-vvv` 运行以显示更详细的信息。您可能会在此处看到作为已安装集合的依赖项添加的其他集合。仅在您的剧本中使用您已直接安装的集合。

要列出特定集合，请将有效的全限定集合名称 (FQCN) 传递给命令 `ansible-galaxy collection list`。将列出该集合的所有实例。

```
> ansible-galaxy collection list fortinet.fortios

# /home/astark/.ansible/collections/ansible_collections
Collection       Version
---------------- -------
fortinet.fortios 1.0.1

# /usr/share/ansible/collections/ansible_collections
Collection       Version
---------------- -------
fortinet.fortios 1.0.6
```



要搜索集合的其他路径，请使用 `-p` 选项。通过使用 `:` 分隔它们来指定多个搜索路径。命令行上指定的路径列表将添加到已配置的集合搜索路径的开头。

```
> ansible-galaxy collection list -p '/opt/ansible/collections:/etc/ansible/collections'

# /opt/ansible/collections/ansible_collections
Collection      Version
--------------- -------
sandwiches.club 1.7.2

# /etc/ansible/collections/ansible_collections
Collection     Version
-------------- -------
sandwiches.pbj 1.2.0

# /home/astark/.ansible/collections/ansible_collections
Collection                 Version
-------------------------- -------
cisco.aci                  0.0.5
cisco.mso                  0.0.4
fortinet.fortios           1.0.1
sandwiches.ham             *
splunk.es                  0.0.5

# /usr/share/ansible/collections/ansible_collections
Collection        Version
----------------- -------
fortinet.fortios  1.0.6
pureport.pureport 0.0.8
sensu.sensu_go    1.3.0
```



## 验证集合

### 使用 ansible-galaxy 验证集合

安装后，您可以验证已安装集合的内容是否与服务器上的集合内容匹配。此功能要求集合安装在已配置的集合路径之一中，并且集合存在于已配置的 Galaxy 服务器之一上。

```
ansible-galaxy collection verify my_namespace.my_collection
```



如果 `ansible-galaxy collection verify` 命令成功，则输出为空。如果集合已被修改，则更改的文件将列在集合名称下。

```
ansible-galaxy collection verify my_namespace.my_collection
Collection my_namespace.my_collection contains modified content in the following files:
my_namespace.my_collection
    plugins/inventory/my_inventory.py
    plugins/modules/my_module.py
```



您可以使用 `-vvv` 标志显示其他信息，例如已安装集合的版本和路径、用于验证的远程集合的 URL 以及成功的验证输出。

```
ansible-galaxy collection verify my_namespace.my_collection -vvv
...
Verifying 'my_namespace.my_collection:1.0.0'.
Installed collection found at '/path/to/ansible_collections/my_namespace/my_collection/'
Remote collection found at 'https://galaxy.ansible.com/download/my_namespace-my_collection-1.0.0.tar.gz'
Successfully verified that checksums for 'my_namespace.my_collection:1.0.0' match the remote collection
```



如果您安装的是预发行版或非最新版本的集合，则应包含特定版本以进行验证。如果省略版本，则会根据服务器上可用的最新版本验证已安装的集合。

```
ansible-galaxy collection verify my_namespace.my_collection:1.0.0
```



除了 `namespace.collection_name:version` 格式之外，您还可以使用 `requirements.yml` 文件提供要验证的集合。 `requirements.yml` 中列出的依赖项不包含在 `verify` 过程中，应单独验证。

```
ansible-galaxy collection verify -r requirements.yml
```



不支持针对 `tar.gz` 文件进行验证。如果您的 `requirements.yml` 包含 tar 文件的路径或安装的 URL，您可以使用 `--ignore-errors` 标志来确保使用文件中 `namespace.name` 格式的所有集合都被处理。



### 验证已签名集合

如果集合已由 分发服务器 签名，则服务器将提供 ASCII 编码的独立签名，以验证 MANIFEST.json 的真实性，然后使用它来验证集合的内容。此选项并非所有分发服务器都提供。有关支持集合签名的服务器列表，请参阅 分发集合。有关如何在安装时验证已签名集合，请参阅 安装带有签名验证的集合。

要验证已签名的已安装集合

```
ansible-galaxy collection verify my_namespace.my_collection  --keyring ~/.ansible/pubring.kbx
```



使用 `--signature` 选项使用附加签名验证 CLI 上提供的集合名称。此选项可以多次使用以提供多个签名。

```
ansible-galaxy collection verify my_namespace.my_collection --signature https://examplehost.com/detached_signature.asc --signature file:///path/to/local/detached_signature.asc --keyring ~/.ansible/pubring.kbx
```



或者，您可以使用 `requirements.yml` 文件验证集合签名。

```
ansible-galaxy collection verify -r requirements.yml --keyring ~/.ansible/pubring.kbx
```



当从分发服务器安装集合时，服务器提供的用于验证集合真实性的签名将与已安装的集合一起保存。当提供 `--offline` 选项时，此数据用于验证集合的内部一致性，而无需再次查询分发服务器。

```
ansible-galaxy collection verify my_namespace.my_collection --offline --keyring ~/.ansible/pubring.kbx
```



## 在 playbook 中使用集合

安装后，你可以通过其完全限定的集合名称 (FQCN) 引用集合内容

```
- name: Reference a collection content using its FQCN
  hosts: all
  tasks:

    - name: Call a module using FQCN
      my_namespace.my_collection.my_module:
        option1: value
```



这适用于角色或集合中分发的任何类型的插件

```
- name: Reference collections contents using their FQCNs
  hosts: all
  tasks:

    - name: Import a role
      ansible.builtin.import_role:
        name: my_namespace.my_collection.role1

    - name: Call a module
      my_namespace.mycollection.my_module:
        option1: value

    - name: Call a debug task
      ansible.builtin.debug:
        msg: '{{ lookup("my_namespace.my_collection.lookup1", 'param1')| my_namespace.my_collection.filter1 }}'
```



### 使用 collections 关键字简化模块名称

collections 关键字允许你定义一个集合列表，你的角色或 playbook 应在其中搜索不合格的模块和操作名称。因此，你可以使用 collections 关键字，然后简单地在整个角色或 playbook 中通过其简短名称引用模块和操作插件。

### 在角色中使用 collections

在角色中，你可以使用角色 meta/main.yml 中的 collections 关键字控制 Ansible 在角色内部搜索哪些集合。Ansible 将使用角色内部定义的集合列表，即使调用该角色的 playbook 在单独的 collections 关键字条目中定义了不同的集合。集合中定义的角色总是首先隐式搜索它们自己的集合，因此你不需要使用 collections 关键字来访问同一集合中包含的模块、操作或其他角色。

```
# myrole/meta/main.yml
collections:
  - my_namespace.first_collection
  - my_namespace.second_collection
  - other_namespace.other_collection
```





### 在 playbook 中使用 collections

在 playbook 中，你可以控制 Ansible 搜索要执行的模块和操作插件的集合。但是，你在 playbook 中调用的任何角色都会定义它们自己的集合搜索顺序；它们不会继承调用 playbook 的设置。即使角色没有定义它自己的 collections 关键字，也是如此。

```
- name: Run a play using the collections keyword
  hosts: all
  collections:
    - my_namespace.my_collection

  tasks:

    - name: Import a role
      ansible.builtin.import_role:
        name: role1

    - name: Run a module not specifying FQCN
      my_module:
        option1: value

    - name: Run a debug task
      ansible.builtin.debug:
        msg: '{{ lookup("my_namespace.my_collection.lookup1", "param1")| my_namespace.my_collection.filter1 }}'
```



collections 关键字只是为非命名空间插件和角色引用创建了一个有序的“搜索路径”。它不会安装内容或以其他方式更改 Ansible 加载插件或角色时的行为。请注意，非操作或模块插件（例如，查找、过滤器和测试）仍然需要 FQCN。

当使用 collections 关键字时，没有必要将 ansible.builtin 作为搜索列表的一部分添加。当省略时，以下内容默认可用

1. 通过 ansible-base/ansible-core 提供的标准 Ansible 模块和插件
2. 支持旧的第三方插件路径

一般来说，最好使用模块或插件的 FQCN 而不是 collections 关键字。



### 使用集合中的 playbook

2.11 版本中的新功能。

你还可以在你的集合中分发 playbook，并使用与插件相同的语义调用它们

```
ansible-playbook my_namespace.my_collection.playbook1 -i ./myinventory
```



从 playbook 内部

```
- name: Import a playbook
  ansible.builtin.import_playbook: my_namespace.my_collection.playbookX
```



在创建此类 playbook 时，有几个建议，hosts: 应该是通用的，或者至少应该有一个变量输入。

```
- hosts: all  # Use --limit or customized inventory to restrict hosts targeted

- hosts: localhost  # For things you want to restrict to the control node

- hosts: '{{target|default("webservers")}}'  # Assumes inventory provides a 'webservers' group, but can also use ``-e 'target=host1,host2'``
```



这将在 collections: 关键字中有一个隐含的条目 my_namespace.my_collection，就像角色一样。



