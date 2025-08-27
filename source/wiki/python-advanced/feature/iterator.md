---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.10 迭代器和可迭代对象
order: 9
---

# 一文看懂python的迭代器和可迭代对象

阅读完本文，你将收获以下知识点：

1. 什么是迭代器和可迭代对象，他们之间有什么关系
2. 迭代器可以迭代的底层原理是什么
3. 如何自定义可迭代对象和迭代器

迭代器和可迭代对象是两个非常难以搞懂的python概念，我试图用最轻松的方式为你揭开它们的神秘面纱，你也将在这个过程中学习到研究技术的方法。

## 1. 判断一个对象是不是可迭代对象，是不是迭代器

**我研究的思路，第一步先找到一种办法，可以判断一个对象是不是可迭代对象，是不是迭代器。**

如果我掌握了这种方法，那么我就可以用它来识别哪些是可迭代对象，哪些是迭代器，这样做非常关键，在我对这两个概念还不熟悉时，最起码能知道谁是，谁不是。

我知道python有着非常强大的自省能力，python有办法判断一个对象是不是函数，是不是类，是不是类里的方法，一定有办法判断一个对象是不是可迭代对象或者迭代器，果真，经过一番谷歌，我找到了判断的方法

```python
from collections.abc import Iterable, Iterator

print(isinstance([1, 2, 3], Iterable))      # True
print(isinstance((1, 2, 3), Iterable))      # True
print(isinstance(set([1, 2, 3]), Iterable))      # True
print(isinstance("python", Iterable))       # True
print(isinstance({'a': 1}, Iterable))       # True
print("分割线----------------")
print(isinstance([1, 2, 3], Iterator))      # False
print(isinstance((1, 2, 3), Iterator))      # False
print(isinstance(set([1, 2, 3]), Iterator))      # False
print(isinstance("python", Iterator))      # False
print(isinstance({'a': 1}, Iterator))      # False
```

经过试验，常见的数据类型中，列表，元组，集合，字典，字符串都是可迭代对象，他们都不是迭代器。**所有可以用for循环遍历的对象，都是可迭代对象，甚至包括文件对象**

```python
from collections.abc import Iterable, Iterator

f = open('test.py')
print(isinstance(f, Iterable))          # True
print(isinstance(f, Iterator))          # True
```

上面的代码给了我一个惊喜，终于找到了一个迭代器。

## 2. 什么是可迭代对象

前面的研究只是开胃菜，接下来的才是重点，可迭代对象的定义如下：
**如果一个对象实现了__iter__方法，那么这个对象就是可迭代对象**。

我们来验证一下这个定义是否成立

```python
from collections.abc import Iterable, Iterator


class Color(object):

    def __init__(self):
        self.colors = ['red', 'white', 'black', 'green']

    # 仅仅是实现了__iter__ 方法,在方法内部什么都不做
    def __iter__(self):
        pass

color_object = Color()
# 判断是否为可迭代对象
print(isinstance(color_object, Iterable))       # True
# 判断是否为迭代器
print(isinstance(color_object, Iterator))       # False
```

你现在能理解我为什么在文章的第一小节里寻找可以判断对象是否为可迭代对象的原因了吧，我需要验证可迭代对象的定义，只有这样才算是把这个知识点吃透。

在上面的定义中，我只是定义了__iter__方法，方法内什么都没有做，可它符合可迭代对象的定义，不过我要强调，它是可迭代对象，然而却不能被迭代，至于原因，要等到阅读完全文以后才能揭晓。

## 3. 什么是迭代器

迭代器的定义如下：**如果一个对象同时实现了__iter__方法和__next__方法，它就是迭代器**。

按照这个定义，我对第二小节中的Color类进行改造

```python
from collections.abc import Iterable, Iterator


class Color(object):

    def __init__(self):
        self.colors = ['red', 'white', 'black', 'green']

    # 仅仅是实现了__iter__ 方法,在方法内部什么都不做
    def __iter__(self):
        pass

    def __next__(self):
        pass

color_object = Color()
# 判断是否为可迭代对象
print(isinstance(color_object, Iterable))       # True
# 判断是否为迭代器
print(isinstance(color_object, Iterator))       # True
```

