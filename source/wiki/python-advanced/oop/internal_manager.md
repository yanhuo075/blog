---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.11 类的内部管理
order: 10
---

# 深入理解python面向对象中类的内部管理

本文将向你解释python如何存储属性以及如何查找这些属性，阅读本文时，要求你已经对面向对象有了一定的了解

## 1. 一个简单的类

我定义了一个简单的类

```python
class AirVehicle(object):
    category = '飞机'

    def __init__(self, name):
        self.name = name

    def fly(self):
        print('{name} is flying'.format(name=self.name))
```

name是实例属性，而category是类属性，在使用过程中，很容易由于理解不清导致一些奇怪的问题

### 1.1 通过类修改category

```python
p1 = AirVehicle('歼20')
p2 = AirVehicle('歼15')

print(p1.category, p1.category)
AirVehicle.category = '战斗机'
print(p1.category, p2.category)
```

程序输出结果

```text
飞机 飞机
战斗机 战斗机
```

通过类修改category以后，对所有实例都生效了，那么通过实例进行修改是否也可以呢？

### 1.2 通过实例修改category

```python
p1 = AirVehicle('歼20')
p2 = AirVehicle('歼15')

print(p1.category, p1.category)
p1.category = '战斗机'
print(p1.category, p2.category)
```

程序输出结果

```python
飞机 飞机
战斗机 飞机
```

通过实例p1修改category属性，只对p1生效了，p2的category属性仍然是飞机，想要搞清楚这种差别，就需要探究python是如何存储实例属性和类属性的

## 2. python如何存储实例属性

python中，实例属性存储在一个字典(__dict__)中，对于属性的操作，都是在操作这个字典

```python
p1 = AirVehicle('歼20')
print(p1.__dict__)
```

程序输出结果

```text
{'name': '歼20'}
```

你甚至可以直接操作这个字典

```python
p1 = AirVehicle('歼20')
p1.__dict__['speed'] = '2马赫'
print(p1.speed)   # 2马赫
```

## 3. python如何存储类属性

同样是存储在字典中，只是这个字典是类的字典

```python
print(AirVehicle.__dict__)
```

程序输出结果

```text
{'__module__': '__main__', 'category': '飞机', '__init__': <function AirVehicle.__init__ at 0x103eefd90>, 
'fly': <function AirVehicle.fly at 0x103f0d268>, '__dict__': <attribute '__dict__' of 'AirVehicle' objects>, '__weakref__': <attribute '__weakref__' of 'AirVehicle' objects>, '__doc__': None}
```

这次输出的信息比较多，我们只关注category和fly，类的属性和方法存在在类的__dict__之中

## 4. 属性寻找规则

在实例p1中，是没有category这个属性的，当解释器在p1的__dict__找不到category时，就会去类AirVehicle的__dict__查找，找到后返回。

当你执行p1.category = '战斗机'时，修改的是p1的__dict__， 而类AirVehicle的__dict__则并没有被修改，因此再次执行print(p1.category, p2.category)时，p1的category是战斗机， 而p2的category还是飞机，因为p2的__dict__中仍然没有category这个key，还是要到AirVehicle中寻找。
