---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 4.2 标准模块
order: 1
---

# 标准模块导读

本章将集中向你介绍python的主要标准模块。这些模块包括：

1. collections
2. functools
3. contextlib
4. itertools
5. operator
6. shutil
7. filecmp
8. mmap

学习这些模块，将有助于你更深入更全面的了解python，在实际应用过程中，使用这些更高级的模块，也将使你的程序事半功倍。



# collections模块导读

collections实现了许多特定容器，这些容器在某些情况下可以替代内置容器 dict, list, tuple, set， 原因在于，他们提供了在某个方面更加强大的功能。本节，将向你介绍以下几个特定容器：

1. Counter--统计对象的个数
2. ChainMap---合并多个字典
3. OrderedDict---有序字典
4. defaultdict---不会引发KeyError的字典
5. deque---一个类似列表的容器
6. namedtuple---有属性名称的元组



# Counter--统计对象的个数

Counter类可以统计对象的个数， 它是字典的子类, 它有3种创建方法, 它支持访问缺失键, 更新计数器, 返回top N数据, 算术和集合操作，下面讲解Counter类的用法

## 1. 创建

Counter有3种创建对象的方法

```python
from collections import Counter

c1 = Counter()      # 创建一个空的Counter对象
c2 = Counter('hello world')      # 从一个可迭代对象(列表,元组,字典,字符串)创建
c3 = Counter(a=3, b=4)      # 从一组键值对创建

print(c2)
print(c3)
```

程序输出结果

```python
Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})
Counter({'b': 4, 'a': 3})
```

只看c2的话，不禁让人感到以后，使用字符串的方法不是可以实现相同的功能么？

```python
word = 'hello world'
print(word.count('h'))  # 1
print(word.count('l'))  # 3
```

其实不然，虽然列表，字符串都提供了count方法，但是想要查看所有元素的数量，却需要你逐个调用才能获取，而Counter类则要简单便捷的多。

## 2. 访问缺失的键

Counter虽然是字典的子类，但访问缺失的键时，不会引发KeyError, 而是返回0

```python
from collections import Counter

c1 = Counter()      # 创建一个空的Counter对象
print(c1['apple'])  # 0
```

## 3. 计数器更新

更新有两种方法，一种是使用update方法，一种是使用subtract方法， update方法用来新增计数， subtract方法用来减少计数， 分别来演示

**使用update**

```python
from collections import Counter

c1 = Counter('hello world')
c1.update('hello')  # 使用另一个iterable对象更新
print(c1['o'])      # 3

c2 = Counter('world')
c1.update(c2)       # 使用另一个Counter对象更新
print(c1['o'])      # 4
```

**使用subtract**

```python
from collections import Counter

c1 = Counter('hello world')
c1.subtract('hello')  # 使用另一个iterable对象更新
print(c1['o'])      # 1

c2 = Counter('world')
c1.subtract(c2)       # 使用另一个Counter对象更新
print(c1['o'])      # 0
```

## 4. 键的删除

同字典一样，使用del即可删除键值对

```python
from collections import Counter

c1 = Counter('hello world')
del c1['o']
print(c1['o'])      # 0
```

## 5. elements()

elements()返回一个迭代器，一个元素的计数是多少，在迭代器中就会有多少

```python
from collections import Counter

c1 = Counter('hello world')
lst = list(c1.elements())
print(lst)      # ['h', 'e', 'l', 'l', 'l', 'o', 'o', ' ', 'w', 'r', 'd']
```

## 6. most_common([n])

most_common返回top N的列表，列表里的元素是元组，如果计数相同，排列无指定顺序, 如果不指定n， 则返回所有元素

```python
from collections import Counter

c1 = Counter('hello world')
print(c1.most_common(2))        # [('l', 3), ('o', 2)]
```

## 7. 算术和集合操作

Counter类还支持+、-、&、|操作， &和|操作分别返回两个Counter对象各元素的最小值和最大值， 需要强调一点， 通过算数和集合操作得到Counter对象将删除计数值小于1的元素

```python
from collections import Counter

c = Counter(a=1, b=3)
d = Counter(a=2, b=2)
print(c + d)  # Counter({'b': 5, 'a': 3})
print(c - d)  # Counter({'b': 1})
print(c & d)  # Counter({'b': 2, 'a': 1})
print(c | d)  # Counter({'b': 3, 'a': 2})
```



# ChainMap---合并多个字典

collections模块的ChainMap可以将多个字典组合成一个可更新的视图, 在使用时, 允许你将多个字典视为一个字典, 这正是ChainMap的作用

