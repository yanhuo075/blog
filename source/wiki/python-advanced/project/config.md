---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 7.3 配置文件
order: 2
---

# 配置文件导读

在工程实践中，那些极易发生变化参数都不会硬编码到程序之中，比如数据库配置，包括host，user, password， port, dbname。对这类工程运行所需的参数，会用专门的配置文件保存，当需要修改时，只修改配置文件，而不是修改程序的代码。

修改配置文件，最大程度上降低了因修改代码引入的bug， 是一种安全的项目维护方法，另外，通过提专门的配置文件，可以让后续接受项目维护工程的人更快的了解项目运行所需的参数配置情况，而无需从几万行代码里寻找这些配置。

python项目的配置文件有很多种方式，包括ini文件，config文件，yaml文件，json文件，xml文件，甚至python脚本自身也以作为项目配置文件来使用。



# python读取ini和config配置文件

python项目可以使用ini文件或者config文件做配置文件,并提供了configparser.ConfigParser来读取ini, ini配置文件以.ini结尾，config文件以.config结尾，他们的配置方式相同，本文以ini文件量讲解读取方法。下面是一份ini配置实例文件

```ini
[mysql]
host=127.0.0.1
port=3306
user=root
password=yourpassword
dbname=test


[redis]
host=127.0.0.1
port=6379
password=88888
db=0
```

配置文件由两部分组成sections与items
，sections 用来区分不同的配置块，items 是sections下面的键值。

python3中提供了标准模块configparser，该模块下有一个ConfigParser类，可以用来解析ini文件

```python
from configparser import ConfigParser


cf = ConfigParser()
cf.read('db.ini')

print(cf.sections())        # ['mysql', 'redis']
print(cf.options('mysql'))  # 输出mysql下的所有配置项
print(cf.items('mysql'))    # 输出mysql下的所有键值对

print(cf.get('mysql', 'host'))  # 输出mysql 下配置项host的值
print(cf.getint('mysql', 'port'))  # 输出port
```

程序输出结果

```text
['mysql', 'redis']
['host', 'port', 'user', 'password', 'dbname']
[('host', '127.0.0.1'), ('port', '3306'), ('user', 'root'), ('password', 'yourpassword'), ('dbname', 'test')]
127.0.0.1
3306
```

ConfigParser的get方法可以根据section和option获取配置的值，这个方法返回的是字符串，此外还提供了getint和getfloat方法，分别获取int类型和float类型的配置项，如果你嫌麻烦，可以统一使用get方法。

ConfigParser 也可以用来设置配置项，它提供了add_section方法和set方法，但实际工作中都是手动配置ini文件，因此这部分内容你可以不必关心。



# python读取yaml配置文件

yaml是专门用来写配置文件的标记语言，非常简洁和强大，比ini更加直观更方便。yaml可以作为python的项目配置文件,使用第三方模块yaml可以读取yaml文件,下面是一份简单的配置文件示例

db.yaml

```yaml
mysql:
    host: 127.0.1.1   # ip
    user: test        # 用户名
    port: 3306        # 端口
    db: test
    password: pw

redis:
    host: 127.0.0.1
    port: 3679
    db: 0
    pwd: 3435

db_ip:
    - 192.168.0.1
    - 192.168.0.2
    - 192.168.0.2
```

yaml对大小写敏感，使用缩进来表示层次关系，这一点与python相同，一个缩进可以是2个空格或者4个空格，注意不要使用tab键代替空格。上面这份配置文件等价于python中的字典

```python
{'db_ip': ['192.168.0.1', '192.168.0.2', '192.168.0.2'],
 'mysql': {'db': 'test',
           'host': '127.0.1.1',
           'password': 'pw',
           'port': 3306,
           'user': 'test'},
 'redis': {'db': 0, 'host': '127.0.0.1', 'port': 3679, 'pwd': 3435}}
```

python没有提供专门的标准库来读取yaml文件，需要安装第三方库

```text
pip3 install PyYaml
```

读取示例如下

