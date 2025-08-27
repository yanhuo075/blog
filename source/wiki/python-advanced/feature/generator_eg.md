---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.9 生成器的作用实例
order: 8
---

# 三个实例带你了解python生成器的作用

python的生成器究竟有什么作用? 本文通过3个具体的例子向你阐述生成器的作用, 1. 使用生成器非常便利的结束两层for循环, 2. 一个for循环遍历多个列表, 3. 并行遍历多个可迭代对象

提到生成器，你可能会有一个简单的概念，如果函数里使用yield关键字，那么这个函数就是一个生成器，不同于return，生成器使用yield来返回值。

令人感到困惑之处在于，似乎yield和return 没啥区别，但实质上区别非常大，最明显之处便在于，return 语句执行后，函数就退出了，而yield语句执行时，仅仅是返回一个值而已，不存在函数结束这个概念，因此生成器都要结合for循环进行使用。

下面通过几个示例，来向你阐释生成器的作用

## 1. 使用生成器非常便利的结束两层for循环

两层循环过程中，如果想通过break终止循环，是一件简单，但却很麻烦的事情,例如下面的这段代码

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]

stop = False
for i in lst1:
    for j in lst2:
        if i + j == 10:
            stop = True
            break
    
    if stop:
        break
```

两层for循环的目的非常简单，从两个列表中各找出一个数，使得他们的和等于10，而且只需找出一个组合即可。

找到满足要求的组合后，为避免不必要的循环，需要终止循环，而此时，if条件语句在for循环的最里层，此处执行break，只能跳出最里层的for循环，想要终止最外层的for循环，就必须传递终止信号给它，代码里，通过stop=True,告知外层for循环可以终止了。

这样的代码写起来，显然有些繁琐，最后的if stop判断总显得多余，面对这种情况，可以巧妙的利用生成器来避免这种复杂的写法。

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]


def num_generator(lst1, lst2):
    for i in lst1:
        for j in lst2:
            yield i, j


for i, j in num_generator(lst1, lst2):
    if i + j == 10:
        print(i, j)
        break
```

生成器num_generator里通过两层for循环对数据进行遍历，真正的业务逻辑使用一个for循环，这里就避免了跳出两层for循环的困境，这一次break，结束了for循环对num_generator的使用。

试想一下，如果没有生成器这种技术，有什么办法能实现一个break跳出两层for循环么？ 这种技术还可以扩展到更多层的for循环。

## 2. chain--一个for循环遍历多个列表

有这样一种应用场景，你需要遍历多个列表来执行某个操作，比如下面两个列表，你需要找出列表里所有的偶数

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]
```

### 2.1 嵌套循环

一种直接的方法是使用多层for循环

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]

for lst in [lst1, lst2]:
    for item in lst:
        if item % 2 == 0:
            print(item)
```

### 2.2 创建新列表

为了减少for循环的层次，你也可以创建一个新的列表，包含这两个列表里的所有数据

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]
lst = [*lst1, *lst2]

for item in lst:
    if item % 2 == 0:
        print(item)
```

### 2.3 chain

上面两个方法虽然都可以满足要求，但还有更简单的方法，使用chain

```python
from itertools import chain

lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]

for item in chain(lst1, lst2):
    if item % 2 == 0:
        print(item)
```

chain内部会对传入的参数逐个进行遍历，lst1 和 lst2仿佛成为一个整体，如果不使用python提供的这个chain类，我们自己可以通过生成器来实现一个效果相同的函数

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]


def my_chain(*iters):
    for iter_item in iters:
        for item in iter_item:
            yield item


for i in my_chain(lst1, lst2):
    if i % 2 == 0:
        print(i)
```

## 3. zip--并行遍历多个可迭代对象

使用chain时，是对列表逐个进行遍历，但有时，我们又希望是并行遍历，python原生的zip函数提供了这样的功能，下面是使用示例

```python
lst1 = ['python', 'java', 'c']
lst2 = [95, 97, 98]

dic = {}
for language, score in zip(lst1, lst2):
    dic[language] = score

print(dic)
```

两个列表，第一个列表是科目，第二个列表是分数，需要将这两个列表的内容转换为字典，使用zip函数，就可以并行遍历两个列表，接下来，我们自己使用生成器来实现一个相同功能的函数

```python
lst1 = ['python', 'java', 'c']
lst2 = [95, 97, 98]


def my_zip(*args):
    min_len = min(len(item) for item in args)
    index = 0
    while index < min_len:
        lst = []
        for iter_item in args:
            lst.append(iter_item[index])

        index += 1
        yield tuple(lst)


info = {}
for language, score in my_zip(lst1, lst2):
    info[language] = score
```