## 1. 遍历多个字典

```python
dic1 = {'python': 100}
dic2 = {'c++': 99}
```

这是两个简单的字典，假如要求你对他们进行遍历，你该如何操作呢？最粗暴的方法是用两个for循环分别对他们进行遍历，但这样做未免过于繁琐

```python
dic1 = {'python': 100}
dic2 = {'c++': 99}

for key, value in dic1.items():
    print(key, value)

for key, value in dic2.items():
    print(key, value)
```

## 2. 合并遍历

ChainMap 并不是真的将字典合并在一起，那样的话会产生一个新的字典，使用ChainMap可以让我们以更简洁的代码遍历多个字典

```python
from collections import ChainMap

dic1 = {'python': 100}
dic2 = {'c++': 99}

dic = ChainMap(dic1, dic2)
print(type(dic))

for key, value in dic.items():
    print(key, value)
```

程序输出结果

```text
<class 'collections.ChainMap'>
c++ 99
python 100
```

需要注意的是ChainMap返回的对象类型并不是字典

## 3. 自己动手实现类似的功能

你可能会好奇，ChainMap是怎么做到允许你像往常一样遍历字典的，其实非常简单，ChainMap 将多个字典保存到列表里，当你遍历dic时，它在内部循环这个列表，对每一个字典进行遍历。

为了进阶提升，我们可以利用生成器实现类似的效果

```python
dic1 = {'python': 100}
dic2 = {'c++': 99}


def pair_chain(*args):
    for dic in args:
        for key in dic:
            yield key, dic[key]


for key, value in pair_chain(dic1, dic2):
    print(key, value)
```

效果一点都不比ChainMap差



# python有序字典---OrderedDict

OrderedDict是dict的子类，它会记录第一次插入键的顺序，这使得你可以按照键的插入顺序来遍历OrderedDict，需要指出的是，从python3.6 开始内置的字典也具备这种能力，因此很多人质疑OrderedDict是否还有存在的必要，但仔细观察 OrderedDict 将发现该类仍然提供有价值的特性。

### 1. 有序字典与无序字典

首先声明一点，在python3.6 之前，字典都是无序的。

```python
dic = {}

dic['a'] = 'a'
dic['b'] = 'b'
dic['c'] = 'c'

for key, value in dic.items():
    print(key, value)
```

上面这段代码，在python2.7环境下执行的结果是

```text
('a', 'a')
('c', 'c')
('b', 'b')
```

在python3.6 环境下执行的结果是

```text
a a
b b
c c
```

从python3.6开始，字典已经是有序的了，这里所说的有序，并不是按照key的大小进行排序，而是指插入的顺序，特别强调一点，这个特性不是python这门语言的特性，而是CPython的特性。

### 2. OrderedDict 有序字典

在python3.6之前，想要实现有序字典，就必须依靠OrderedDict，在3.6以后，则不再是必须,如果还有什么理由使用OrderedDict的话，OrderedDict提供了move_to_end方法，可以将一个key-value移动到末尾，使用popitem方法可以按照先进后出的原则删除最后加入的key-value对

```python
from collections import OrderedDict


order_dict = OrderedDict()

order_dict[1] = 1
order_dict['a'] = 2
order_dict['0'] = 3

for key, value in order_dict.items():
    print(key, value)

print("*"*20)
order_dict.move_to_end(1)

for key, value in order_dict.items():
    print(key, value)
```



# defaultdict---不会引发KeyError的字典

### 1. KeyError异常

使用python标准模块提供的字典时,如果所用的key不存在, 就会引发KeyError异常, 但使用collections 模块提供的defaultdict就可以避免这种错误

```python
dic = {}
print(dic['python'])
```

异常内容为

```text
    print(dic['python'])
KeyError: 'python'
```

### 2. defaultdict 一般用法

defaultdict是字典的子类，它的特殊在于永远不会引发KeyError 异常，下面的代码是一段示例代码

```python
from collections import defaultdict

dic = defaultdict(int)
print(dic['python'])
```

在创建这种字典时，必须传入一个工厂函数，比如int, float, list, set。defaultdict内部有一个__missing__(key) 方法，当key不存在时，defaultdict会调用__missing__ 方法根据你所传入的工厂函数返回一个默认值。你传入的工厂函数是int，那么返回的就是0， 如果传入的是list，返回的就是空列表。

### 3. defaultdict应用

既然不会引发KeyError，而且还能获取默认值，那么就可以利用这两个特点来简化一些算法

