---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 11.3 类的封装
order: 2
---

# python面向对象---类的封装

python的面向对象, 并没有严格意义上的私有属性和方法, 私有只是一种约定, 隐藏实现的细节，只对外公开我们想让他们使用的属性和方法，这就叫做封装，封装的目的在于保护类内部数据结构的完整性， 因为使用类的用户无法直接看到类中的数据结构，只能使用类允许公开的数据，很好地避免了外部对内部数据的影响，提高了程序的可维护性。用户只能通过暴露出来的方法访问数据时，你可以在这些方法中加入一些控制逻辑，以实现一些特殊的控制

## 1 私有属性和方法

面向对象的内容，在学习时要侧重其理念，但理念这东西需要实践加持，否则，很难理解。

假设，你和你的同事一起做项目，现在，需要你写一个类让同事使用，这个类，你定义了5个属性，10个方法，但是呢，你的同事其实只会用到其中一个方法，剩下那9个，都是用到的那个方法在执行过程中调用的方法。

那么问题来了，你明确告诉他，你就调用那个A方法就好了，其他方法，你不要使用，不然，可能会影响结果。如果你的同事是个很安分守己的人，他听从了你的建议，老老实实的调用A方法，其他方法，他一概不动，这样就是安全的。

可是过了几天，来了新同事，他偏偏不听话，非得调用剩余的9个方法，它觉得，自己调用那9个方法，可以更好的实现功能，结果呢，出了大问题了，那9个方法，他用的不好，导致程序出错了。

我们写了一个类，有些属性，有些方法，我们不希望被其他人使用，因为那样很容易就产生错误，那么这时，我们就需隐藏实现的细节，只对外公开我们想让他们使用的属性和方法，这就叫做封装。就好比把一些东西用一个盒子封装起来，只留一个口，内部让你看不见。

如何才能做到这一点呢，在python里，如果属性和方法前面是双下划线，那么这个属性和方法就变成了私有的属性。

```python
class Animal:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    def __run(self):
        print("run")

a = Animal('猫', 2)
a.__run()
print(a.__name, a.__age)
```

__run()方法是以双下划线开头的，这样的方法，类的对象无法使用，双下划线开头的属性也同样无法访问，通过这种方法，就可以将不希望外部访问的属性和方法隐藏起来。

属性隐藏起来，并非不让外部使用，而是在使用时收到控制，比如年龄，如果年龄属性定义成age，那么就会出现这样的情况

```python
a.age = 10000
```

你见过哪个动物的年龄有10000岁呢，这显然是有问题的，但age属性暴露出来了，别人在使用时就可能会犯这样的错误，所以，为了防止这样的错误发生，就可以将age属性设置成私有的，然后提供一个set_age方法

```python
class Animal:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    def set_age(self, age):
        if age > 100 or age < 1:
            raise Exception("年龄范围错误")
        self.__age = age

    def get_age(self):
        return self.__age

    def __run(self):
        print("run")

a = Animal('猫', 2)
a.set_age(3)
print(a.get_age())

a.set_age(101)
```

## 2. 类属性

类可以有自己属性，这个属性不同于实例的属性，类属性属于类，但是可以被实例访问

```python
class People(object):
    country = '中国'

print(id(People.country), People.country)       # 通过类去访问

p = People()
# 实例对象p并没有country属性,但会从类属性里找到同名的属性
print(id(p.country), p.country)

p.country = "美国"    # 创建了country实例属性, 而不是修改了类的country属性
print(id(p.country), p.country)
print(id(People.country), People.country)
```

## 3. classmethod

在这之前，所有的示例代码中，类中的方法都是普通方法，第一个参数是self，这种方法只能被实例调用，如果被classmethod装饰，这个方法就是类方法，可以被类和实例调用

```python
class People(object):
    country = '中国'
    @classmethod
    def sing_the_national_anthem(cls):
        print('唱{country}国歌'.format(country=cls.country))


People.sing_the_national_anthem()

p = People()
p.sing_the_national_anthem()
```

如果一个方法不访问示例的属性，但会访问类的属性和方法时，就可以设置成类方法。

## 4. staticmethod

如果一个方法，既不会访问示例属性或方法，也不会访问类的属性或方法，方法里的都是有关逻辑性的代码，那么这种代码就可以设置成静态方法，你可能会好奇，既然这个方法跟类和实例都没啥交互，那要这个方法做什么呢？

回到我们最初描述类的定义上，类提供了一种组合数据和功能的方法，我们定义一个类，是希望这个类能够完成特定的功能，但这里的功能不一定非得访问什么属性啊，它只需要完成我们期望的事情就好了，将这个方法定义在这个类里，是因为它的逻辑和这个类在逻辑功能上有关系，所以，放在一起，便于管理

```python
class People(object):
    country = '中国'

    @staticmethod
    def sing_the_national_anthem():
        print('唱国歌')


People.sing_the_national_anthem()

p = People()
p.sing_the_national_anthem()
```

## 5. 三种类方法对比

| 名称     | 定义方法                                                     | 权限                                               | 调用方法                       |
| -------- | ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------ |
| 实例方法 | 第一个参数必须是示例，一般命名为self                         | 可以访问实例的属性和方法，也可以访问类的实例和方法 | 一般通过示例调用，类也可以调用 |
| 类方法   | 使用装饰器@classmethod修饰，第一个参数必须是当前的类对象，一般命名为cls | 可以访问类的实例和方法                             | 类实例和类都可以调用           |
| 静态方法 | 使用装饰器@staticmethod修饰，参数随意，没有self和cls         | 不可以访问类和实例的属性和方法                     | 实例对象和类对象都可以调用     |
