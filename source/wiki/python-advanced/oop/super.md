---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.4 super()
order: 3
---

# python面向对象--super()

对于单继承，直接用类名调用父类方法是没有问题的，但对于多继承， 会涉及到查找顺序（MRO）、重复调用（菱形继承）等问题。为了解决python菱形继承导致基类方法重复调用的问题，引入了super()， super() 函数可用于调用父类(超类)的一个方法

## 1. 普通继承

子类中调用父类方法并不难，下面是一个简单示例

```python
class A:
    def __init__(self):
        self.attr_a = 1
        print('执行A的初始化函数')


class B(A):
    def __init__(self):
        A.__init__(self)
        self.attr_b = 2

b = B()
print(b.attr_a, b.attr_b)
```

程序输出结果

```text
执行A的初始化函数
1 2
```

如果不执行A.__init__(self)， 对象b将没有attr_a属性， 这显然与预期不符， 既然是继承，那么子类就应当继承父类的属性与方法。

## 2. 菱形继承

在子类中直接调用父类的方法虽然可行，但在特定场景下会有问题，比如菱形继承

```python
class A:
    def __init__(self):
        self.attr_a = 1
        print('执行A的初始化函数')


class B(A):
    def __init__(self):
        A.__init__(self)
        self.attr_b = 2
        print('执行B的初始化函数')

class C(A):
    def __init__(self):
        A.__init__(self)
        self.attr_c = 3
        print('执行C的初始化函数')

class D(B, C):
    def __init__(self):
        B.__init__(self)
        C.__init__(self)
        self.attr_d = 4
        print('执行D的初始化函数')


d = D()
print(d.attr_a, d.attr_b, d.attr_c, d.attr_d)
```

A是B和C的父类，D同时继承了B和C， 这样就形成了菱形继承，所谓菱形继承，仅仅是因形状上像菱形。程序执行结果

```text
执行A的初始化函数
执行B的初始化函数
执行A的初始化函数
执行C的初始化函数
执行D的初始化函数
1 2 3 4
```

你应该注意到，类A的初始化函数被执行了两次，这是一个非常危险的行为，如果A的初始化函数执行了一些一个进程中只能执行一次的代码，这样的多进程就会导致严重的问题， super的引入就是为了解决这种问题。

```python
class A:
    def __init__(self):
        self.attr_a = 1
        print('执行A的初始化函数')


class B(A):
    def __init__(self):
        super().__init__()
        self.attr_b = 2
        print('执行B的初始化函数')

class C(A):
    def __init__(self):
        super().__init__()
        self.attr_c = 3
        print('执行C的初始化函数')

class D(B, C):
    def __init__(self):
        super().__init__()
        self.attr_d = 4
        print('执行D的初始化函数')


d = D()
print(d.attr_a, d.attr_b, d.attr_c, d.attr_d)
```

程序执行结果

```text
执行A的初始化函数
执行C的初始化函数
执行B的初始化函数
执行D的初始化函数
1 2 3 4
```

在D的初始化函数中，只使用了一行代码super().__init__()， 就将两个父类B和C的初始化函数都执行了， 而且不会重复执行A的初始化函数，这些都是super帮助我们完成的。

## 3. MRO

在第2小结的示例中， 执行D的初始化函数，使用了super，会自动执行B，C的初始化函数，那么B与C的初始化函数先执行哪个呢？对于这个问题，在我们定义类时，python会计算出一个方法解析顺序列表，也就是MRO，这个MRO列表就是一个简单的所有基类的线性顺序表, 类的mro()方法可以获得MRO

```python
print(D.mro())
```

输出结果

```text
[<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
```

按照顺序，先执行B的初始化函数，在执行C的初始化函数，最后执行A的初始化函数。

## 4. 带参数的super方法使用示例

为了简化代码，前面的示例代码在执行super方法时，都没有使用任何参数，下面提供了有参数的示例

```python
class A:
    def __init__(self):
        self.attr_a = 1

    def add(self, a, b):
        return a + b

class B(A):
    def __init__(self):
        super().__init__()
        self.attr_b = 2

    def add(self, a, b):
        print('执行B的add')
        return a + b + 1


class C(A):
    def __init__(self):
        super().__init__()
        self.attr_c = 3

    def add(self, a, b):
        print('执行C的add')
        return a + b + 2

class D(B, C):
    def __init__(self):
        super().__init__()
        self.attr_d = 4

    def add(self, a, b):
        return super().add(a, b)

d = D()
print(d.add(1, 2))
```

程序执行结果

```text
执行B的add
4
```

D的父类B和C都有add方法，在D的add方法时，super().add()会根据mro来决定调用哪个父类的add方法，根据顺序，应该执行B的add方法， 如果你希望执行C的add方法, 那么可以这样来实现add方法

```python
def add(self, a, b):
    return super(B, self).add(a, b)
```

在mro列表里，B的后面是C， super的第一个函数指定为B， 第二个参数设置为self，就会执行C的add方法。
