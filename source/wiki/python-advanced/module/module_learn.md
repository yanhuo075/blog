---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 4.1 深入学习模块
order: 0
---

# 模块概念深入学习

在python入门阶段，只需要初步了解模块的概念，能够使用import 或者from ... import ... 语法将自己需要的模块导入即可。进入到进阶阶段，需要你进一步深入思考模块是如何被导入的，导入模块时的原理和过程是什么样的？如何才能实现模块的动态加载，如何实现模块的惰性导入，本章的内容将为你揭晓这些答案。



# python模块import原理

## 1. import导入过程

python执行import语句时，只有两个步骤，第一步是搜索模块，第二步是将搜索结果绑定到局部命名空间。

搜索时，分为两步：

1. 搜索sys.modules
2. 搜索sys.meta_path

导入一个模块时，会将这个导入的模块以及这个模块里调用的其他模块信息以字典的形式保存到sys.modules中，如果再次导入词模块，则优先从sys.modules查找模块，你可以在脚本里执行print(sys.modules)查看已经加载的模块,我们甚至可以直接修改sys.modules里的内容

```python
import os
import sys

sys.modules['fos'] = os
import fos
print(fos.getpid())
```

执行import fos时，会先到sys.modules里查找是否有该模块，'fos'做key,找到的value是os模块，因此可以调用getpid方法。

如果在sys.modules模块中找不到目标模块，则从sys.meta_path中继续寻找。sys.meta_path是一个list，里面的对象是importer对象，importer对象是指实现了finders 和 loaders 接口的对象，输出sys.meta_path里的内容可以查看有什么

```text
<class '_frozen_importlib.BuiltinImporter'>
<class '_frozen_importlib.FrozenImporter'>
<class '_frozen_importlib_external.PathFinder'>
```

这三个importer对象分别查找及导入build-in模块，frozen模块（即已编译为Unix可执行文件的模块），import path中的模块，如果都找不到，就会报ModuleNotFoundError的错误。

## 2. sys.path

导入模块时，首先会去sys.modules里查看，如果查不到会使用sys.meta_path里的importer继续查找，这些importer首先会查找内置模块，然后查找frozen模块，最后会根据sys.path里的路径进行查找。在脚本里执行print(sys.path)，在我的电脑上输出结果为

```text
/Users/kwsy/kwsy/coolpython
/Library/Frameworks/Python.framework/Versions/3.6/lib/python36.zip
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/proxypool-2.0.0-py3.6.egg
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/bs4-0.0.1-py3.6.egg
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/Flask_Mail-0.9.1-py3.6.egg
/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/blinker-1.4-py3.6.egg
```

sys.path里路径的顺序决定了搜索的顺序，这里的路径分为3类

1. sys.path[0] 是当前路径，也是最先被搜索的
2. 第二类是安装python时内置进去的，比如 `/Library/Frameworks/Python.framework/Versions/3.6/lib/python36.zip/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages`
3. 最后一类就是安装的第三方模块

如果模块存在于这些路径中，那么不管它身在何处，都可以使用import直接导入，下面是一个python脚本的地址

```text
/Users/kwsy/PycharmProjects/pythonclass/mytest/import_demo/conf/offline.py
```

脚本内容为

```text
host='192.168.0.2'
```

在其他的项目里，只要将offline.py的地址加入到sys.path中，就可以在脚本里直接引入offline

```python
import sys
import importlib

sys.path.append('/Users/kwsy/PycharmProjects/pythonclass/mytest/import_demo/conf')
import offline
print(offline.host)

module = __import__('offline')
print(module.host)

module = importlib.import_module('offline')
print(module.host)
```

实践中你的编辑器甚至会在import offline这一行显示红色的波浪线，那是在警告你找不到模块，这个警告是编辑器发出的，因为offline.py的目录是在程序执行期间加入到sys.path中的，编辑器在你编写代码阶段还检查不到sys.path中有这个目录，因此是编辑器找到不到这个模块，程序执行时，python解释器却可以找得到，这也是一种动态加载模块的技术。

## 3. import hooks