```python
lst = [1, 3, 4, 2, 1, 3, 5]
```

现在请写代码统计列表里各个值出现的次数，如果使用普通的dict类来实现，代码可以这样写

```python
lst = [1, 3, 4, 2, 1, 3, 5]
count_dict = {}

for item in lst:
    if item not in count_dict:
        count_dict[item] = 0

    count_dict[item] += 1

print(count_dict)
```

使用defaultdict，则可以更加简洁一些，代码缩进更少

```python
from collections import defaultdict

lst = [1, 3, 4, 2, 1, 3, 5]
count_dict = defaultdict(int)

for item in lst:
    count_dict[item] += 1

print(count_dict)
```



# deque---一个类似列表的容器

阅读本篇教程，需要你具备一定的数据结构功底，否则，可能什么也看不懂。

很多人把collections模块下的deque说成是双端队列，这是极不准确的，标准模块里对它的定义如下

```text
list-like container with fast appends and pops on either end
```

翻译过来应该是： 类似列表的容器，支持在两端快速的追加和删除元素。它是如此的灵活，提供了以下方法：

- append （在末尾追加数据）
- appendleft （在头部追加数据）
- pop （删除并返回指定索引的数据，默认是末尾数据）
- popleft（删除并返回头部数据）
- insert（在指定位置插入数据）
- remove（删除指定数据）

这些方法，已经远远超出了双端队列所能提供的方法，我个人对deque是这样理解的，它提供了及其灵活的功能，自身并不属于你所熟悉的任何一种数据结构，但我们可以在它的基础上非常轻松的实现数据结构，比如单向队列，双端队列，栈。

鉴于python标准库里已经实现了单向队列，所以本篇文章以deque为基础实现双端队列和栈这两种结构，向你展示如何使用deque

## 1. 实现双端队列

![双端队列](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827092702316.jpeg)

代码实现

```python
from collections import deque


class DoubleQueue():
    def __init__(self):
        self.queue = deque()

    def is_empty(self):
        """
        判断是否为空队列
        :return:
        """
        return len(self.queue) == 0

    def insert_front(self, data):
        """
        从队首插入数据
        :param data:
        :return:
        """
        self.queue.appendleft(data)

    def insert_rear(self, data):
        """
        从队尾插入数据
        :param data:
        :return:
        """
        self.queue.append(data)

    def delete_front(self):
        """
        从队首删除数据
        :return:
        """
        return self.queue.popleft()

    def delete_rear(self):
        """
        从队尾删除数据
        :return:
        """
        return self.queue.pop()

    def size(self):
        """
        返回队列长度
        :return:
        """
        return len(self.queue)


dq = DoubleQueue()

print(dq.is_empty())
dq.insert_front(4)
dq.insert_rear(5)
dq.insert_rear(6)

print(dq.delete_front())
print(dq.delete_rear())

print(dq.size())
```

## 2. 实现栈

![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827115118759.jpeg)

代码实现

```python
from collections import deque


class Stack:
    def __init__(self):
        self.stack = deque()

    def push(self, data):
        """
        入栈
        :param data:
        :return:
        """
        self.stack.append(data)

    def pop(self):
        """
        出栈
        :return:
        """
        return self.stack.pop()

    def size(self):
        """
        栈大小
        :return:
        """
        return len(self.stack)

    def is_empty(self):
        return self.size() == 0


stack = Stack()

stack.push(4)
stack.push(3)
stack.push(1)

print(stack.pop())
print(stack.size())
```



# namedtuple --- 有属性名称的元组

## 1. 有属性的元组

对元组里数据的访问只能通过索引来进行，假设我们想用元组来表示一个平面上的坐标，可以这样写

```python
point = (3, 5)

# 输出x,y坐标
print(point[0], point[1])
```

namedtuple 允许我们创建有属性名称的元组，这样就可以通过属性名称来获取数据

```python
from collections import namedtuple

Point = namedtuple('Point', ['x_coord', 'y_coord'])
print(issubclass(Point, tuple))
point = Point(3, 5)

# 输出x,y坐标
print(point.x_coord, point.y_coord)
```

程序输出结果

```text
True
3 5
```

特别强调一点，Point是tuple的子类

## 2. 为什么不用类

有人会有疑惑，对于上面的问题，明明可以用类来解决啊，为什么弄出一个namedtuple？

```python
class Point():
    def __init__(self, x, y):
        self.x_coord = x
        self.y_coord = y

point = Point(3, 5)
# 输出x,y坐标
print(point.x_coord, point.y_coord)
```

