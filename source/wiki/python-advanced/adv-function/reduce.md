---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.7 reduce
order: 6
---

# reduce

## 1. reduce一般用法

reduce在2.x中是内置函数，到了python3以后移动到了functools 模块，它的作用是从左到右对一个序列的项累计地应用有两个参数的函数，以此合并序列到一个单一值， 下面先来看一个简单示例。

```python
from functools import reduce


lst = [2, 3, 4]
res = reduce(lambda x, y: x*y, lst)
print(res)
```

对reduce功能理解的关键在于累计二字，第一次计算时，传入到lambda函数中的两个参数是2和3，计算的结果是6，第二次传入lambda的是6和4，6是上一次的计算结果，会作为下一次计算的参数传入。

reduce的第一个参数可以是lambda函数，也可以是自定义函数。

## 2. 自己实现reduce

为了简化难度，只考虑可迭代对象是列表和元组的情况。

```python
def my_reduce(func, iterobj):
    if len(iterobj) < 2:
        return None

    res = func(iterobj[0], iterobj[1])
    for index in range(2, len(iterobj)):
        res = func(res, iterobj[index])

    return res


lst = [2, 3, 4]
res = my_reduce(lambda x, y: x*y, lst)
print(res)
```

首先要考虑的是可迭代对象的长度，如果长度小于2，我认为就没有办法计算了，因此返回None。取可迭代对象的前两项数据，使用func函数进行计算，所得到的结果作为下一次计算的参数传入。
