---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.9 字典视图对象
order: 8
---

# 有趣的python字典视图对象

## 1. 什么是字典视图对象

python中，字典视图对象是只读的字典代理，可以使用字典的keys, values，items以及types模块的MappingProxyType来创建。关于MappingProxyType你可能没有见过，但前面所提到的3个方法，你一定有印象。

自以为对python已经了解的足够深入了，竟然也是才知道有字典视图对象这个概念，自己之前对keys，values方法的理解也有错误。

在python2中，字典的keys(),values(),items()方法返回的都是列表，但在python3中，他们的返回值不再是字典，而是有各自的类型

```python
stu_dict = {
    '小明': 14,
    '小红': 13
}

keys = stu_dict.keys()
values = stu_dict.values()
items = stu_dict.items()

print(type(keys))
print(type(values))
print(type(items))
```

程序输出结果

```text
<class 'dict_keys'>
<class 'dict_values'>
<class 'dict_items'>
```

看源码注释，解释的非常清楚

1. keys方法返回的是一个类集合的对象，是字典所有key的一个视图
2. values方法返回的是字典所有value的视图
3. items方法返回的是字典所有键值对的视图

## 2. 如何理解视图的概念

python2中keys方法返回的是列表，实实在在的数据，而python3中返回的是一个类集合的，字典所有key的视图。在mysql里，视图表并不真正存储数据，但透过视图表，可以看到真实的数据，视图表将若干个真实存储数据的表里的数据整合模拟出一张只读的表。

keys方法返回的正是这样一个对象，它并不真正存储字典的key，但通过dict_keys对象，可以访问字典的key

```python
print('小明' in keys)
for key in keys:
    print(key)
```

上面的两个操作都是可行的，但你绝不可以对它进行任何的修改，因为字典的视图对象都是只读的。

```text
keys = stu_dict.keys()
stu_dict['小刚'] = 15
print(keys)   # dict_keys(['小明', '小红', '小刚'])
```

在通过keys方法获得字典的所有key之后，对字典的修改，会直接反应在已经获得字典视图对象身上，这更加充分的说明，字典视图对象是不直接存储数据的，它只是字典数据的一个代理

## 3. 字典视图对象不是生成器

有一段时间，我以为keys,values，items返回的数据的类型是生成器，因为这样可以节省内存，但后来发现它不具备生成器的特性，比如，他们都支持反复遍历，这显然是生成器做不到的，实际的检测也证实这一点

```python
from types import GeneratorType

print(isinstance(keys, GeneratorType))   # False
print(isinstance(values, GeneratorType))   # False
print(isinstance(items, GeneratorType))   # False
```

能对其进行遍历，那起码得是可迭代对象

```python
from collections import Iterable
print(isinstance(keys, Iterable))  # True
print(isinstance(values, Iterable))  # True
print(isinstance(items, Iterable))  # True
```

## 4. MappingProxyType 创建字典的整体视图

```python
from types import MappingProxyType
map_dict = MappingProxyType(stu_dict)
print(map_dict)
map_dict['小红'] = 14     # 会报错
```

使用types模块的MappingProxyType，可以创建整个字典的视图，对这个视图的所有读操作都是支持的，但任何尝试修改的操作都将引发错误。如果你需要在设计程序时确保一个字典不被修改，那么可以考虑使用字典的视图对象，任何人拿到一个字典的视图，都无法修改它。

需要提醒你注意一点，字典的视图不能修改，但修改原字典后，字典的视图也等同于被修改，要想清楚一点，视图只是字典数据的只读的代理而已。
