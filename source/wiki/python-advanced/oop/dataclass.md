---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.14 理解python数据类
order: 13
---

# 理解python数据类--dataclass

dataclass 是dataclasses模块提供的一个装饰器，当一个普通的类被dataclass装饰时，这个普通的类将被赋予一些神奇的魔法，这对于我们处理数据是十分有帮助的。dataclass所装饰的类仍然属于面向对象范畴，它并没有发明新的轮子，而是对旧轮子进行了改造升级，让开发人员在处理数据时更便捷。

通过与普通类进行比较，我们能非常清晰的看到dataclass所带来的优势。

## 1. 自动实现__init__方法

这里，我要定义一个Cat类，先来看使用普通类如何进行

```python
class Cat():
    def __init__(self, name, age):
        self.name = name
        self.age = age


cat = Cat('花花', 3)
print(cat.name, cat.age)        # 花花 3
```

这是一段极其普通的代码，那些具备实际工作经验的人对于__init__方法多少会有一些反感，这种反感来源于当一个类有多大十几个属性时的机械操作。你不得不在__init__方法里定义十几个参数，同时在方法内部定义十几个实例属性，这种编程的体验十分糟糕。

dataclass的出现，使得这一状况得到改善，dataclass装饰器会自动生成__init__方法，实现对实例属性的赋值，下面是使用示例

```python
from dataclasses import dataclass, asdict

@dataclass
class Cat():
    name: str
    age: int = 1


cat = Cat('花花', 3)
print(cat.name, cat.age)        # 花花 3
```

被dataclass装饰的类，可以通过类型标注的形式声明属性，属性的类型一目了然，这对于提高代码的可阅读性十分有用。虽然我没有实现__init__方法，但可以像普通类那样访问实例属性，这一切都是dataclass的功劳。当一个类又有数十个属性时，使用dataclass装饰器能大大减小代码编写的繁琐程度。

## 2. 自动重载比较运算符

dataclasses这个模块专门为数据处理而生，dataclass装饰器只有在数据处理的场景下才能大显身手。如果你对ORM有一定了解，你应该知道顶一个Model时，一个类对应数据库里的一张表，一个类属性对应表里的一个字段，这样的类，专门为处理数据而生，它自身没有太多的方法，能够灵活高效的表达数据即可。

这样专门处理数据的类，我们使用dataclass进行装饰再好不过了，它会自动实现重载比较运算符。

```python
from dataclasses import dataclass, asdict

@dataclass(order=True)
class Number():
    var: int

n1 = Number(3)
n2 = Number(4)

print(n1 > n2)
```

如果这一个普通的Number类，它的实例对象在比较大小时并不会根据自身的属性进行比较，但被dataclass装饰过就可以，这也带来了一个新的问题，如果类有多个属性，大小关系如何比较呢，答案是根据属性定义的顺序逐一比较直至分出大小。

回到上一小节定义的Cat类

```python
from dataclasses import dataclass, asdict

@dataclass(order=True)
class Cat():
    name: str
    age: int = 1


cat1 = Cat('花花1', 3)
cat2 = Cat('花花2', 2)

print(cat1 > cat2)      # False
```

cat1 与 cat2 进行比较时，先使用name进行比较，直接分出胜负，'花花1' < '花花2'， 可名字怎么能用来比较大小呢，应该用age才对，但dataclass装饰器目前还不能提供这样的能力。

我的看法是，自动重载比较运算符是一个比较鸡肋的功能，也正因如此，order参数默认是False，也就是不会自动重载比较运算符，讲到这里，我们有必要了解一下dataclass定义

```python
@dataclasses.dataclass(*, init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)
```

init, repr, eq 等于True，表示dataclass会自动为被装饰的类生成对应的魔法方法，这里对比较运算符做了特别处理，拆分成eq 和 order两部分，正如前面所言，比较大小时根据字段定义顺序逐一比较非常鸡肋，但是在比较相等时就再正常不过，因此eq方法默认是自动实现的。

## 3. 进制属性被修改

对于一个普通类的实例对象，我们没有办法保证它的属性不被修改，但实践中，我们又确实存在这样的需求：一个实例对象一旦被创建，它的属性就不允许被修改。这一特性，使用dataclass就非常方便

```python
from dataclasses import dataclass, asdict

@dataclass(frozen=True)
class Cat():
    name: str
    age: int = 1


cat = Cat('花花1', 3)
cat.age = 4
```

设置装饰器参数frozen为True，实例对象的属性就无法被修改，最后一行代码会引发异常

```text
dataclasses.FrozenInstanceError: cannot assign to field 'age'
```

## 4. 实例对象转json

这个功能可谓贴心至极，是我多年梦寐以求的功能，直接将python对象转成字典，属性做key，属性值做value，可以进一步讲字典转成json数据，这对于专门处理数据的程序而言实在太方便了。

```python
from dataclasses import dataclass, asdict

@dataclass(frozen=True)
class Cat():
    name: str
    age: int = 1


cat = Cat('花花1', 3)

print(asdict(cat))      # {'name': '花花1', 'age': 3}
```

使用asdict函数，可将cat对象转成字典对象，普通类的实例，也可以实现，但要开发人员自己遍历实例的属性进行处理，asdict的出现算是在官方层面上提供了这个大家渴望已久的功能。

## 5. 详细的数据展示

dataclass装饰器的设计初衷，就是为了处理数据时方便，使用print输出普通的示例对象，我们往往得到的是下面的结果

```python
class Cat():
    def __init__(self, name, age):
        self.name = name
        self.age = age


cat = Cat('花花', 3)
print(cat)      # <__main__.Cat object at 0x10a810d30>
```

除了内存地址，<__main__.Cat object at 0x10a810d30> 不能为我们带来更多的信息，我们更想知道对象属性的情况，诚然，你可以通过实现repr方法来实现，但dataclass可以为你节省开发工作量

```python
from dataclasses import dataclass, asdict

@dataclass
class Cat():
    name: str
    age: int = 1


cat = Cat('花花1', 3)
print(cat)  # Cat(name='花花1', age=3)
```

## 6. 总结

如果一个类，主要的任务是处理数据，有许多的属性需要存储和处理，那么使用dataclass装饰器将大大提高你的开发效率，提高代码的可阅读性。
