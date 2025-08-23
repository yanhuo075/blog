---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 2.5 print
order: 4
---

# python内置函数print

print是python的一个内置函数，但我把它放到基础语法这一章来讲解，这是因为，几乎所有编程语言都要提供一个类似的函数来帮我们在标准输出终端输出内容，这是我们与计算机的交互方式。

print可以输出任何你想输出的内容，一个整数，一个字符串，一个列表，只要是python的对象，它都可以输出。通过输出对象的内容，我们可以了解我们正在操作的数据究竟是什么。

在上面这段话中，我提到了对象，数据，在本教程中，如果不做特殊说明，你可以认为他们是同一个东西，然而实际上，对象是比数据更大的概念。

下面是几个简单的使用print的示例

```python
print("hello world")
print([1, 2, 3])
print(100)
```

这几行代码，你可以在pycharm里新建一个脚本来执行，也可以在交互式解释器里执行。
