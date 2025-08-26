---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.8 使用pudb在终端调试python代码
order: 7
---

# 如何使用pudb在终端调试python代码

pudb是一款可以在终端调试python代码的工具，debug是写代码的必备技能，pudb提供了语法高亮，断点，调用栈，命令行等功能，在终端下可以非常方便的对python代码进行调试。
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234532333.jpeg)

- 1是代码区域
- 2是命令行区域，你可以在这里主动查看各个变量的数据，如果对这个命令行功能不满意，还可以通过 ！键进入到python交互式解释器来进行操作
- 3是变量区域，可以观察变量的变化情况
- 4是栈区域，可以查看栈的信息

只要屏幕上的光标不在第2个区域，快捷键都可以使用。使用ctrl + p 可以进入到工具设置界面
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234532042.jpeg)

shift+? 可以进入help界面
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234553579.jpeg)

详细的介绍了工具的快捷键，如何运行下一步，如果跳出当前函数等等

功能强大就不说了，关键安装使用还特别方便

```text
pip install pudb
```

使用示例代码

```python
from pudb import set_trace
set_trace()
lst = [1, 2, 3, 5, 7, 8, 10]

for item in lst:
    if item % 2 == 0:
        print(item)
```

重点关注前两行代码， set_trace()设置断点

启动工具调试代码的命令如下

```text
pudb3 test.py
```

有了这个工具，再也不担心在linux下调试代码了