```python
import yaml

f = open('db.yaml')
data = f.read()
yaml_reader = yaml.load(data)

print(yaml_reader['mysql'])    # 以字典的形式输出mysql下的所有配置
print(yaml_reader['redis']['port'])     # 输出某一项
print(yaml_reader['db_ip'])     # db_ip配置是一个数组
```

程序输出结果

```text
{'host': '127.0.1.1', 'user': 'test', 'port': 3306, 'db': 'test', 'password': 'pw'}
3679
['192.168.0.1', '192.168.0.2', '192.168.0.2']
```



# python读取xml配置文件

xml是一种标记语言，被用来设计存储和传输数据，xml可以用来做python的项目配置文件，可以使用标准模块xml.dom.minidom来解析xml文件。

### db.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<dbconfig>
    <mysql>
        <host>127.0.0.1</host>
        <port>3306</port>
        <dbname>test</dbname>
        <username>root</username>
        <password>4355</password>
    </mysql>
    <redis>
        <host>127.0.0.1</host>
        <port>3306</port>
        <db>0</db>
        <password>44546</password>
    </redis>
</dbconfig>
```

读取的方法很多，我是用python提供的标准模块

```python
import  xml.dom.minidom

dom = xml.dom.minidom.parse('db.xml')
root = dom.documentElement      # 获得根节点
print(root.nodeName)            # 节点名称 dbconfig

# 获取mysql节点
mysql_node = root.getElementsByTagName('mysql')[0]
for node in mysql_node.childNodes:
    if node.nodeType == 1:   # ELEMENT_NODE
        print(node.nodeName, node.firstChild.data)
```

程序输出结果

```text
dbconfig
host 127.0.0.1
port 3306
dbname test
username root
password 4355
```



# python读取json配置文件

json数据更多用于数据的存储和传输，由于大多数语言都支持这种数据格式，因此也可以被用来做python项目的配置文件。使用json数据做配置文件有个弊端，无法写注释。普通的json字符串内容紧凑，失去了key-value数据的可读性，这一点可以通过格式优化来解决，有很多处理json字符串的网站可以将一个json字符串格式化成便于阅读的形式，比如 http://www.kjson.com/

下面是一个json配置文件示例，db.json

```javascript
{
    "mysql": {
        "host": "127.0.0.1",
        "port": 3306,
        "db": "test",
        "username": "test",
        "password": "123456"
    },
    "redis": {
        "host": "127.0.0.1",
        "port": 3679,
        "db": 0,
        "password": "43435"
    }
}
```

使用python读取也是十分方便

```python
import json

f = open('db.json')
data = json.load(f)
f.close()

print(data)
```



# python脚本做配置文件

直接使用python脚本作为配置文件也是一种比较常见的方式，方便之处在于，不用特意去读取配置了，只需要引入配置脚本就可以了。弊端在于这种配置显得很不正式，而且在引入脚本后可以随意修改配置项，不够安全，小一点的项目图省事这样操作是没问题的。

构建这样的配置脚本，一般有两种方法，一种是使用字典，一种是使用类，先来看字典的方式

db.py

```python
mysql_config = {
    'host': '127.0.0.1',
    'port': 3306,
    'db': 'test',
    'username': 'test',
    'password': '123456'
}

redis_config = {
    'host': '127.0.0.1',
    'port': 3679,
    'db': 0,
    'password': '43435'
}
```

使用时，直接引入想要使用的字典

```python
from db import mysql_config
```

第二种方式用类来构建配置项，相比字典，用的时候代码写起来更方便

```python
class MySqlConfig:
    host = '127.0.0.1'
    port = 3306
    db = 'test'
    username = 'test'
    password = 'test'


class RedisConfig:
    host = '127.0.0.1'
    port = 3679
    db = 0
    password = '4355'