为什么不使用类，而使用namedtuple，最常见的解释是定义一个类大材小用，没有必要，对此我是不认同的，原因有两点：

1. 创建一个namedtuple对象，不比创建一个类简单
2. namedtuple 返回的对象本身就是一个类，这和直接创建类没有区别

真正的原因是元组，元组是不可变对象，因此元组可以做字典的key，可以存储到集合中，如果我们用自定义的普通类来替代namedtuple，一旦需要做字典的key，那么普通的类创建出的对象就无能为力了。而namedtuple创建的是tuple的子类，因此具有tuple的一切属性。



# functools模块导读

functools模块提供了一些非常神奇的高阶函数， 其中为人熟知的有reduce，partial，wraps， 他们是作用于或返回其他函数的函数， 一般来说任何可调用的对象都可以用这个模块里的函数进行处理。

本模块包含以下函数

1. partial--偏函数



# partial---偏函数

## 1. 偏函数

偏函数partial是functools 模块里提供的一个函数。和装饰器对比来理解，装饰器改变了一个函数的行为，而偏函数不能改变一个函数的行为。偏函数只能根据已有的函数生成一个新的函数，这个新的函数完成已有函数相同的功能，但是，这个新的函数的部分参数已被偏函数确定下来

## 2. 场景示例

### 2.1 常规实现

为了便于理解，我们构造一个使用场景，假设我们的程序要在dest目录下新建一些文件夹，那么常见的实现功能代码如下

```python
import os
from os import mkdir


mkdir(os.path.join('./dest', 'dir1'))
mkdir(os.path.join('./dest', 'dir2'))
mkdir(os.path.join('./dest', 'dir3'))
```

功能很简单，代码很简洁，但是有个小小的不如意之处，每次都是在dest目录下新建文件夹，既然它这么固定，是不是可以不用传递dest参数呢？

## 2.2 偏函数实现

```python
import os
from os import mkdir
from functools import partial


dest_join = partial(os.path.join, './dest')

mkdir(dest_join('dir1'))
mkdir(dest_join('dir2'))
mkdir(dest_join('dir3'))
```

dest_join是partial创建出来的一个新的函数，这个函数在执行时会调用os.path.join并将'./dest'作为参数传给join

偏函数将预先确定的参数冻结，起到了缓存的作用，在获得了剩余的参数后，连同之前冻结的确定参数一同传给最终的处理函数。

虽然看起来代码没有减少，但如果你实际使用就会发现，用了偏函数，免去了反复输入相同参数的麻烦，尤其当这些参数很多很难记忆时



# cmp_to_key

python标准模块functools中的cmp_to_key可以将一个cmp函数变成一个key函数，从而支持自定义排序

python的列表提供了sort方法，下面是该方法的一个示例

```python
lst = [(9, 4), (2, 10), (4, 3), (3, 6)]
lst.sort(key=lambda item: item[0])
print(lst)
```

sort方法的key参数需要设置一个函数，这个函数返回元素参与大小比较的值，这看起来没有问题，但如果想实现更加复杂的自定义排序，就不那么容易了。

目前的排序规则是根据元组里第一个元素的大小进行排序，我现在修改规则，如果元组里第一个元素是奇数，就用元组里第一个元素进行排序，如果元组里第一个元素是偶数，则用这个元组里的第二个元组进行大小比较，面对这样的需求，列表的sort方法无法满足。

对于这种情形，可以使用functools.cmp_to_key来解决

```python
from functools import cmp_to_key
lst = [(9, 4), (2, 10), (4, 3), (3, 6)]

def cmp(x, y):
    a = x[0] if x[0] %2 == 1 else x[1]
    b = y[0] if y[0] %2 == 1 else y[1]

    return 1 if a > b else -1 if a < b else 0

lst.sort(key=cmp_to_key(cmp))
print(lst)
```

仍然使用sort进行排序，我实现了一个cmp函数，该函数实现了需求中所提到的要求，该函数最终要返回两个元组比较的大小关系，其实cmp_to_key的实现非常简单

```python
def cmp_to_key(mycmp):
    """Convert a cmp= function into a key= function"""
    class K(object):
        __slots__ = ['obj']
        def __init__(self, obj):
            self.obj = obj
        def __lt__(self, other):
            return mycmp(self.obj, other.obj) < 0
        def __gt__(self, other):
            return mycmp(self.obj, other.obj) > 0
        def __eq__(self, other):
            return mycmp(self.obj, other.obj) == 0
        def __le__(self, other):
            return mycmp(self.obj, other.obj) <= 0
        def __ge__(self, other):
            return mycmp(self.obj, other.obj) >= 0
        __hash__ = None
    return K
```

