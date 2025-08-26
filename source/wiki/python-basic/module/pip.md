---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 8.3 什么是pip
order: 2
---

# 什么是pip，如何安装管理第三方模块

pip 是python标准库的管理工具，使用它可以安装管理第三方库，本篇教程一篇新手引导教程，通过本篇教程，你可以学会掌握以下几点技能

1. 安装第三方库
2. 在 Python Package Index (PyPI) 上查找第三方库
3. 管理requirements.txt文件
4. 卸载第三方库

## 1. 安装第三方库

python是一门自带电池的语言，它本身提供了非常丰富的标准库，同时，python又有着非常活跃的开源社区，在这里，人们实现并贡献了大量第三方库，这些第三方库是对python标准库的补充，那些优秀的第三方库经过一段时间发展后有可能成为标准库随python一同发布。

下面是一个使用标准库的示例

```python
import json

data = [1, 2, 3]
json.dump(data)
```

作为标准库，你可以直接import使用，而对于第三方库，你必须先安装，比如http请求库 requests,安装方法也是极为简单，从python3.4开始，pip已经作为python的一部分发布，因此，你不需要额外安装pip，直接使用就好

```text
pip install requests
```

就是这么简单，在安装时，你可以指定第三方库的版本

```text
pip install requests==2.22.0
```

如果安装成功，一定会出现 Successfully installed 的字样。

第三方库会经常更新，你可以使用下面的命令罗列出已经过期的第三方库

```text
pip list --outdated
```

想要对其中某个库进行升级，可以使用upgrade命令

```text
pip install --upgrade 库名
```

## 2. 在 Python Package Index (PyPI) 上查找第三方库

人们在实现了一个有着特定功能的第三方库后，会发布到https://pypi.org/ 上，pip在安装过程中会到这个网站上寻找合适的库进行下载和安装，我们自己也可以使用命令进行第三方库查找，需要使用search命令

```text
pip search reqeusts
```

使用这个命令可以搜索到非常多的第三方库

```text
requests-hawk (1.0.0)                  - requests-hawk
requests-dump (0.1.3)                  - `requests-dump` provides hook functions for requests.
requests-aws4auth (0.9)                - AWS4 authentication for Requests
requests-auth (4.0.1)                  - Easy Authentication for Requests
Requests-OpenTracing (0.2.0)           - OpenTracing support for Requests
yamlsettings-requests (1.0.0)          - YamlSettings Request Extension
pydantic-requests (0.1.3)              - A pydantic integration with requests.
requests-foauth (0.1.1)                - Requests TransportAdapter for foauth.org!
requests-core (0.0.0)                  - A minimal HTTP Client, for Requests.
pyfortified-requests (0.3.6)           - Extension to Python `requests` functionality.
requests-file (1.4.3)                  - File transport adapter for Requests
```

我只列了其中一部分，正是因为有了这些延伸到每一个领域里的第三方库，才让python变得无所不能。

## 3. 管理requirements.txt文件

开发一个稍大点的项目，都会用到很多第三方库，在部署这个项目时，项目所在的部署环境也需要安装这些第三方库，那么问题来了，该怎样安装这些第三方库呢，总不能一个一个的使用pip命令来安装吧，就算你有耐心一个一个的安装，你也总应该记录下来，这个项目都使用了哪些第三方库吧，以免安装过程中有遗漏。

在python项目管理中，会使用requirements.txt文件保存项目所使用的第三方库，其内容类似下面的示例

```text
flask==1.0.2
cacheout==0.11.0
requests==2.19.1
flask-restful==0.3.7
elasticsearch==6.3.1
redis==2.10.5
PyMySQL==0.9.2
```

有了这份文件，在部署项目时，安装第三方库就变得方便了

```text
pip install -r requirements.txt
```

如果你想检查python第三方库的安装情况，可以使用pip list命令，这个命令会展示出已经安装的第三方库的信息

```text
Package                Version
---------------------- ----------
aniso8601              7.0.0
arrow                  0.13.0
asn1crypto             0.24.0
bcrypt                 3.1.7
bitarray               0.8.3
```

如果你想把自己开发环境的所有第三方库信息都保存到requirements.txt文件中，pip同样提供了命令

```text
pip freeze > requirements.txt
```

## 4. 卸载第三方库

卸载第三方库，使用uninstall命令

```text
pip uninstall requests
```

这时，界面会提示你是否真的卸载，需要你输入y，如果你确实真的想卸载，可以在命令的后面加上-y

```text
pip uninstall requests -y
```

这样就不会再出现询问的指令了

## 5. pip支持的全部命令

```text
pip help
```

使用help 命令，可以查看pip支持的所有命令

```text
Usage:
  pip3 <command> [options]

Commands:
  install                     Install packages.
  download                    Download packages.
  uninstall                   Uninstall packages.
  freeze                      Output installed packages in requirements format.
  list                        List installed packages.
  show                        Show information about installed packages.
  check                       Verify installed packages have compatible dependencies.
  config                      Manage local and global configuration.
  search                      Search PyPI for packages.
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.
  completion                  A helper command used for command completion.
  help                        Show help for commands.

General Options:
  -h, --help                  Show help.
  --isolated                  Run pip in an isolated mode, ignoring environment variables and user configuration.
  -v, --verbose               Give more output. Option is additive, and can be used up to 3 times.
  -V, --version               Show version and exit.
  -q, --quiet                 Give less output. Option is additive, and can be used up to 3 times (corresponding to WARNING, ERROR, and CRITICAL logging levels).
  --log <path>                Path to a verbose appending log.
  --proxy <proxy>             Specify a proxy in the form [user:passwd@]proxy.server:port.
  --retries <retries>         Maximum number of retries each connection should attempt (default 5 times).
  --timeout <sec>             Set the socket timeout (default 15 seconds).
  --exists-action <action>    Default action when a path already exists: (s)witch, (i)gnore, (w)ipe, (b)ackup, (a)bort).
  --trusted-host <hostname>   Mark this host as trusted, even though it does not have valid or any HTTPS.
  --cert <path>               Path to alternate CA bundle.
  --client-cert <path>        Path to SSL client certificate, a single file containing the private key and the certificate in PEM format.
  --cache-dir <dir>           Store the cache data in <dir>.
  --no-cache-dir              Disable the cache.
  --disable-pip-version-check
                              Don't periodically check PyPI to determine whether a new version of pip is available for download. Implied with --no-index.
  --no-color                  Suppress colored output
```