改造后，color_object 是可迭代对象，也是迭代器，尽管它不能正常的工作，但这并不影响它的身份。同时我们也可以得出一个结论，**迭代器一定是可迭代对象**，因为迭代器要求必须同时实现__iter__方法和__next__方法， 而一旦实现了__iter__方法就必然是一个可迭代对象。但是反过来则不成立，可迭代对象可以不是迭代器。

## 4. 迭代器工作原理

接下来，我们要研究一下迭代器是如何工作的，它是怎样实现迭代的，首先，我们要认识一下内置函数iter

### 4.1 内置函数iter获得迭代器

**iter函数的作用是从可迭代对象那里获得一个迭代器**， 我们设计一个实验来验证这个说法

```python
from collections.abc import Iterator

lst_iter = iter([1, 2, 3])
print(isinstance(lst_iter, Iterator))       # True
```

所言非虚，iter会返回一个迭代器

### 4.2 使用内置函数next遍历迭代器

**内置函数next的功能是从迭代器那里返回下一个值**，设计实验来验证它

```python
from collections.abc import Iterator

lst_iter = iter([1, 2, 3])
print(next(lst_iter))       # 1
```

实践与理论完美结合，让我们多调用几次next函数

```python
from collections.abc import Iterator

lst_iter = iter([1, 2, 3])
print(next(lst_iter))       # 1
print(next(lst_iter))       # 2
print(next(lst_iter))       # 3
print(next(lst_iter))       # StopIteration
```

前3次调用next函数都能正常工作，第4次会抛出StopIteration异常，迭代器里已经没有下一个值了。

现在，让我们来做一个总结，遍历迭代器需要使用next方法，每调用一次next方法，就会返回一个值，没有值可以返回时，就会引发StopIteration异常。

### 4.3 为什么迭代器不能重复使用

有了4.2的铺垫，你应当已经理解迭代器不能重复使用的原因，next方法永远返回下一个值，第一次调用时，返回的是第一个值，就本例而言是1，下一次调用时，下一个值是2，返回的必然是2。此时，如果你想从头遍历，该怎么办呢，很简单，使用iter函数重新获得一个迭代器。

```python
from collections.abc import Iterator

lst_iter = iter([1, 2, 3])
print(next(lst_iter))       # 1
print(next(lst_iter))       # 2  到了这一步，你想从头开始遍历，那么重新获得一个迭代器使用

lst_iter_2 = iter([1, 2, 3])
print(next(lst_iter_2))     # 1
print(next(lst_iter_2))     # 2
print(next(lst_iter_2))     # 3
```

### 4.4 for 循环的工作原理

for循环的工作原理可以描述为如下动作：

1. 使用iter获得可迭代对象的迭代器
2. 反复对迭代器使用next方法
3. 捕获StopIteration异常，退出循环

## 5. 自定义可迭代对象和迭代器

当我们对一个概念和底层原理足够了解后，我们一定要自己去实现它，一方面验证自己对理论的理解是否正确，一方面加深对底层原理的应用能力。在第3节，已经实现了一个Color类，它的实例既是可迭代对象，又是迭代器，但它不能工作，因为__iter__方法和__next__方法都没有具体实现

### 5.1 实现__iter__方法

如果可迭代对象实现了__iter__方法，那么内置函数iter会调用对象的__iter__方法方法返回一个迭代器，由于Color类实现了__next__方法，因此Color的实例也是迭代器，在__iter__方法里返回self即可。

```python
class Color(object):

    def __init__(self):
        self.index = -1
        self.colors = ['red', 'white', 'black', 'green']

    def __iter__(self):
        self.index = -1
        return self
```

我增加了一个index属性，初始值设置为-1，对于这个属性的用途，你一会便知

### 5.2 实现__next__方法

我们使用内置函数next对迭代器进行遍历，在这个过程中，是在调用迭代器的__next__方法, 内置函数的作用是返回迭代器的下一个值，这个功能的实现，我们需要放在__next__方法中。

```python
    def __next__(self):
        self.index += 1
        if self.index >= len(self.colors):
            raise StopIteration

        return self.colors[self.index]
```

怎样才能做到返回下一个值呢？在调用__next__方法时，我将索引index的值加1，这样下一次执行return self.colors[self.index]时就会返回下一个值，这里再一次解释了迭代器不能重复使用的原因。

