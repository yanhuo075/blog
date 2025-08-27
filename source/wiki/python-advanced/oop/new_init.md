---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.3 __new__ 和 __init__
order: 2
---

# python面向对象--__new__ 和 __init__

很多人误以为__init__是python对象的构造函数，其实它是初始化函数，真正的构造函数是__new__，构造函数创建对象以后才会调用__init__为实例的属性赋值。下面的代码展示了如何是用__init__函数

```python
class Stu:
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score
        
stu = Stu('小明', 18, 600)
```

__init__不是构造函数的最直接证据便是self，self是实例，这说明实例已经被创建。

一般来说，不建议重写__new__，除非是构建str,int或者tuple不可变类型的子类,比如，我现在定义一个正整数类，如果只在__init__方法上下文章是解决不了问题的

```python
class PositiveInteger(int):
    def __init__(self, value):
        print(type(self), self)
        super(PositiveInteger, self).__init__()

i = PositiveInteger(-5)
print(i)
```

最终输出的结果是

```text
<class '__main__.PositiveInteger'> -5
-5
```

这段代码阅读起来有些苦涩难懂，根源在于你以为继承了int之后，应该可以将value赋值给某个示例属性，当执行print(i)时输出的是i的这个属性。而实际情况是self的值就是-5，并没有某个属性来存储-5。这就意味着我们无法通过重写__init__方法来让对象i的值变为5，在构造i这个对象时，它的值就已经是-5了。为了创建出正整数类，只能重载__new__方法

```python
class PositiveInteger(int):
    def __new__(cls, value):
        return super(PositiveInteger, cls).__new__(cls, abs(value))

i = PositiveInteger(-5)
print(i)
```

构造PositiveInteger实例时，将参数的值修改为abs(value)，这样就可以确保得到的是正整数。
