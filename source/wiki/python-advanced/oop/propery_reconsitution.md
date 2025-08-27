---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.8 用propery重构代码
order: 7
---

# python面向对象--利用property重构代码

本文从一个简单的例子开始，为你讲解property类的作用

```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width
        self.area = length * width

rectangle = Rectangle(5, 2)

print(rectangle.length)
print(rectangle.width)
print(rectangle.area)
```

上面这段代码定义了一个Rectangle类，创建对象时初始化参数为length和width。创建好对象以后，三个print语句分别输出长宽和面积。

乍看上去，这段代码没有什么问题，但稍微仔细推敲一下就会发现一个严重的问题，area属性被暴露出来了，这导致你可以使用下面的语句对这个属性进行修改

```python
rectangle.area = 20
```

长方形的面积被修改为20，但面积本应该由长和宽来决定，不应该被随意修改，为了避免面积被修改，一个可能的解决办法是修改类的定义

```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def get_area(self):
        return self.length * self.width
```

之所以说这是一个可能的方法，是因为这样修改可能会带来一些麻烦，如果系统里已经有很多处代码使用 rectangle.area 这种方式获取面积，那么你需要将所有这种代码都修改为rectangle.get_area()，这绝对是一个糟糕的修改过程。

面对这种情况，你可以使用@property修饰符，一个类里定义的方法一旦被@property修饰，你可以像使用属性一样去使用这个方法

```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    @property
    def area(self):
        return self.length * self.width

rectangle = Rectangle(5, 2)

print(rectangle.length)
print(rectangle.width)
print(rectangle.area)
```

area 虽然是一个方法，但是你可以把它当成属性来使用，调用时不需要写后面的小括号，而且不必担心有人对其进行赋值操作，rectangle.area = 20 这种代码一定会报错的。

用@property重构了属性area，之前使用的rectangle.area代码依然可以使用，不需要进行任何修改，这就是property的强大之处，兵不血刃的重构了类的定义却不影响外部的使用。