我们可以通过一些技术手段来扩展import的行为，为了让你有一个直观的理解，推荐一个第三方库pypi,此模块实现了一个神奇的功能，你可以在代码里导入根本不存在的模块，遗憾的是还不能使用pip来安装这个模块，你可以直接将pypi.py文件放在site-packages文件下或者直接放在项目里，该模块的git地址是 https://github.com/miedzinski/import-pypi

现在，来做一个实现

```python
import pypi
import requests

print(requests)
```

我在执行这段代码时，已经将requests模块卸载，但是执行这段代码时却不会报错误，在等待一段时间后，程序正常执行print语句。

pypi.py的源码并不复杂

```python
import importlib.abc
import importlib.machinery
import subprocess
import sys


def install(pkgname, version=None):
    cmd = [sys.executable, '-m', 'pip', 'install']
    if version:
        cmd.append('{}=={}'.format(pkgname, version))
    else:
        cmd.append(pkgname)
    subprocess.check_call(
        cmd,
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
    )


class PipFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        try:
            install(fullname)
        except subprocess.CalledProcessError:
            return None
        else:
            return importlib.machinery.PathFinder().find_spec(
                fullname,
                path,
                target,
            )


sys.meta_path.append(PipFinder())
```

sys.meta_path里存储了importer对象，如果在sys.modules里找不到目标模块，就会利用这里的对象继续寻找，在pypi中，作者实现了一个名为PipFinder的importer并将其放入到sys.meta_path，find_spec专门用来寻找指定的模块，进入函数后，首先调用install函数对模块进行安装，假设模块已经安装那么什么都不会发生，假设模块之前没有被安装则直接进行安装。安装结束后，调用importlib模块加载指定模块。

## 4. 参考资料

1. https://sikx.io/2017/10/09/Python_import/
2. https://mozillazg.com/2016/04/apm-python-agent-principle.html
3. https://github.com/miedzinski/import-pypi/blob/master/pypi.py



# 动态加载

## 1. __import__

当你使用import关键字导入模块时，底层实现默认调用的是__import__，直接使用该函数的情况很少见，一般用于动态加载。假设有这样一个场景，项目里有两份配置文件，一份是线下开发环境配置

**offline.py**

```python
host='129.168.0.1'
```

另一份是线上测试环境
**online.py**

```python
host='192.168.0.2'
```

不同的环境需要加载不同的配置文件，这种情况就可以使用__import__来动态加载

```python
import platform

if platform.uname().system == 'Darwin':     # mac电脑
    config = __import__('offline')
else:
    config = __import__('online')

print(config.host)
```

在mac电脑上执行这段代码时将加载offline模块，其他系统上会加载online模块。

## 2. fromlist

__import__有很多参数，其中name是必须的，其他参数中fromlist是一个比较重要的参数。在第一节的示例中，两份配置文件与引入模块的文件在同一个文件夹下，现在，我更改项目结构

```text
├── conf
│   ├── __init__.py
│   ├── offline.py
│   └── online.py
└── demo.py
```

在demo.py脚本中，继续使用上一小节中的方法则需要修改 __import__函数中的name

```python
import platform

if platform.uname().system == 'Darwin':     # mac电脑
    config = __import__('conf.offline')
else:
    config = __import__('conf.online')

print(config)
```

输出的结果是

```text
<module 'conf' from '/Users/kwsy/PycharmProjects/pythonclass/mytest/import_demo/conf/__init__.py'>
```

我们的本意是导入不同的模块，或者offline或者online，但实际导入的是conf模块，这与我们的预期不符。这种情况下，需要使用fromlist参数

```python
import platform

if platform.uname().system == 'Darwin':     # mac电脑
    config = __import__('conf.offline', fromlist=('offline'))
else:
    config = __import__('conf.online', fromlist=('online'))

print(config)
```

程序输出结果

```text
<module 'conf.offline' from '/Users/kwsy/PycharmProjects/pythonclass/mytest/import_demo/conf/offline.py'>
```

经过上面的实验，可以得出一个简单的结论，在使用__import__函数时，name的地址如果包含了包的名字，一定要使用fromlist参数来指明要导入包里的哪个模块，否则就会将整个包导入。

