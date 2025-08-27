---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.13 python的魔法方法
order: 12
---

# python的魔法方法

python中的魔法方法是一些可以让你对类添加“魔法”的特殊方法，它们经常是两个下划线包围来命名的，比如 __init__, __call__。魔法方法可以使Python的自由度变得更高，在面向对象方面表现的更好。

python有很多魔法方法，根据其作用和特点，归纳总结为以下几类

## 1. 基础魔法方法

### 1.1 __new__

__new__ 是真正的构造函数，在实例化对象时被调用，通常你不会实现这个方法，除非你想对实例的创建进行控制，比如利用__new__ 写一个单例模式，参见文章[python实现单例模式的5种方法](http://www.coolpython.net/informal_essay/20-09/five-methods-to-achieve-singleton.html#toc_4)

### 1.2 __init__

__init__ 被用于初始化实例对象，__new__方法创建对象后就会调用该方法对实例的属性进行初始化。很多人误以为__init__是构造函数，这是不对的，面试的时候经常拿他们两个做比较。

### 1.3 __del__

__del__ 是析构方法，当对象的引用计数变为0时，这个对象会被销毁掉，此时就会调用__del__方法

```python
class T:
    def __del__(self):
        print("对象被销毁")

t = T()
t = 0  # t指向0， 此前创建的T的实例被销毁
print('ok')
```

程序输出结果

```text
对象被销毁
ok
```

当变量t重新赋值为0时，此前创建的T的实例由于引用计数变为了0，对象被销毁，__del__ 被调用，而后执行print('ok')语句。

### 1.4 __call__

允许一个类的实例像函数一样被调用，这是一个非常重要的特性，与他相关的python概念还有callbale，参见文章[python callable概念](http://www.coolpython.net/python_senior/senior_feature/callable.html)

下面的示例代码向你展示如何使用__call__

```python
class T:
    def __call__(self, *args, **kwargs):
        print("类的实例可以像函数一样被调用")

t = T()
t()
```

### 1.5 __len__

```python
class T:
    def __len__(self):
        return 100

t = T()
print(len(t))
```

len是内置函数，len(t) 会调用实例的__len__方法

### 1.6 __str__

当被 str() 调用时, 调用该方法

```python
class T:
    pass

t = T()

print(str(t))   # <__main__.T object at 0x000001FB39A9A518>
```

如果你没有实现__str__方法，那么会调用默认的__str__，以固定格式返回对象的描述，下面是一个重载后的示例

```python
class T:
    def __str__(self):
        return f"我是{self.__class__.__name__}的一个实例"

t = T()

print(str(t))   # 我是T的一个实例
```

### 1.7 __hash__

当被 hash() 调用时, 执行__hash__， 很少有机会来实现这个魔法方法。

### 1.8 __bool__

```python
class T:
    def __init__(self, age):
        self.age = age

    def __bool__(self):
        return self.age > 18

print(bool(T(17)), bool(T(19)))     # False True
```

### 1.9 __format__

__format__ 在格式化字符串方面是非常有用的

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __format__(self, code):
        return 'x: {x}, y: {y}'.format(x = self.x, y = self.y)

point = Point(3, 4)
print("this point is {p}".format(p=point))  # this point is x: 3, y: 4
```

## 2. 属性相关的方法

### 2.1 __getattr__

实例的属性存储在__dict__中，如果访问的属性不存在，就会调用__getattr__方法，如果没有实现这个方法，则会抛出AttributeError异常。

```python
class Point:
    def __init__(self, data):
        self.data = data

    def __getattr__(self, item):
        return self.data.get(item)


point = Point({'x': 3, 'y': 4})
print(point.x)      # 3
print(point.y)      # 4
print(point.z)      # None
```

point实例并不存在x,y,z这三个属性，因此都会调用__getattr__来返回数据，可以利用这个机制来返回属性

### 2.2 __getattribute__

访问实例的属性时，不论属性是否存在，都将会调用__getattribute__方法

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __getattribute__(self, item):
        if item == 'x':
            return object.__getattribute__(self, item)
        else:
            raise AttributeError(f'不能访问{item}')


point = Point(3, 4)
print(point.x)      # 3
print(point.y)      # 异常
```

假设你只希望x属性可以被访问，y属性无法被访问，那么你可以通过__getattribute__来达到目的

### 2.3 __setattr__

对实例的属性进行赋值时，调用__setattr__方法

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __setattr__(self, key, value):
        if key == 'x':
            value = abs(value)
        object.__setattr__(self, key, value)


point = Point(3, 4)
point.x = -9
print(point.x)      # 9
```

如果对x进行赋值，不论赋给它的值是多少，都要转成正整数。

### 2.4 __delattr__

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __delattr__(self, item):
        print(f"{item}属性被删除了")
        object.__delattr__(self, item)

point = Point(3, 4)
del point.x     # 删除x属性
print(point.x)      # 引发异常
```

### 2.5 __dir__

当被dir函数调用时，实例的__dir__方法被执行，默认会返回实例所拥有的属性和方法，一般来说，没有必要实现这个方法

### 2.6 __set__, __get__ , __delete__

这三个方法放在一起讲是因为他们都涉及到描述器的概念，如果一个类定义了三者中的一个方法，那么这个类的实例就是描述器。

```python
class Coord():
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, cls):
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, (int, float)):
            try:
                value = int(value)
            except:
                try:
                    value = float(value)
                except:
                    raise AttributeError(f"{value}不是坐标值")

        instance.__dict__[self.name] = value

class Point:
    x = Coord('x')
    y = Coord('y')

    def __init__(self, x, y):
        self.x = x
        self.y = y


point = Point('3', 4)
print(point.x)      # 3

point.y = 'k'       # 引发异常
```

在这个实例中，x, y 是类Point的类属性，且他们都是描述器，描述器是一种代理机制，当对一个描述器赋值时

```python
self.x = 3
```

根据描述器的协议，上面的代码等价于

```text
Coord.__set__(x, self, 3)
```

这样就实现了对属性赋值的检查。

### 2.7 __enter__() 和__exit__

如果一个对象实现了__enter__() 和__exit__()， 那么它就是一个上下文管理器， 上下文管理在python中有着非常重要的位置，你平时使用with语句打开一个文件时，用的正是上下文管理。

通常__enter__() 会返回对象管理器本身，在进入with语句块执行代码之前，先要执行__enter__()， 从with语句块退出时执行__exit__()，做一些清理工作。

关于上下文管理器，请参考文章[python with 语句](http://www.coolpython.net/python_senior/senior_feature/with.html)

## 3. 比较操作符

比较操作符包括：

1. __lt__(self, other)定义小于号的行为：x < y
2. __le__(self, other)定义小于等于号的行为：x <= y
3. __eq__(self, other)定义等于号的行为：x == y
4. __ne__(self, other)定义不等号的行为：x != y
5. __gt__(self, other)定义大于号的行为：x > y
6. __ge__(self, other)定义大于等于号的行为：x >= y

下面的示例演示__lt__ 和 __eq__方法

```python
from math import sqrt

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __lt__(self, other):
        return sqrt(self.x**2 + self.y**2) < sqrt(other.x**2 + other.y**2)

    def __eq__(self, other):
        return sqrt(self.x ** 2 + self.y ** 2) == sqrt(other.x ** 2 + other.y ** 2)

point1 = Point(3, 4)
point2 = Point(4, 3)
point3 = Point(3.5, 3.5)

print(point1 < point3)
print(point1 == point2)
```

## 4. 容器类型

### 4.1 __getitem__， __setitem__, __delitem__

__getitem__定义获取容器中指定元素的行为，相当于 self[key],__setitem__定义设置容器中指定元素的行为，相当于 self[key] = value,__delitem__)定义删除容器中指定元素的行为，相当于 del self[key]。

你应该已经注意到，这种用法和字典非常相似，实际上字典就是实现了这三个魔法方法，才提供了以[]为基础的操作。

```python
class MyData():
    def __init__(self, data={}):
        self.data = data

    def __getitem__(self, item):
        return self.data.get(item)

    def __setitem__(self, key, value):
        self.data[key] = value

    def __delitem__(self, key):
        del self.data[key]


my_data = MyData()
my_data['name'] = '小刚'
my_data['age'] = 14

print(my_data['name'])      # 小刚
print(my_data['age'])       # 14
```

### 4.2 __iter__， __next__

如果一个对象实现了__iter__， 那么它就是一个可迭代对象，如果既实现__iter__ 又 实现__next__，那么它就是一个迭代器。

```python
from collections import Iterable, Iterator


class Color(object):

    def __init__(self):
        self.index = -1
        self.colors = ['red', 'white', 'black', 'green']

    # 返回对象本身
    def __iter__(self):
        self.index = -1
        return self

    def __next__(self):
        self.index += 1
        if self.index >= 4:
            raise StopIteration
        return self.colors[self.index]

color_object = Color()
# 判断是否为可迭代对象
print(isinstance(color_object, Iterable))       # True
# 判断是否为迭代器
print(isinstance(color_object, Iterator))       # True

# 遍历输出所有颜色
for color in color_object:
    print(color)
```
