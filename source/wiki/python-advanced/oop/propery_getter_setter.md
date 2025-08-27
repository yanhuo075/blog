---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.7 用propery代替getter和setter方法
order: 6
---

# python面向对象--用propery代替getter和setter方法

封装是面向对象三大特性之一，下面是一个符合封装特性的类定义

```python
class Book:
    def __init__(self, name, price):
        self._name = name
        self._price = price

    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price):
        self._price = price

book = Book('python进阶', 58.5)
print(book.get_name())
print(book.get_price())

book.set_price(50.0)
print(book.get_price())
```

上面这种代码，在C++中非常的常见，这种定义类的初衷概括为两点:

1. 确保属性是私有的，不可随意修改
2. 提供一个获取属性的getter方法和一个修改属性的setter方法

这种做法的好处在于，属性是私有的，外部无法直接修改和方法，提供的setter方法可以对新的值做检查，比如set_price方法里可以判断传入的price参数的值是否大于0，小于等于0的数肯定是有问题的。如果直接将属性暴露出来，你无法约束使用者合理的使用这个类。

我最早转型python时，就喜欢这样定义类，但逐渐发现，python提供的property可以让我们以一种更加优雅的方式定义类

```python
class Book:
    def __init__(self, name, price):
        self._name = name
        self._price = price

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, name):
        self._name = name

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, price):
        if price > 0:
            self._price = price
        else:
            raise 'price must bigger than zero'

book = Book('python进阶', 58.5)
print(book.name)
print(book.price)

book.price = 50.0
print(book.price)
```

使用@property装饰器修饰price方法后，你就可以像使用属性一样使用price，如果你希望可以对price进行赋值，那么需要用@price.setter装饰器再装饰一个方法，该方法完成对_price属性的赋值操作。

我们以为自己直接操作了对象的属性，但其实我们是在使用类的方法，而且关键的是省去了调用方法时的那对小括号。
