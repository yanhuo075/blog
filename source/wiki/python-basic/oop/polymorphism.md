---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 11.5 类的多态
order: 4
---

# python面向对象---类的多态

python面向对象的多态依赖于继承, 因为继承，使得子类拥有了父类的方法, 子类的方法与父类方法重名时是重写, 同一类事物，有多重形态, 这就是面向对象概念里的多态，多态使得不同的子类对象调用相同的 类方法，产生不同的执行结果，可以增加代码的外部调用灵活度。

## 1. 多态

### 1.1 重写

多态这个概念依赖于继承，因为继承，使得子类拥有了父类的方法，这里就产生了一个问题，如果子类有一个方法和父类的方法同名，那么子类在调用这个方法时，究竟是调用子类自己的方法，还是父类的方法？

```python
class Base():
    def print(self):
        print("base")


class A(Base):
    def print(self):
        print("A")


a = A()
a.print()
```

父类和子类都有print方法，那么子类A的对象a调用print方法时，调用的是谁的print方法呢？

答案是子类的print方法，如果A类没有定义print方法，那么a.print()调用的是父类的print方法，但是A类定义了print方法，这种情况称之为重写，A类重写了父类的print方法。

### 1.2 继承的搜索

强调继承时，子类“拥有”父类的方法和属性，特意加了双引号，因为，这种拥有不是真实意义上的拥有。

```python
class Base():
    def print(self):
        print("base")


class A(Base):
    pass


print(id(Base.print))
print(id(A.print))
```

Base.print 和 A.print 的内存地址是相同的，这说明他们是同一个方法。执行A.print时，python会寻找print方法，它会先从A类的定义中寻找，没有找到，然后去A的父类里寻找print方法，如果还是找不到，就继续向上寻找。

这便是子类拥有父类属性和方法的真相

### 1.3 多态的表现形式

同一类事物，有多重形态

```python
class Animal:
    def run(self):
        raise NotImplementedError

class People(Animal):
    def run(self):
        print("人在行走")


class Pig(Animal):
    def run(self):
        print("猪在跑")


p1 = People()
p1.run()

p2 = Pig()
p2.run()
```

People和 Pig 都继承了Animal，都是动物，是同一类事物，他们都有run方法，但是最终的运行结果却不一样，这就是多态，同一类事物有多种形态。
