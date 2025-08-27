---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.15 子类的定义
order: 14
---

# 用__init_subclass__ 检查python 子类的定义

__init_subclass__ 是python面向对象编程里的一个魔法方法，它在初始化子类时被调用，可用于检查子类的定义是否符合要求，比如要求子类必须实现某个方法

```python
class Base():
    def abc_func(self):
        raise NotImplementedError


class Test(Base):
    def __init__(self, name):
        self.name = name
```

在基类Base 里定义了一个抽象方法abc_func， 这里我为了演示__init_subclass__用法，没有使用abc 模块定义类的抽象方法，Test作为子类没有重写父类的abc_func方法，但并不会报错。当Test类的示例对象调用abc_func方法时，实际调用的是Base 的abc_func方法，此时会抛出NotImplementedError 异常。

如果想在类定义阶段就能发现问题，一种方法是使用元类技术，在子类创建阶段就找出问题，第二种方法是实现基类的__init_subclass__，在子类初始化阶段找出问题。

```python
class Base():

    def __init_subclass__(cls, **kwargs):
        if getattr(cls, 'abc_func') is Base.abc_func:
            raise Exception(f"子类{cls.__name__}没有实现abc_func方法")

    def abc_func(self):
        raise NotImplementedError


class Test(Base):
    def __init__(self, name):
        self.name = name
```

相比于使用元类技术，使用__init_subclass__ 更加简单，更容易理解，__init_subclass__ 被调用时，已经完成了子类的创建，对子类的类属性和方法都可以进行检查，在基类里实现了对子类定义的约束。