```



# 环境变量在python工程中的妙用

工作中，开发，测试，生产都是linux系统，最常见的是centos， 在工程测试和生产部署的时候，时常面临一个不同环境下的参数配置难题。你的项目需要用到mysql数据库，但开发，测试，生产环境的mysql数据库必然是不同的，除了数据库，其他一些配置也会区分这三个环境。一个可行的办法是为这3个环境配置三份不同的配置文件，不同环境下加载不同的配置文件，这就需要手动修改这些配置文件的名称，毕竟程序里加载的配置文件名称是固定的。

那么，能否依据环境的不同加载不同的配置文件呢？

很早以前，我会在程序获取机器的信息，比如机器的操作系统，机器的ip地址，来判断当前机器的环境属于哪一个，这是一个比较low的办法，其实，有一个非常简单实用的方法，就是配置环境变量。

修改 /etc/profile 文件，增加一个新的配置

```text
export PRO_ENV=qa   # 测试环境， dev 开发环境， product 生产环境
```

在不同环境下的机器上，配置不同的环境变量，当程序启动时，获取这个环境变量，以此来判断当前机器所在的环境，根据环境变量加载不同的配置文件

```python
import os

pro_env = os.environ.get('PRO_ENV', 'product')
if pro_env == 'product':
    import product_conf as config
elif pro_env == 'qa':
    import qa_conf as config
else:
    import dev_conf as config
```



# OptionParser解析命令行参数

- OptionParser解析命令行参数

  - 1. OptionParser

    - 1.1 打印help

  - 2. 添加参数

    - 2.1 选项的名称
    - 2.2 action 选项的动作
    - 2.3 dest 指定选项的目标变量名
    - 2.4 type
    - 2.5 default
    - 2.6 help
    - 2.7 metavar

  - 3. 在命令行里如何设置可选项

  - 4. 在程序中使用选项的值和参数的值

## 1. OptionParser

OptionParser 是python内置的功能非常强大的命令行参数解析模块，它使用灵活，可以方便的生成标准的、符合Unix/Posix 规范的命令行说明，下面用一个非常小的示例来向你展示它的功能和使用方法，创建demo.py文件

```python
import sys
from optparse import OptionParser

parser = OptionParser(
                         usage="python %prog [OPTION]",
                         description="酷python，分享最专业的python技术",
                         version="1.1"
                       )

parser.add_option("-n", "--name", action="store", dest="username", help="姓名")
parser.add_option("-a", "--age", action="store", help="年龄", type=int, default=20)
parser.add_option("-d", "--address", action="store", help="住址", metavar="北京")
parser.add_option("-f", "--friends", action="append", help="朋友")
parser.add_option("-t", "--times", action="count", help="次数")
parser.add_option('-o', "--out", action="store_true", help="是否输出")


if __name__ == '__main__':
    (options, args) = parser.parse_args(sys.argv[1:])
    print(options, args)
    print(options.age)
    print(options.username)
```

### 1.1 打印help

OptionParser 可以自动生成程序的help说明，它有助于用户了解程序能够提供的功能，在命令行里行下面的命令

```python
python demo.py --help
```

输出结果

```plain_text
Usage: python demo.py [OPTION]

酷python，分享最专业的python技术

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -n USERNAME, --name=USERNAME
                        姓名
  -a AGE, --age=AGE     年龄
  -d 北京, --address=北京   住址
  -f FRIENDS, --friends=FRIENDS
                        朋友
  -t, --times           次数
  -o, --out             是否输出
