---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 7.4 绝对路径与相对路径
order: 3
---

# 绝对路径与相对路径

操作文件时，就会遇到绝对路径与相对路径这个问题，很多初学者都在这里栽跟头，傻傻搞不清楚，阅读完本文，你再也不会糊涂

## 1. 绝对路径

这是一个比较好理解的概念，绝对路径，换一种说法就是全路径，在windows下就是从盘符开始的，比如

```text
C:\windows\system32\cmd.exe
```

C盘再往前已经没有文件夹了，这个路径是从根上出发的，那它就是绝对路径

## 2. 相对路径

相对路径，就不需要从根目录出发了，而是相对于某个文件目录的路径，那么问题来了，相对路径到底相对于谁呢？

为了弄清楚这个问题，我建一个实验文件夹，目录结构如下

```text
test_path
├── __init__.py
└── mypath
    ├── __init__.py
    ├── data
    └── test.py
```

一共有两层文件夹，test_path在外层，mypath是内层，data文件内容为

```text
123
python
```

test.py文件内容为

```text
with open("./data") as f:
    print(f.read())
```

这里就用到了相对路径---"./data" 这里的./ 就是当前目录，那么当前目录又是什么呢？ 很多教程告诉你，当前目录就是test.py所在的目录，这个答案是错误的，当前目录是启程程序的命令执行时所在的目录。

现在，我进入到 mypath文件夹中，执行test.py脚本
![绝对路径与相对路径](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823232945729.jpeg)
你可以看到，程序可以正常执行，注意，执行脚本的命令发生在mypath目录中

现在，我进入到test_path 文件夹中执行test.py脚本
![绝对路径与相对路径](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823232945744.jpeg)
程序发生了错误，因为当前执行脚本的命令是在test_path中发生的，因此，所谓的当前目录就是test_path，"./data" 则是在test_path中寻找data文件，自然找不到。

继续验证，还是在test_path目录中执行程序，但是对程序代码稍作修改

```python
with open("./mypath/data") as f:
    print(f.read())
```

再次执行脚本
![绝对路径与相对路径](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823233007316.jpeg)
一切正常

捋一捋思路，相对路径，是相对于程序执行命令所在的目录，./ 表示的不是脚本所在的目录，而是程序执行命令所在的目录，也就是所谓的当前目录。

## 3. 回退一级目录

现在，你已经知道了./ 表示哪个目录，此外，你还会遇到路径中包含 ../ 的情况，这种方式表示回退一级目录

还是以刚才的实验代码为例，将data文件剪切移动到test_path目录下，将test.py脚本修改

```text
with open("../data") as f:
    print(f.read())
```

现在进入到mypath目录下执行脚本
![绝对路径与相对路径](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823232945750.jpeg)

- 执行程序的命令发生在mypath目录中，mypath就是当前目录
- ../ 是回退一级目录，这样就来到了test_path目录中，而这里刚好有剪切移动过来的data文件

## 4. os.getcwd()

程序的工作目录，也就是前面所提到的执行程序的命令所发生的的目录，可以通过os.getcwd()方法获取到。

如果你的程序中自带了一些数据，需要在程序运行期间读取，那么千万不要用相对路径，因为你可能并不知道你的程序是在哪里被启动的，这就意味着你并不清楚程序的工作目录，这个时候，你要先获取脚本的绝对路径，然后根据脚本与数据时间的目录关系进行查找，还是以前面的实验项目为例

```text
test_path
├── __init__.py
└── mypath
    ├── __init__.py
    ├── data
    └── test.py
```

修改test.py脚本

```python
import os

file_path = os.path.abspath(__file__)  # 先通过abspath方法获取当前脚本的绝对路径
print(file_path)
file_path = os.path.dirname(file_path)  # 获取当前路径
print(file_path)

with open(os.path.join(file_path, 'data')) as f:
    print(f.read())
```

程序通过os.path模块，获取到了data文件的绝对地址，这时候，不管在哪个目录下启动程序，程序都可以正常执行
![绝对路径与相对路径](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823233013003.jpeg)
