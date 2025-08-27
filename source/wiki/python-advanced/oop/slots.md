---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.10 使用__slots__节省内存空间
order: 9
---

# python面向对象--使用__slots__节省内存空间

假如你的python程序需要创建大量对象，这个数量级达到了百万，那么你可以通过使用__slots__来节省内存

先来看一个普通类

```python
import os
import psutil

class Stu:
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

lst = []
for i in range(1000000):
    lst.append(Stu('小明', 18, 600))

process = psutil.Process(os.getpid())
print('Used Memory:',process.memory_info().rss / 1024 / 1024,'MB')
```

对于普通定义的类，每个实例都会创建一个字典来保存实例属性，也就是__dict__。熟悉数据结构的朋友肯定知道，为了实现O(1)的查找效率，字典的实现额外的使用一些内存空间，即便是一个空字典也要占用200多字节的空间

```python
print(sys.getsizeof({}))
```

如果你创建几百万个Stu实例，内存的消耗就是你不得不考虑的事情，上面的代码执行会占用176M的内存空间。

使用了__slots__后，就不再为每个实例创建用于保存属性的字典，由于属性已经固定，不允许增加，因此python在内部采取一种更加紧凑的内部表示，这和列表和元组有些相似。

```python
import os
import psutil

class Stu:
    __slots__ = ['name', 'age', 'score']
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

lst = []
for i in range(1000000):
    lst.append(Stu('小明', 18, 600))

process = psutil.Process(os.getpid())
print('Used Memory:',process.memory_info().rss / 1024 / 1024,'MB')
```

使用内存为76M
