---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 8.2 模块
order: 1
---

# 模块

## 1. 什么是模块

模块是一个包含所有你定义的函数和变量的文件，其后缀名是.py
你可以简单的理解成，一个.py文件就是一个模块

## 2.模块存在的意义

那些有着相似作用的函数，比如和时间相关的，和文件路径相关的，和系统环境相关的，他们所能处理的问题有着紧密的联系，把他们设计成一个模块，在使用的时候，便于查找，也便于维护。

这就好比，你家里的抽屉，里面存放的物品，大多数情况下都是按类别和用途分层存储的，这样，找物品的时候就比较的方便。

我们维护代码时，也采用这样的思路。以《文件读写》教程里的4.2 练习题为例，在这个练习题里我一共写了三个函数，他们分别是：

1. get_file_lst
2. get_program_line_count
3. get_py_program_count

这其中函数get_file_lst非常具有通用性，它既可以在计算文件夹下所有python脚本代码行数这个事情上使用，也可以在其他场景下使用，函数的功能越是简单，越是有通用性。那么这样的函数，通常我们会把它单独的存放到一个模块里，这样，其他的模块就可以在需要的时候引用它。

## 3. import

为了向你演示如何引入其他模块为己所用，我将《文件读写》教程里的4.2 练习题的文件结构进行修改

utils.py文件的内容如下

```python
import os


def get_file_lst(py_path, suffix):
    """
    遍历指定文件目录下的所有指定类型文件
    :param py_path:
    :return:
    """
    file_lst = []
    for root, dirs, files in os.walk(py_path):
        for file in files:
            file_name = os.path.join(root, file)
            if file_name.endswith(suffix):
                file_lst.append(file_name)

    return file_lst
```

如何在program_count.py 文件中使用utils.py文件中的函数呢？使用import，直接导入utils模块，使用get_file_lst函数时，则需要用utils.get_file_lst这种方法来调用函数。

```text
import utils


def get_program_line_count(file_name):
    """
    返回单个py脚本的代码行数
    :param filename:
    :return:
    """
    # 存储代码行数
    count = 0
    with open(file_name, 'r', encoding='utf-8')as f:
        for line in f:
            line = line.strip()
            if not line:
                continue

            if line.startswith("#"):
                continue
            count += 1
    return count


def get_py_program_count(py_path):
    # 获取指定目录下所有的python脚本
    file_lst = utils.get_file_lst(py_path, '.py')
    count = 0
    # 遍历列表,获取单个python脚本的代码行数
    for file_name in file_lst:
        count += get_program_line_count(file_name)  # 累加代码函数

    return count   # 返回代码总函数


if __name__ == '__main__':
    py_path = '/Users/kwsy/PycharmProjects/pythonclass'
    print(get_py_program_count(py_path))
```

## 4. from ... import ...

import 会将整个模块的内容全部导入，但有时，我们不需要那么多，只需要其中一个两个就可以了
于是，我们也可以这样写

```python
from utils import get_file_lst


def get_py_program_count(py_path):
    file_lst = get_file_lst(py_path, '.py')
    count = 0
    for file_name in file_lst:
        count += get_program_line_count(file_name)

    return count
```

如果你想全部导入，也可以写成from ... import * 的形式，*表示全部导入。这种写法和import的区别在于，import一个模块A后，调用A里的函数是必须写成A.func的形式，而from ... import ... 的方法，就可以不用模块去调用方法了，可以直接调用函数

## 5. globals()

globals函数以字典的形式返回当前位置的全部全局变量
在一个模块中执行

```python
print(globals())
```

你会得到一个字典，包含了当前模块的全部全局变量，这里面有你定义的函数，全局变量，还有一些是系统自带的全局变量，例如__name__, __file__, __doc__

- __name__ 模块名称
- __file__ 当前脚本的绝对路径
- __doc__ 当前模块的注释

```python
'''
当前模块的注释说明
'''
def test():
    print("调用函数test")


g_var = globals()
print(g_var)

print(__file__)
print(__name__)
print(__doc__)

g_var['test']()   # 可以调用test函数
```

函数test 也是对象，而且是全局变量，g_var是字典，字符串 'test' 作为key，其value就是函数test
