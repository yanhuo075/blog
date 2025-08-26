---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.5 神奇的点
order: 4
---

# 轻松学会python面向对象第4篇---神奇的点

点动成线，线动成面，面动成立体，宇宙起源于一个奇点，看来，点是一个很神奇的东西。在编程语言里，点无处不在。

## 1. 追踪寻找

```python
import os

print(os.path.exists('/data'))
```

上面的代码里出现了两个点，那么这两个点的作用是什么呢？代码的目的是要判断/data目录是否存在，要用到exists函数，而这个exists函数写在了path模块里，path并没有直接对外暴露，而是存在与os模块里。因此，想要使用exists函数，就要从os模块开始，用“点”来一步步追踪寻找。你也可以换一种方式

```python
from os.path import exists

print(exists('/data'))
```

这里的点，表达的是层级关系，path模块是os模块的子模块，通过点，可以从高层的模块，逐步的向下寻找自己想要的模块或方法。

## 2. 获取之意

```python
class Dog():
    def __init__(self, _name, _age):
        self.name = _name
        self.age = _age

    def eat_moon(self):
        print("天狗吃月")


dog = Dog('二哈', 3)
print(dog.name)     # 二哈
print(dog.age)      # 3
```

dog.name 的值是二哈，这里的点也可以解读为追踪寻找之意，但我更喜欢将其理解为获取之意，获取对象dog的name属性。

## 3. 总结

追踪寻找，获取之意，本质上是相同的，你所需之物，被层层封装，或封装在模块里，或封装在类中，想要得到，就必须按照规则，使用“点”层层递进，直至触达。
