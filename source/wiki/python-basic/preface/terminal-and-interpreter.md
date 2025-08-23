---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 1.4 终端和交互解释器
order: 3
---

# 什么是终端

终端是mac电脑上的一个应用程序，可以在终端里执行命令，在windows电脑上，也有一个类似的程序，你大概听说过，叫cmd,大部分学习者使用的是windows，因此我们先说cmd

## 打开cmd

win+r 打开运行，在输入框内输入cmd点击回车即可进入cmd命令窗口
![cmd](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145747022.jpeg)
进入以后，大概是类似这样的画面，每个人电脑里出现的结果会不一样，如果你在这里输入python命令并回车，就会进入python交互式解释器。
![python交互式解释器](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145747065.jpeg)

想要通过python命令运行python脚本，也需要在cmd命令窗口里进行，我会专门写一篇运行python脚本的文章

## 打开mac终端

在应用程序里找到 **实用工具** ，在实用工具中可以找到终端
![mac终端](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145747085.jpeg)

打开后，为了以后方便，你可以选择将其保留在Dock上
![dock保留](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145747034.jpeg)

同理，在这里输入python，也会进入python交互式解释器,mac默认已经安装好了python2.7，所以， 你输入python，进入的python2.7 的交互式解释器，我在mac上又安装了python3.6, 让python3指向了python3.6
![python交互式解释器](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145812153.jpeg)



# 什么是python交互式解释器

## 1. python交互式解释器

python是一门解释型语言，我们编写的代码，必须通过解释器来运行。python的解释器有很多种，分别是基于不同语言开发的，常见的五种python交互式解释器有下面5个

1. CPython
2. IPython
3. PyPy
4. Jython
5. IronPython

我们平时用的是CPython，底层是用c语言实现的。

## 2. 如何进入python交互式解释器

在mac电脑的终端里，输入python命令并回车，即可进入python交互式解释器
![mac电脑进入python交互式解释器](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145718451.jpeg)

在windows电脑上，你需要先启动cmd，然后在cmd命令窗口里输入python命令，然后回车
![通过cmd进入python交互式解释器](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145718496.jpeg)

## 3. 交互式解释器有什么作用

交互式，体现在你可以即时的获取代码执行的反馈，如果是在pycharm这样的编辑器里，你写完一段代码后，先要知道代码的执行效果必须执行代码，稍作修改后，还得再执行一次代码才能知道效果。

而在交互式解释器里，就如同一个始终在运行着程序，你随时可以修改，随时可以获得反馈

```python
Python 3.6.6 (v3.6.6:4cf1f54eb7, Jun 26 2018, 19:50:54)
[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> lst = [3, 4, 1, 5, 6]
>>> lst
[3, 4, 1, 5, 6]
>>> lst.append(0)
>>> lst
[3, 4, 1, 5, 6, 0]
>>> lst[3] = 100
>>> lst
[3, 4, 1, 100, 6, 0]
>>>
```

在刚刚接触python时，利用python交互式解释器可以更快的练习所学习的知识，通过观察输出的结果，加深对所学习语法的认识和理解，在交互式解释器里，你想输出一个变量的内容，不需要使用print语句，解释器会默认输出变量的内容。
