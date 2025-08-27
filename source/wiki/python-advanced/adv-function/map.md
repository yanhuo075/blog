---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.6 map
order: 5
---

# map

## 1. map函数的一般用法

map函数可以根据指定函数对序列做映射，下面是map函数的语法

```python
map(func, *iterables)
```

第一个参数是函数，可以是lambda函数也可以是自定义函数， 第二个参数是可变参数，可以传入多个可迭代对象，下面来看一个简单的应用示例

```python
lst1 = [1, 2, 3]

lst = map(lambda x: x*2, lst1)
print(lst)

for item in lst:
    print(item)
```

程序输出结果

```text
<map object at 0x105664358>
2
4
6
```

map函数的第一个参数是lambda函数，它接受lst1里的元素并乘以2返回，最终的效果就是返回了一个新的序列，这个序列以lst1为基础，每个元素乘以2.

## 2. 传入多个可迭代对象

能正确传入多个可迭代对象，才算是真正掌握了map函数

```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]

lst = map(lambda x, y: x*y, lst1, lst2)
print(lst)

for item in lst:
    print(item)
```

这段代码里，传入了lst1,lst2两个可迭代对象，在map的内部，会同时遍历他们，从这两个列表里各自取出一个元素作为参数传给lambda函数，最终的效果就是1*4， 2*5，3*6， 程序输出结果

```text
<map object at 0x1032f1400>
4
10
18
```

## 3. 自己实现map函数

map函数可以接收多个可迭代对象，如果只考虑列表和元组，那么实现这个函数就比较简单，因为他们都有索引，但如果加上集合，事情就变得有些麻烦，集合是没有索引概念的，我先来实现一个只考虑可迭代对象为列表或元组的情况

```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]


def my_map(func, *iterobj):
    min_len = min([len(item) for item in iterobj])
    index = 0
    while index < min_len:
        tmp = [item[index] for item in iterobj]
        index += 1
        yield func(*tmp)   # 解包传入参数


lst = my_map(lambda x, y: x*y, lst1, lst2)
print(lst)

for item in lst:
    print(item)
```

在my_map函数内部，先获得所传入的可迭代对象的最小长度，利用循环，每次从所传入的可迭代对象里各自获取一个元素，组装成列表，接下来问题的关键是如何传入到函数中，tmp是一个列表，在函数调用时，tmp前面加上星号，这是一个解包的过程，列表里的数据会像平时调用函数那样依次传入函数中。

接下来考虑可迭代对象是集合的情况，由于集合没有索引，因此无法通过索引来获取数据，为了解决获取数据，可以使用next函数。

```python
lst2 = {4, 5, 6}


def my_map(func, *iterobj):
    lst = [iter(item) for item in iterobj]
    while True:
        tmp = []
        for iter_item in lst:
            tmp.append(next(iter_item))
        yield func(*tmp)


lst = my_map(lambda x, y: x*y, lst1, lst2)
print(lst)

for item in lst:
    print(item)
```

iter函数作用于可迭代对象，返回一个迭代器，next方法可以从迭代器里获取获取数据，这也是for循环内部的真实操作，当迭代器被迭代到末尾时，再次使用next方法将引发异常，for循环就是根据这个异常来终止循环的。
