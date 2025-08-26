---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.17 没有public，protected, private
order: 16
---

# 轻松学会python面向对象第15篇---没有public，protected, private

C++, Java等面向对象编程语言，使用public， protected， private 关键字来控制对类内部成员的访问，python里没有这3个关键字，而是使用下划线实现相似的资源访问控制，其规则如下：

1. 以单下划线做前缀的属性和方法，是受保护的属性和方法，其效果等同于其他语言里的protected
2. 以双下划线做前缀的属性和方法（魔法方法除外），是私有的属性和方法，其效果等同于其他语言里的private
3. 不以下划线开头的属性和方法，类外部可以随意访问，其效果等同其他编程语言里public

### 1. public 属性和方法

准确的说，python中没有公共属性和方法，为了能够和其他面向对象编程语言进行对比学习，固采取此说法

```python
class Dog():
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def print(self):
        print(self.name, self.age)

dog = Dog('花花', 3)
print(self.name)    # 花花
dog.age = 4
dog.print()     # 花花 4
```

实例属性name和age，都不是以下划线开头的，他们是public属性，print方法是public方法，在类的外部，可以访问和修改public属性，可以随意调用public方法。

### 2. protected 属性和方法

在python类中，以单下划线开头的属性和方法是受保护的，但这只是一个约定，你可以直接访问和修改受保护的属性，可直接调用受保护的方法。模块中的函数，如果是以单下划线做前缀，那么它是这个模块受保护的函数，其他模块原则上不应当引用该方法，多数编辑器都会提示你不应当这么做。

```python
class Dog():
    def __init__(self, name, age):
        self._name = name
        self._age = age

    def _print(self):
        print(self._name, self._age)


dog = Dog('花花', 3)
print(dog._name)
dog._age = 4
dog._print()
```

在C++中，如果一个类定义了一个protected属性，子类中，这个属性会自动变为private，这样可以使子类继承父类特定的资源，在python中，不存在这样的机制。

这样看来，python的protected属性和方法似乎很鸡肋，但我认为，这样很符合python一贯坚持的风格。以单下划线做前缀的属性和方法，被视为受保护的，python没有从解释器层面上做出强制性检查，它只是作为一个约定而存在，他们都应被视为API或任何Python代码的非公开部分，无论它是函数，方法还是数据成员。你可以选择遵守这个约定，也可以不遵守，由于不遵守此约定所造成的负面后果自然也需要你来承担。

访问受保护的属性，pycharm会提示你为属性添加property， 这是python提供的保护protected属性的方法。

```python
class Dog():
    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, newname):
        self._name = newname

    def _print(self):
        print(self._name, self._age)


dog = Dog('花花', 3)
print(dog.name)
dog.name = '小花'
dog._print()
print(dog._name)    # 仍然可以访问
```

再次强调，protected属性是受保护的，负责人的工程师不应该去直接修改它。

### 3. private 属性和方法

以双下划线做前缀的属性和方法，是私有的，这意味着你不能在class外部访问它

```python
class Dog():
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    def __print(self):
        print(self.__name, self.__age)


dog = Dog('花花', 3)
print(dog.__name)           # 报错
dog.__name = '小花'          # 报错
dog.__print()                # 报错
```

似乎和C++中的private关键字等效了，但是，从根本上，python没有任何机制来限制你对类成员的访问，包括以双下划线做前缀的属性和方法，它和单下划线一样，仍然只是一个约定, 将对成员访问的方法修改为这种模式：instance._class__variable

```python
dog = Dog('花花', 3)
print(dog._Dog__name)           # 报错
dog._Dog__name = '小花'          # 报错
dog._Dog__print()                # 报错
```

你仍然可以访问和修改私有属性，调用使用方法。

python在属性和方法前面加一个或两个下划线，以模拟protected和private的行为，但这只是一个模拟，一个约定，而不是语言的机制，你总是有办法绕过这些不具备强制性的约定，典型的防君子不防小人。