它在内部定义了一个类K， 并使用我传入的cmp函数完成了比较关系运算符的重载，函数返回的是一个类，而sort函数的key需要的是一个函数，看起来矛盾，但在python中，这样做完全可行，因为类和函数都是callable的，这里把类当成了函数来用。

在本篇第一段代码中

```python
lst.sort(key=lambda item: item[0])
```

lambda表达式生成的匿名函数返回的是元组的第一个元素进行大小比较，而现在，cmp_to_key返回的是类K，参与比较的是K的对象，由于K已经实现了比较关系运算符重载，且算法就是我刚刚实现的cmp函数，这样就最终实现了自定义排序。



# functools.singledispatch

python标准模块functools中的singledispatch是一个装饰器，将普通函数转为泛型函数，这种泛型函数将根据第一个参数类型的不同表现出不同的行为，被装饰的函数是默认实现，如果有其他功能可以使用register进行注册。

## 1. 使用if 语句处理逻辑分支

这段话理解起来有一点费力，我们还是通过具体的案例来说明吧

```python
class Stu(object):
    def __init__(self, name):
        self.name = name

    def wake_up(self):
        print('起床')

class Police(object):
    def __init__(self, name):
        self.name = name

    def wake_up(self):
        print('起床')

stu = Stu('小明')
police = Police('小明爸爸')

def wake_up(obj):
    if isinstance(obj, Stu):
        print('今天周末休息,让孩子们再睡一会')
    elif isinstance(obj, Police):
        print('警察很辛苦,又要起床了')
        obj.wake_up()
    else:
        print('不处理')

wake_up(stu)
wake_up(police)
wake_up('一个字符串')
```

在上面这段代码里，我主要做了一下几件事情

1. 定义了Stu类，Police类
2. 定义了函数wake_up
3. 调用wake_up函数唤醒stu,police，和一个字符串

在函数wake_up里，传入的参数类型是不一样的，函数要根据obj类型的不同来决定自己的行为，假设obj可能有10种类型，那么我就得在wake_up函数里用if ... elif 写10个逻辑分支来处理，这是一件非常痛苦的事情，而singledispatch就是为了解决这个问题而引入的

## 2. 使用singledispatch处理逻辑分支

```python
from functools import singledispatch

class Stu(object):
    def __init__(self, name):
        self.name = name

    def wake_up(self):
        print('起床')

class Police(object):
    def __init__(self, name):
        self.name = name

    def wake_up(self):
        print('起床')


@singledispatch
def wake_up(obj):
    print('不处理')


@wake_up.register(Stu)
def wake_stu(obj):
    print('今天周末休息,让孩子们再睡一会')


@wake_up.register(Police)
def wake_police(obj):
    print('警察很辛苦,又要起床了')
    obj.wake_up()


stu = Stu('小明')
police = Police('小明爸爸')

wake_up(stu)
wake_police(police)
wake_up('一个字符串')
```

wake_up被singledispatch装饰以后，变成了泛型函数，使用@wake_up.register注册一个类型来装饰一个函数，这个函数就专门来处理对应类型的数据。我这里没有注册字符串，当第一个参数是没有被注册的类型时，默认使用wake_up来处理。



# functools.reduce

reduce在2.x中是内置函数，到了python3以后移动到了functools 模块

## 1. reduce一般用法

reduce的作用是从左到右对一个序列的项累计地应用有两个参数的函数，以此合并序列到一个单一值， 下面先来看一个简单示例。

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



# functools.wraps

python标准模块functools提供的wraps函数可以让被装饰器装饰以后的函数保留原有的函数信息，包括函数的名称和函数的注释doc信息。

## 1. 自省信息丢失

函数被装饰以后，一些原本属于自己的自省信息会丢失，先来看装饰前的样子

```python
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```

执行输出结果

```text
test

    测试装饰器
    :param sleep_time:
    :return:
```

test是函数的名字，__doc__是函数的注释说明

但在被普通的装饰器装饰以后，这些信息就会丢失

```python
import time

def cost(func):
    def warpper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper

@cost
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```

程序输出结果

```text
warpper
None
```

这是我们所不希望看到的

## 2. 修复自省信息

wraps可以防止被装饰的函数丢失自己的自省信息，只需要增加@wraps(func)即可