## 3. importlib.import_module

importlib模块的import_module方法相比于__import__更加友好，使用起来更加方便。下图是项目的结构

```text
├── conf
│   ├── __init__.py
│   ├── offline.py
│   └── online.py
└── demo.py
```

在demo.py文件中根据系统来加载不同的模块，使用import_module方法的示例代码如下

```python
import platform
import importlib

if platform.uname().system == 'Darwin3':     # mac电脑
    config = importlib.import_module('conf.offline')    # 绝对导入
else:
    config = importlib.import_module('.online', package='conf')     # 相对导入

print(config.host)
```

使用相对导入时，务必在name前面加一个点



# 不建议使用 from ... import *

你可以使用下面的导入方式

```text
from ... import *
```

但我不建议你这么做，使用这种方式会将目标模块里的所有内容都导入，除了以下划线开始的变量。使用这种方式导入模块，会导入很多你原本不需要的东西，比如函数，类，造成一些不必要的麻烦。

如果你编写的模块可能会被其他人使用，你可以通过在模块里定义列表__all__ 来防止对方导入不需要的模块或者你不希望被其他人导入的模块。

脚本my_utils.py

```python
def check_phone(phone):
    print('check_phone')

def check_id_no(id_no):
    print('check_phone')

__all__ = []
```

定义__all__为空列表，如果其他模块在导入my_utils模块时使用from ... import *的方式，那么任何内容都无法正常导入，逼迫对方导入指定的函数

```text
from my_utils import check_phone
```

即便__all__是空列表，只要导入模块时指定要导入的函数就可以通过编译。

如果你想稍稍放开限制，允许其他人是用from ... import *的方式，但是限制可以导入的内容，那么只需要在__all__填写你允许被* 导入的内容即可，比如你允许导入check_phone 函数，那么可以这样来定义

```python
__all__ = ['check_phone']
```

这样，即便使用了from ... import *的方式，也只能导入check_phone这一个函数

```python
from my_utils import *

check_phone('')
```

使用其他函数都是不被允许的。



# python的第三方库库安装在哪里了

python的第三方库会被安装在site-packages文件夹中，这是很多人都知道的事情，但这个文件夹又在哪里呢？当你的系统里安装了好几python的版本时，往往会搞不清楚自己当前所用的python的目录是哪里，想要进入到site-packages进行一些操作，却发现自己根本不知道。

我也曾被这样的事情困扰过，还为此百度搜索，最后发现，找到所用的python安装目录是一个非常简单的事情，这里给大家介绍两种方法

## 方法1，通过sys.path

在交互式解释器里，import sys，然后输出sys.path的内容

```python
Python 3.6.8 (tags/v3.6.8:3c6b436a57, Dec 24 2018, 00:16:47) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', 'D:\\python\\python36.zip', 'D:\\python\\DLLs', 'D:\\python\\lib', 'D:\\python', 'D:\\python\\lib\\site-packages', 'D:\\python\\lib\\site-packages\\utility-0.1.9-py3.6.egg', 'D:\\python\\lib\\site-packages\\requests-2.21.0-py3.6.egg', 'D:\\python\\lib\\site-packages\\kazoo-2.7.0-py3.6.egg', 'D:\\python\\lib\\site-packages\\urllib3-1.24.3-py3.6.egg', 'D:\\python\\lib\\site-packages\\idna-2.8-py3.6.egg', 'D:\\python\\lib\\site-packages\\chardet-3.0.4-py3.6.egg', 'D:\\python\\lib\\site-packages\\certifi-2020.4.5.1-py3.6.egg', 'D:\\python\\lib\\site-packages\\win32', 'D:\\python\\lib\\site-packages\\win32\\lib', 'D:\\python\\lib\\site-packages\\Pythonwin']
```

sys.path包含的是 python的搜索模块的路径集 ，当我们在脚本里执行import语句时，会在sys.path的路径中依次进行查找。

## 方法2，使用sysconfig

sysconfig模块可以获得python的许多配置参数，在shell里执行

```text
python -m sysconfig
```

即可获得当前所用python的详细配置参数

