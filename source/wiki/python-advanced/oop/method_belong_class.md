---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.2 类方法/实例属性
order: 1
---

# python面向对象--方法属于类，属性属于实例

不只是python如此，大多数面向对象语言都符合这个不成文的理论，方法属于类，属性属于实例

每当你看到实例两个字时，你都应该意识到内存中存在一个实例对象，实例属性也随之存在，当实例被销毁时，属性也随之被销毁。实例属性必须依附于实例。实例方法则不然，不论实例是否存在，实例方法都客观存在，实例方法是在定义类的时候创建的，只要这个类在，那么这些实例方法就存在。

类的存在，是一种约定，约定了实例有哪些属性，约定了实例可以调用的函数（方法）。

```python
class Stu:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def run(self):
        print("{name} is running".format(name=self.name))

    def print(self):
        print("ok")

Stu.print(None)
s = Stu("小刚", 18)
Stu.run(s)
```

通过类调用类里所定义的print方法，我这里直接传入None,在执行过程中，self就是None,即便如此，这个方法也是可以被正常执行的，因为在函数里没有使用实例属性，如果使用了呢？就会报错，self是None，不是类Stu的实例，因此没有实例属性。

Stu这个类可以直接调用方法，因为方法是属于类的，但属性属于对象，你无法通过类直接访问对象的属性，下面的写法是错误的

```python
class Stu:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def run(self):
        print("{name} is running".format(name=self.name))

    def print(self):
        print("ok")

print(Stu.name)
```

原因在于，对象还没有被创建，那么name，age根本就不存在，即便是创建了，name和age属性，也是属于对象的。
