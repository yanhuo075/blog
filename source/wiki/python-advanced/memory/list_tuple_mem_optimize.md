---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 3.6 列表/元组内存分配优化
order: 5
---

# python 列表, 元组内存分配优化

## 1. 空元组与空列表

```python
>>> a = ()
>>> b = ()
>>> a is b
True
>>> id(a)
4374097992
>>> id(b)
4374097992
>>> a = []
>>> b = []
>>> a is b
False
```

元组是不可变对象，空元组只存在一个，实在是没有必要每次都创建出一个空元组来，列表则不同，列表是可变对象，每次都会创建出一个新的空列表

## 2. 小元组的分配优化

为了减少内存碎片，加快分配速度，python会重用旧的元组，如果一个元组不再被使用且元组的长度小于20，那么python不会直接释放它，而是将它移动到一个列表中，这个列表被分为20组，每一组存储一系列长度在0到20之间的元组，每个组可以最多存储2000个元组。第一组只存储了一个元组，正是空元组。

```python
>>> a = (1, 2, 3)
>>> id(a)
4377330024
>>> del a
>>> b = (1, 2, 4)
>>> id(b)
4377330024
```

元组a被del后，内存并不是真的被回收，它的这片内存空间可以被b直接使用，强调一点，元组的长度必须和a相同，这样才能重用这片内存。

## 3. 列表大小调整

python里的列表和C++中的vector很像，看似有无限的空间可以使用，但其实，他们总是预先分配一些容量，当存储的数据超过容量时，则采取一定的算法增加容量，这样做可以避免过于频繁的申请内存，又能保证插入效率。

python容量增长的方式为

```text
0、4、8、16、25、35、46、58、72、88，...
```

在不阅读源码的情况，我们可以通过简单的手段来验证这一说法

```python
>>> a = []
>>> import sys
>>> sys.getsizeof(a)
64
>>> a.append(1)
>>> sys.getsizeof(a)
96
>>> a.append(2)
>>> sys.getsizeof(a)
96
>>> a.append(3)
>>> sys.getsizeof(a)
96
>>> a.append(4)
>>> sys.getsizeof(a)
96
>>> a.append(5)
>>> sys.getsizeof(a)
128
```

一个空的列表占用64字节，向列表里写入一个元素后，整个列表占用的内存增加了32个字节，我们可以推测，这个时候增加了4个插槽的容量，每个插槽占用8个字节的大小。

继续向列表里新增元素，发现列表的大小竟然不发生变化了，直到写入第5个元素后，再一次超出了现有的容量，列表再次扩容，变为8，又增加了32个字节的内存，最终列表占用内存的大小为128。

下面的代码计算出列表增长过程中内存的变化

```python
import sys
from matplotlib import pyplot as plt
from matplotlib import font_manager

size = 100
length = []
memory = []
l = []


for counter in range(size):
    l.append(counter)
    length.append(len(l))
    memory.append(sys.getsizeof(l))

my_font = font_manager.FontProperties(fname="/Library/Fonts/Songti.ttc")
plt.plot(length, memory)
plt.title("python列表长度与内存增长关系", fontproperties=my_font)
plt.xlabel("列表长度", fontproperties=my_font)
plt.ylabel("内存", fontproperties=my_font)
plt.savefig('list_memory.png')
```

生成的曲线图如下
![python列表内存分配](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091818852.jpeg)
