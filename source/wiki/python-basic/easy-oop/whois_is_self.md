---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.12 灵魂拷问，self是谁
order: 11
---

# 轻松学会python面向对象第11篇---灵魂拷问，self是谁

![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823235705356.jpeg)

```text
class Dog():
    def __init__(self, _name, _age):
        self.name = _name
        self.age = _age

    def eat_moon(self):
        print(f"{self.name}吃月")
```

在python类里定义方法时，总是每个方法都有一个self参数，而且如果你的编辑器足够智能，self是被自动添加的，不需要你动手。

那么这个self究竟是谁呢？如果你强行删除掉这个参数，程序就会报错，代码里出现红色的错误提示：方法必须有一个参数，通常叫self。

关于这个self，我教大家一个简单的方法来识别它---谁调用该方法，self就是谁，下面用代码来举例子

```python
dog1 = Dog('二哈', 4)
dog1.eat_moon()     # 二哈吃月
```

代码里，是对象实例dog1调用了eat_moon，那么dog1就是eat_moon方法里的self。print输出的结果就是二哈吃月。

```python
dog2 = Dog('老哈', 8)
dog2.eat_moon()     # 老哈吃月
```

我又创建了一个dog2对象，dog2对象调用eat_moon方法，那么在eat_moon执行过程中，传入的self参数就是dog2，dog2.name的值是老哈。

在第10篇教程里，我已经解释了，dog1.eat_moon和dog2.eat_moon的内存地址是相同的

```text
print(id(dog1.eat_moon) == id(dog2.eat_moon))  # True
```

他们都是类里所定义的eat_moon函数的绑定，因此两次调用，执行的都是同一个方法，print语句输出的结果不一样，是因为每次调用时，传入的self不同，谁调用，self就是谁。