```python
import time
from functools import wraps


def cost(func):
    @wraps(func)
    def warpper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper

@cost
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```



# python内置LRU缓存

python标准模块functools模块中的lru_cache装饰器实现了LRU缓存算法。

## 1. 什么是LRU缓存

在软件或系统开发中，缓存总是必不可少，这是一种空间换时间的技术，通过将频繁访问的数据缓存起来，下一次访问时就可以快速获得期望的结果。

一个缓存系统，关键核心的指标就是缓存命中率，如果命中率很低，那么这个缓存除了浪费了空间，对性能的提升毫无帮助。

LRU是一种常用的缓存算法，即最近最少使用，如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小， LRU算法选择将最近最少使用的数据淘汰，保留那些经常被命中的数据。

## 2. functools.lru_cache

从python 3.2开始，就已经将LRU作为标准库的一部分发布，functools模块中的lru_cache装饰器实现了LRU缓存算法。以计算斐波那契数列为例，对比有缓存的算法效率和无缓存的算法效率

```python
import time
from functools import lru_cache


@lru_cache()        # 测试无缓存时将本行注释掉
def fib_memoization(number: int) -> int:
    if number == 0: return 0
    if number == 1: return 1

    return fib_memoization(number-1) + fib_memoization(number-2)

start = time.time()
res = fib_memoization(33)
print(res)
print(f'耗时: {time.time() - start}s')
```

在有缓存的情况下，计算33的斐波那契数列耗时仅为0.00008秒，去掉装饰器后，耗时则为2.9秒，可谓天差地别。不过要强调一点，之所以有如此巨大的性能差距，只要原因是由于斐波那契算法的特殊性导致的。

## 3. functools.lru_cache参数说明

lru_cache装饰器定义如下

```python
def lru_cache(maxsize=128, typed=False):
    pass
```

只有连个参数，第一个参数规定缓存的数量，第二个参数如果设置为True，则严格检查被装饰函数的参数类型，默认为False

```python
from functools import lru_cache

@lru_cache(typed=False)
def add(x, y):
    print('add')
    return x + y


print(add(3, 4))
print(add(3, 4.0))
```

第二次调用add函数时参数是4.0， 如果你认为这种情况可以使用缓存命中上一次3+4的结果，就将typed设置为False，如果你严格要求只有函数的参数完全一致时才能命中，那么将typed设置为True



# bisect--按排序顺序维护列表

python的bisect模块提供了一种算法，该算法可以将元素插入到列表中，同时保持列表是有序的，如果你希望一个列表在增加元素的过程中始终保持有序，那么使用bisect模块是你的不二选择。

## 1. bisect一般用法

```python
import bisect

values = [23, 1, 54, 34, 56]

lst = []
for item in values:
    index = bisect.bisect(lst, item)        # bisect返回item准备插入的索引位置
    bisect.insort(lst, item)                # insort方法将item插入到lst中的合适位置
    print('{:3}  {:3}'.format(item, index), lst)
```

1. bisect返回元素在插入lst后的索引位置，这个时候还没有进行插入操作，只是预先获知应该插入的位置
2. insort 方法将数据插入到列表的合适位置上

程序输出结果为

```text
 23    0 [23]
  1    0 [1, 23]
 54    2 [1, 23, 54]
 34    2 [1, 23, 34, 54]
 56    4 [1, 23, 34, 54, 56]
```

可以看到，每一次插入数据，列表都保持着有序的特性，虽然你可以在全部插入后进行一次sort排序，但中间过程如果也需要有序的话，使用bisect最为合适。

除了insort方法外，bisect还提供了insort_left方法和insort_right方法，他们用来处理重复数据，插入重复数据时，insort_left会在重复数据的左侧插入，insort_right在重复数据的右侧插入，insort是insort_right的别名。虽然提供了这两种方法，但实在看不出这两个方法有什么特别的用处，毕竟重复数据的位置对于列表的最终结果没有任何影响啊。

## 2. bisect实现原理

其实这种算法的实现原理非常简单，因为列表始终保持有序，因此只需要使用二分查找法，就可以快速定位到需要插入的位置，源码如下

```python
def insort_right(a, x, lo=0, hi=None):
    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo+hi)//2
        if x < a[mid]: hi = mid
        else: lo = mid+1
    a.insert(lo, x)
```



# heapq--适用于列表的最小堆排序算法

heapq模块实现了适用于python列表的最小堆排序算法，该模块提供heappush和heapify两种方法来构建最小堆，heappop方法可以删除堆顶元素，nlargest和nsmallest可以分别查看最大的几个元素和最小的几个元素，关于堆，如果不了解其概念，可以先看下面这段介绍。