```text
Platform: "win-amd64"
Python version: "3.6"
Current installation scheme: "nt"

Paths:
        data = "D:\python"
        include = "D:\python\Include"
        platinclude = "D:\python\Include"
        platlib = "D:\python\Lib\site-packages"
        platstdlib = "D:\python\Lib"
        purelib = "D:\python\Lib\site-packages"
        scripts = "D:\python\Scripts"
        stdlib = "D:\python\Lib"

Variables:
        BINDIR = "D:\python"
        BINLIBDEST = "D:\python\Lib"
        EXE = ".exe"
        EXT_SUFFIX = ".pyd"
        INCLUDEPY = "D:\python\Include"
        LIBDEST = "D:\python\Lib"
        SO = ".pyd"
        VERSION = "36"
        abiflags = ""
        base = "D:\python"
        exec_prefix = "D:\python"
        installed_base = "D:\python"
        installed_platbase = "D:\python"
        platbase = "D:\python"
        prefix = "D:\python"
        projectbase = "D:\python"
        py_version = "3.6.8"
        py_version_nodot = "36"
        py_version_short = "3.6"
        srcdir = "D:\python"
        userbase = "C:\Users\zhangdongsheng\AppData\Roaming\Python"
```

输出的信息可能会很多，如果不方便查看，你还可以编写代码输出指定的信息

```python
import sysconfig

print(sysconfig.get_path("platlib"))  # D:\python\Lib\site-packages
```



# 获取python安装的第三方库列表

pkg_resources模块提供了find_distributions函数，可以返回python环境里安装的第三方模块，使用该函数时，需要指定python模块的安装地址，你可以通过sys.path来获取这些地址。下面的示例，演示了如果获取site-packages中所安装的第三方模块

```python
from pkg_resources import find_distributions
path = "D:\\python\\lib\\site-packages"
res = find_distributions(path)
for item in res:
    print(item.project_name, item._version)
```

我们在使用pip安装一个库时，pip会先获取系统里已经安装的第三方库的信息，然后判断你想要安装的库是否已经存在，如果不存在，则会进行安装。

pkg_resources模块的require函数可以返回一个包的依赖包和当前所安装的包的具体信息

```python
from pkg_resources import require

lst = require('requests')
for item in lst:
    print(item)
```

程序输出结果

```text
requests 2.21.0
urllib3 1.24.3
idna 2.8
chardet 3.0.4
certifi 2020.4.5.1
```



# python修改sys.path的三种方法

sys.path是一个列表，存放的是python搜索模块时可以搜索的路径，启动python脚本时，会将执行当前命令所在的目录添加到这个列表中，而且是在列表的最前面，正是因为这个操作，你才能在自己的项目里引用自己编写的模块，当模块名称与第三方模块或系统模块冲突时，优先引用项目里的模块。通常，sys.python里的内容如下所示：

```python
>>> import sys
>>> sys.path
['', '/root/.pyenv/versions/3.6.5/lib/python36.zip', '/root/.pyenv/versions/3.6.5/lib/python3.6', '/root/.pyenv/versions/3.6.5/lib/python3.6/lib-dynload', '/root/.pyenv/versions/3.6.5/lib/python3.6/site-packages']
```

这里，你重点关注site-packages， 我们安装的第三方库和模块都放在了这里。将sys.path设计为一个可变的列表，而不是元组，就是考虑到了开发人员有很强的动机和需要去修改模块的搜索路径和顺序。

### 方法1，直接修改sys.path列表

设想，如果你的系统允许用户提交自定义的python脚本，那么你可以为此专门创建一个目录用于存放这些脚本，并将这个目录加入到sys.path中，这样，在你的系统里，你可以像引用其他模块一样去引用用户上传的python脚本来执行他们，这就是本文所提到的修改sys.path的三种方法中的一个，你只需要使用sys.path.append方法将目录添加即可。

### 方法2， 创建.pth文件

另一种修改sys.path的方法时在site-packages目录新建一个.pth文件，帮在文件中加入搜索模块的路径

```text
/root/test
```

重新启动一个python交互式解释器，输出sys.path，你可以看到/root/test目录也在其中。