### 5.3 用for循环遍历

```python
class Color(object):

    def __init__(self):
        self.index = -1
        self.colors = ['red', 'white', 'black', 'green']

    def __iter__(self):
        self.index = -1
        return self

    def __next__(self):
        self.index += 1
        if self.index >= len(self.colors):
            raise StopIteration

        return self.colors[self.index]

color_object = Color()
for color in color_object:
    print(color)
```

程序输出结果

```text
red
white
black
green
```

## 6. 迭代器的数量问题

### 6.1 只能获得一个迭代器

在第5节中，color_object是可迭代对象，iter(color_object)会返回一个迭代器，现在请思考，如果多次对color_object调用iter函数，所得到的迭代器是同一个还是多个？如果想不清楚，不妨实验一下

```python
color_object = Color()
iter_1 = iter(color_object)
iter_2 = iter(color_object)
iter_3 = iter(color_object)

print(id(iter_1))       # 2683506554888
print(id(iter_2))       # 2683506554888
print(id(iter_3))       # 2683506554888
```

三个迭代器的内存地址相同，说明它是同一个对象，不论对color_object调用多少次，返回的都是同一个迭代器，这是因为__iter__方法方法里返回的是self，正是color_object本身。这样一来，iter能够从color_object那里获得迭代器永远只有一个。

这并不是bug，而是一个设计问题，文件对象也只有一个迭代器，新建一个脚本 test.py

```python
f = open('test.py')
iter_1 = iter(f)
iter_2 = iter(f)

print(id(iter_1) == id(iter_2))    # True
print(next(iter_1))
print(next(iter_2))
```

对于这种设计，我是这样理解的，打开文件后，对文件对象的遍历是通过文件指针进行的，而文件指针只有一个。遍历到文件内容的第3行时，文件指针就指向了这里，如果有多个迭代器，就不得不在迭代器里维护自己遍历时的行号，多个迭代器同时工作，文件指针就要在文件里跳来跳去，而且最关键的是，文件内容遍历一次就应当获得了想要的内容，不应该多次遍历，想多次遍历，请多次打开。

### 6.2 可以获得多个迭代器

以列表为例，使用iter方法可以获得多个迭代器

```python
lst = [1, 2, 3]
iter_1 = iter(lst)
iter_2 = iter(lst)

print(id(iter_1) == id(iter_2))   # False
```

迭代器iter_1 和 iter_2 不是同一个对象，他们可以分开同时进行遍历，互不影响。前面已经介绍了，列表是可迭代对象，不是迭代器，那么在列表的__iter__方法里，就不能像我在Color的__iter__方法里定义的一样，返回self实例，而是要创建出一个迭代器。

列表的__iter__方法是用C语言实现的，我们看不到源码，但根据前面的理论储备，我们可以模拟这个过程。

```python
lst = [1, 2, 3]

class ListIterator:
    def __init__(self, lst):
        self.lst = lst
        self.index = -1

    def __iter__(self):
        return self

    def __next__(self):
        self.index += 1
        if self.index > len(self.lst):
            raise StopIteration

        return self.lst[self.index]


def my_iter(lst):
    return ListIterator(lst)

iter_1 = my_iter(lst)
iter_2 = my_iter(lst)
print(id(iter_1) == id(iter_2))   # False
```

ListIterator 的实现，和Color类的实现几乎完全相同。

## 7. 总结

### 7.1 什么是迭代器和可迭代对象，他们之间有什么关系

1.迭代器： **如果一个对象同时实现了__iter__方法和__next__方法，它就是迭代器**

2.可迭代对象： **如果一个对象实现了__iter__方法，那么这个对象就是可迭代对象**

3.他们之间有什么关系： **迭代器一定是可迭代对象，反之则不成立，可迭代对象的__iter__方法必须返回一个迭代器**

### 7.2 迭代器可以迭代的底层原理是什么

使用next函数可以返回迭代器下一个值，所谓迭代就是不停的调用next函数直到引发StopIteration异常，next函数内部调用迭代器的__next__方法，具体返回哪个值是由__next__来决定的

### 7.3 如何自定义可迭代对象和迭代器

根据迭代器和可迭代对象的概念，自行定义类即可，5,6两节都给出了具体实现示例。
