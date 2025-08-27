---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.9 filter
order: 8
---

# filter

## 1. 一般用法

filter()函数被用于过滤序列，它会过滤掉不符合条件的数据，符合条件的数据将会被留下，filter函数返回的结果是一个可迭代对象。

之所以称它为高级语法，因为想要正确理解使用它并不容易，同时还要配合上lambda表达式。

filter的语法如下

```python
filter(function, iterable)
```

function 是判断函数，filter会遍历iterable里的每一个数据，用function进行判断，如果符合条件，才会被留下。

iterable 是可迭代对象，如列表，元组，甚至集合都可以

下面是一个简单的使用示例

```python
from collections import Iterable

lst = [1, 2, 5, 6, 7]
res = filter(lambda x: x % 2 == 0, lst)
# res 是一个可迭代对象
print(isinstance(res, Iterable), type(res))
for item in res:
    print(item)
```

程序输出结果

```text
True <class 'filter'>
2
6
```

理解代码的关键有3点
**1. filter 做了什么**
filter函数会遍历列表lst, 用lambda表达式来判断所遍历到的数据是否符合要求

**2. lambda 表达式做了什么**
列表lst中的数据会逐个传给lambda表达式进行判断，lambda表达式返回这些数是否为2的倍数，尽管没有return语句，但lambda表达式默认返回 x % 2 == 0 的结果

**3. filter函数的返回值是什么**
filter函数的返回值是一个可迭代对象，这一点很关键，这也是为什么我说filter函数是高级语法的原因。

为什么不返回列表？，如果返回的是列表，那么在filter函数执行过程中，就必须对列表里的每一个数据进行对2取模运算，这样很浪费空间，因此filter在实现时采用了迭代器技术，将计算延迟到对filter函数返回结果进行遍历时才进行。

怎么验证上面的说法呢，对代码稍加修改

```python
from collections import Iterable

def func(x):
    print("接收参数" + str(x))
    return x%2 == 0

lst = [1, 2, 5, 6, 7]
res = filter(func, lst)
# res 是一个可迭代对象
print(isinstance(res, Iterable), type(res))

for item in res:
    print(item)
```

现在，我不使用lambda表达式对数据进行判断，而是使用自由度更高的函数，在函数func中，我输出接收到的参数，如果对lst数据中的过滤发生执行filter函数期间，那么，在输出res类型之前，就会执行函数func中的print语句，现在来看程序实际的执行结果

```text
True <class 'filter'>
接收参数1
接收参数2
2
接收参数5
接收参数6
6
接收参数7
```

程序输出结果表明，在执行filter函数时，并没有调用func函数对lst中的数据进行过滤，而是在对res进行遍历时才调用func函数进行过滤

## 2. 结合偏函数，更加强大

单纯的使用filter函数，就已经很强大了，结合了偏函数，则更加强大

```python
from functools import partial

def even(x):
    return x%2 == 0

# 创建一个专门用来过滤保留偶数的函数
even_filter = partial(filter, even)
lst = [1, 2, 5, 6, 7]
res = even_filter(lst)

for item in res:
    print(item)
```

通过使用偏函数partial，我创建了一个专门用来过滤保留偶数的函数，其实最终还是调用执行了filter(even, lst)，但使用even_filter时，我们可以不用去管函数even,在创建even_filter时，已经约定even作为filter的第一个参数来使用了。

## 3. 自己实现filter

只要理解了filter的功能，实现一个类似的函数并不困难，定义一个函数

```python
def my_filter(func, iterobj):
    pass
```

函数接受两个参数，第一个参数是函数，可以是lambda函数也可以是自定义函数，第二个参数是可迭代对象。函数内部对可迭代对象进行遍历，并使用func函数检查所遍历到的对象是否符合要求，符合要求则使用yield返回，示例代码如下

```python
lst = [1, 2, 5, 6, 7]


def my_filter(func, iterobj):
    for item in iterobj:   # 遍历可迭代对象
        if func(item):  # 使用func函数判断对象是否符合条件要求
            yield item      # 使用yield返回

res = my_filter(lambda x: x % 2 == 0, lst)

for item in res:
    print(item)
```