## 1.堆的概念

假定在数据记录中存在一个能够标识数据记录的数据项，并可依据该数据项对数据进行组织，则称此数据项为关键码（key）。

如果有一个关键码集合K = {k0 , k1 , k2 , k3 , ... kn-1 } ，把它的所有元素按照完全二叉树的顺序存储方式存放在一个一维数组中，并且满足ki ≤ k2i+1 且 ki ≤ k2i+2 （或者 ki ≥ k2i+1 且 ki ≥ k2i+2） ,i = 0, 1,2, ... (n-2)/2,则称这个集合为最小堆(最大堆)。

在最小堆中，父节点的关键码小于等于它的左右子女的关键码，最大堆中，父节点的关键码大于等于左右子女的关键码

## 2. 构建最小堆

### 2.1 使用heappush方法创建

```python
import heapq

random_lst = [5, 7, 2, 1, 6, 10, 8, 9]
heap_lst = []

for item in random_lst:
    heapq.heappush(heap_lst, item)

print(heap_lst)
```

使用heappush方法，将混乱的数据依次放入到heap_lst中，就会得到一个最小堆，程序输出结果为

```text
[1, 2, 5, 7, 6, 10, 8, 9]
```

如果你不熟悉堆的概念，恐怕难以从结果上看出门道，不妨编写一个输出堆的函数，以更加立体的方式展示堆

```python
import math
from io import StringIO

def print_heap(heap, width=36, fill=' '):
    output = StringIO()
    last_row = -1
    for index, item in enumerate(heap):
        # 计算元素在哪一行
        if index:
            row = int(math.floor(math.log(index + 1, 2)))
        else:
            row = 0

        # 换行操作
        if row != last_row:
            output.write('\n')
        columns = 2 ** row   # 计算一行有多少个数据
        col_width = int(math.floor(width / columns))
        output.write(str(item).center(col_width, fill))
        last_row = row
    print(output.getvalue())


print_heap(heap_lst)
```

程序输出结果为

```text
                 1                  
        2                 5         
    7        6        10       8    
 9  
```

每一个元素的左右子节点都比自己小，这就是最小堆

### 2.2 使用heapify创建最小堆

```python
random_lst = [5, 7, 2, 1, 6, 10, 8, 9]
heapq.heapify(random_lst)
print_heap(random_lst)
```

使用heapify方法可以直接将一个混乱的列表调整为一个最小堆，使用的是向下调整算法。

## 3. 访问堆里的元素

### 3.1 删除堆顶元素

堆顶元素就是列表的第一个元素，删除它是一件很容易的事情，但删除后要立即对堆进行调整，使之维持堆的特性，好在，这些都已经被heappop方法实现。

```python
random_lst = [9, 7, 5, 2, 1, 6, 10, 8]
heapq.heapify(random_lst)
print_heap(random_lst)

print("-"*20)

heapq.heappop(random_lst)
print_heap(random_lst)
```

程序输出结果

```text
                 1                  
        2                 5         
    8        7        6        10   
 9  
--------------------

                 2                  
        7                 5         
    8        9        6        10   
```

### 3.2 替换堆顶元素

heapreplace方法允许使用新的元素替换掉堆顶的元素，这个过程等价于先删除堆顶元素然后将新的元素加入到堆中，先后进行了两次调整

```python
random_lst = [9, 7, 5, 2, 1, 6, 10, 8]
heapq.heapify(random_lst)
print_heap(random_lst)

print("-"*20)

heapq.heapreplace(random_lst, 4)
print_heap(random_lst)
```

程序输出结果

```text
                 1                  
        2                 5         
    8        7        6        10   
 9  
--------------------

                 2                  
        4                 5         
    8        7        6        10   
 9  
```

## 4. Top k问题

在处理大规模数据时，要从海量数据中寻找最大K个数据，或者最小K个数，这类问题被称之为Top k问题。比如现在有10亿个数，从这里面找出最大的20个数。

这么多的数据不可能都载入内存进行排序，使用最小堆就可以完美的解决这个问题。构建一个容量为20的最小堆，先放入20个元素，此后所有的数据都和堆顶的元素进行大小比较，如果比堆顶元素大，则替换堆顶元素，否则不做任何处理，遍历完这10亿个数据后，最小堆里的数据就是最大的20个数， 下面用一段代码模拟这个过程， 从10个数据中寻找最大的3个数