```

输出help信息时，程序执行完(options, args) = parser.parse_args(sys.argv[1:]) 就会退出。

这些help信息就包含两部分，一部分是创建OptionParser对象时设置的usage和description，一部分是使用add_option添加的参数信息。

usage 是使用示例说明，OptionParser允许传入prog参数，该参数默认是当前所执行的脚本，就本例而言prog的值是demo.py，prog的值被用于格式化usage。description 是对程序功能的描述。

option部分由add_option所添加的参数构成，此外还有--version 和 --help 这两个原生存在的参数。

## 2. 添加参数

添加命令行选项使用add_option 方法，下面重点讲解该函数的参数。

### 2.1 选项的名称

选项的名称分短名称和长名称，短名称只能是横杠和一个字母的组合，长名称必须是两个横杠和一个单词的组合。

1. 短名称 -n
2. 长名称 --name

这两个名称至少定义一个，都不定义是错误的。

在命令行传入参数时，任选其一使用，但要注意使用的方法有区别：

1. -n cool 短名称与值之间使用空格
2. --name=cool 长名称与值之间使用等号

### 2.2 action 选项的动作

这个参数有以下值可用:

1. store 将选项值存储到目标变量， action默认值是store
2. store_true 如果出现该选项，将目标变量设置为True，示例中选项out的action是store_true，实际使用时无需为out设置value，只要命令行里出现-o 或者--out， 就会将True赋值给out
3. store_false 参考store_true
4. append 将选项值追加到列表中，一旦action设置为append，目标变量的类型就是列表，无需为该选项设置value
5. count 每出现一次该选项，目标变量的极速器就加1 ，无需为该选项设置value

下面的4个实例分别演示action在取不同值时的效果

```python
# store
python demo.py -n cool

{'username': 'cool', 'age': None, 'address': None, 'friends': None, 'times': None, 'out': None} []

# store_true
python demo.py --out

{'username': None, 'age': None, 'address': None, 'friends': None, 'times': None, 'out': True} []

# append
python demo.py -f 小明 -f 小刚

{'username': None, 'age': None, 'address': None, 'friends': ['小明', '小刚'], 'times': None, 'out': None} []

# count
python demo.py -t -t

{'username': None, 'age': None, 'address': None, 'friends': None, 'times': 2, 'out': None} []
```

### 2.3 dest 指定选项的目标变量名

dest设置选项的目标变量名，如果不设置这个参数，那么目标变量名是长名称，没有长名称则使用短名称，在上面的示例中，我为姓名设置了dest参数，那么最终的目标变量名就是username

### 2.4 type

type指定目标变量的类型，以年龄为例，它的类型我设置为int，OptionParser会对传入的值做类型转换，如果你传入字符串会报错

```python
python demo.py -a fff

Usage: python demo.py [OPTION]

demo.py: error: option -a: invalid integer value: 'fff'
```

### 2.5 default

为选项设置默认值，如果命令行里没有设置该选项则选项的值为默认值，本示例中age的默认值是20

```python
python demo.py -n cool

{'username': 'cool', 'age': 20, 'address': None, 'friends': None, 'times': None, 'out': None} []
```

### 2.6 help

选项的解释说明

### 2.7 metavar

指定在帮助信息中显示选项值的名称，在示例中，我为住址设置了metavar， 在help信息里你可以看到如下的内容

```plain_text
-d 北京, --address=北京   住址
```

它的作用是为用户提供单个选项的使用示例。

## 3. 在命令行里如何设置可选项

在命令行里设置可选项时，可选项的顺序不影响解析，不必非得按照add_option添加选项的顺序设置选项值，顺序可以打乱。

除了这些选项，你仍然可以向程序传递其他参数，解析后，选项的信息保存在options里，参数的信息保存在args

```plain_text
python demo.py -n cool --age=20 -f 小刚 -o test1 test2 test3

{'username': 'cool', 'age': 20, 'address': None, 'friends': ['小刚'], 'times': None, 'out': True} ['test1', 'test2', 'test3']
```

在为选项设置完值以后，我增加了3个参数，这3个参数不在我所添加的选项里，被识别为args。

arg的添加同样可以不考虑顺序问题,下面的命令同样可以被正确解析

```python
python demo.py -n cool test1 --age=20 test2 -f 小刚 -o  test3

{'username': 'cool', 'age': 20, 'address': None, 'friends': ['小刚'], 'times': None, 'out': True} ['test1', 'test2', 'test3']
```

但我强烈不建议你这么做，因为看起来太乱了，造成使用时的困惑。

## 4. 在程序中使用选项的值和参数的值

选项的值存储在options对象中，参数的值以列表的形式存储在args中

```python
if __name__ == '__main__':
    (options, args) = parser.parse_args(sys.argv[1:])
    print(options, args)
    print(options.age)
    print(options.username)
```
