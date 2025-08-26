---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.2 本章简介
order: 0
---

# python基础教程-循环控制导读

与其他编程语言一样，python提供了for循环和while循环这两种循环方式。循环对于任何一门编程语言都是不可缺少的程序控制方式，如果没有循环，计算机将变得和人一样慢。现在，我要求你用print函数输出0到100之间的所有整数，如果你真的去做这件事情而又不使用循环，那么你只能写出下面的代码

```python
print(0)
print(1)
print(2)
print(3)
...
print(99)
print(100)
```

看到没，这就是没有循环的下场，编程将变成一场灾难。如果使用循环呢，只需要两行代码就可以搞定

```python
for i in range(0, 101):
    print(i)
```