```python
random_lst = [9, 7, 5, 2, 1, 6, 10, 8, 4, 3]
heap_lst = []

for item in random_lst[:3]:
    heapq.heappush(heap_lst, item)

for item in random_lst[3:]:
    if item > heap_lst[0]:
        heapq.heapreplace(heap_lst, item)

print(heap_lst)
```

程序输出结果

```text
[8, 9, 10]
```

对于一个给定的最小堆，nlargest和nsmallest可以分别查看最大的几个元素和最小的几个元素

```python
random_lst = [9, 7, 5, 2, 1, 6, 10, 8, 4, 3]
heapq.heapify(random_lst)

print("最大的3个元素", heapq.nlargest(3, random_lst))
print("最小的3个元素", heapq.nsmallest(3, random_lst))
```

程序输出结果

```text
最大的3个元素 [10, 9, 8]
最小的3个元素 [1, 2, 3]
```

## 5. 合并多个已排序列表

假设有多个以排序好的列表，将他们合并到一个列表中，你可以使用归并排序来处理这个问题，只不多，当需要合并的列表太多时，归并排序就会变的麻烦，毕竟，我们平时接触到归并排序的问题通常是两个列表。

```python
import heapq
import random

lst = []

for i in range(5):
    l = random.sample(range(1, 100), 5)
    l.sort()
    print(l)
    lst.append(l)

print("合并后")
merge = [i for i in heapq.merge(*lst)]
print(merge)
```

程序输出结果

```text
[3, 10, 13, 20, 21, 43, 48, 53, 54, 56, 60, 62, 63, 63, 69, 70, 75, 77, 80, 81, 83, 89, 93, 95, 97]
```



# queue--线程安全的FIFO实现

python的内置模块queue提供了线程安全的队列数据结构，有基本的先进先出的队列Queue， 后进先出的队列LifoQueue和优先级队列PriorityQueue，本文假定你已经对队列这种数据结构有一定了解，如果不是这样，需要你先阅读一些这方面的参考资料，然后再阅读本文

## 1. 基本FIFO队列

first in first out, 先进先出队列，将元素从队列的一端放入，从另一端删除

```python
import queue

q = queue.Queue()

for i in range(10):
    q.put(i)

while not q.empty():
    print(q.get())
```

0是最先放入到队列中的，使用get方法时，第一个取到的数值就是0，当9被取出来时，队列变成了空队列，程序输出结果

```text
0 1 2 3 4 5 6 7 8 9
```

## 2. LIFO队列

与FIFO队列相反，LIFO队列是后进先出，这是一种栈结构

```python
import queue

q = queue.LifoQueue()

for i in range(10):
    q.put(i)

while not q.empty():
    print(q.get())
```

9最后被放入到队列中，第一个被取出来，程序输出结果

```text
9 8 7 6 5 4 3 2 1 0
```

这种数据结构有很广泛的用途，比如下面这个字符串

```text
((sdf)sdf)(sg()())gsg((sgsg()))
```

请写算法判断，字符串里的小括号是否是成对出现，使用后进先出这种数据结构，可以非常轻松的实现算法

```python
import queue

string = '((sdf)sdf)(sg()())gsg((sgsg()))'


def is_pair(string):
    q = queue.LifoQueue()

    for item in string:
        if item == '(':
            q.put(item)
        elif item == ')':
            if q.empty():
                return False
            q.get()

    return q.empty()
```

## 3.优先队列

前面介绍的队列，都是根据进入队列的顺序来决定出队列的顺序，或是先进先出，或是先进后出，除了这两种情况之外，还需要根据优先级来决定出队列的顺序。当有多个任务进入到队列中，根据任务的优先级来决定先执行哪一个，优先级高的先出队列。

```python
import queue


class Task:
    def __init__(self, priority, description):
        self.priority = priority
        self.description = description

    def __eq__(self, other):
        return self.priority == other.priority

    def __lt__(self, other):
        return self.priority < other.priority

    def __str__(self):
        msg = '{_class} 优先级:{priority} 任务:{task}'
        return msg.format(_class=self.__class__, priority=self.priority, task=self.description)

q = queue.PriorityQueue()
q.put(Task(3, '打游戏'))
q.put(Task(5, '做饭'))
q.put(Task(1, '洗碗'))

while not q.empty():
    print(q.get())
```

程序输出结果

```text
<class '__main__.Task'> 优先级:1 任务:洗碗
<class '__main__.Task'> 优先级:3 任务:打游戏
<class '__main__.Task'> 优先级:5 任务:做饭
```