### 方法3，设置PYTHONPATH环境变量

第三种修改方法，通过PYTHONPATH环境变量，我使用export 命令设置该环境变量

```text
[root@sheng studyflask]# export PYTHONPATH=/root/studyflask
[root@sheng studyflask]# echo $PYTHONPATH
/root/studyflask
```

这种设置方法仅仅是为了验证是否凑效，退出终端后，环境变量就会失效，如果你想永久生效，可以在/etc/profile，或者.bashrc中进行设置，设置完PYTHONPATH后，启动一个新的python交互式解释器，输出sys.path

```python
>>> import sys
>>> sys.path
['', '/root/studyflask', '/root/.pyenv/versions/3.6.5/lib/python36.zip', '/root/.pyenv/versions/3.6.5/lib/python3.6', '/root/.pyenv/versions/3.6.5/lib/python3.6/lib-dynload', '/root/.pyenv/versions/3.6.5/lib/python3.6/site-packages']
```

### 4， 三种方法比较

从灵活性上比较，方法1最为灵活，搜索目录的位置可以通过调用列表的insert方法自由控制，而方法2所添加的搜索目录会加在sys.path的末尾，方法3会加在列表的开头。

从有效范围上比较，方法的生效范围最小，你在脚本里对sys.path进行修改，那么它只对所修改脚本的执行时的进程有效果，其他脚本在执行时不会受到影响，方法2的有效范围就要大一些，假如你有多个python版本，但只在某一个python的site-packages里新建了.pth文件，那么只有当你使用这个python作为启动python脚本的应用程序时，.pth才会生效。生效范围最大的是PYTHONPATH，一旦设置生效，不论你用系统上的哪个python来执行脚本，它都生效，因为python在执行脚本时会自动加载这个环境变量，只要环境变量可以访问，就必然生效。



# python模块绝对引用和相对引用

python项目中的模块引用问题，是一个比较复杂的问题，无非是绝对引用，相对引用，看起来似乎很简单，但在实践中，总是会出现一些“莫名其妙”的错误，但解决起来倒也方便，import语句的写法多试验几次也就可以搞定了，关于这方面，很少有文章全面深入的讨论，我也是在这块吃过几次亏后，痛定思痛，决定认真研究一下。本文所使用示例在python3.7环境下顺利通过，如果文中观点有错误之处，请各位看官不吝赐教。

### 入口脚本

首先明确一点，一个项目的入口脚本，或者说启动脚本，必须放在项目的根目录下，启动脚本所在的目录，将被加入到sys.path里，而我们在脚本里使用import引入模块时，会根据sys.path里的目录逐个进行查找，这一点很关键，后面会用到。

```text
pypackage/
├── __init__.py
├── log.py
├── run.py
├── utils
│   ├── fileutil.py
│   └── __init__.py
└── view
    ├── __init__.py
    ├── one.py
    ├── two.py
    └── view2
        ├── four.py
        ├── __ini__.py
        └── three.py
```

run.py 作为项目的启动脚本，放在了根目录下，它可以引用项目里的任意模块。

### 绝对引用

从层级上看，log.py也在根目录下，four.py 在view/view2目录下， 那么 four.py脚本里可以引入log模块么？答案是肯定的，因为启动脚本所在的目录，将被加入到sys.path里，因此在four.py 里可就可以这样写

```python
import log
```

受此启发，fileutil 也可以引入four模块

```python
from view.view2 import four
```

这里的关键就在于启动脚本所在的目录被加入到sys.path里

### 相对引用

如果我想在one.py 里引用two模块，除了绝对引用外，还可以使用相对引用

```python
from view import two    # 绝对引用
from . import two       # 相对引用
```

如果我想在one.py 里引用fileutil 模块，讲道理的话，下面两种方法都可行

```python
from utils import fileutil  # 绝对引用
from ..utils import fileutil  # 相对引用  实测不可行
```

一个. 表达式当前目录， 两个 .. 表示上一级目录，那么 from ..utils import fileutil 本应该也可行，但实测却发现无法引用，所以，遇到这种莫名其妙的问题，我建议你使用绝对引用，避免使用相对引用
