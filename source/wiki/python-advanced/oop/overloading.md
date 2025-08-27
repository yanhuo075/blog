---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.9 运算符重载
order: 8
---

# python面向对象--运算符重载

下表是常见的运算符重载方法说明

| 方法名                                       | 重载说明           | 运算符调用方式                                               |
| -------------------------------------------- | ------------------ | ------------------------------------------------------------ |
| __init__                                     | 构造函数           | 对象创建: X = Class(args)                                    |
| __del__                                      | 析构函数           | X对象收回                                                    |
| __add__/__sub__                              | 加减运算           | X+Y， X+=Y/X-Y， X-=Y                                        |
| __or__                                       | 运算符\|           | X\|Y, X\|=Y                                                  |
| _repr__／__str__                             | 打印／转换         | print(X)、repr(X)／str(X)                                    |
| __call__                                     | 函数调用           | X(*args, **kwargs)                                           |
| __getattr__                                  | 属性引用           | X.undefined                                                  |
| __setattr__                                  | 属性赋值           | X.any=value                                                  |
| __delattr__                                  | 属性删除           | del X.any                                                    |
| __getattribute__                             | 属性获取           | X.any                                                        |
| __getitem__                                  | 索引运算           | X[key]，X[i:j]                                               |
| __setitem__                                  | 索引赋值           | X[key]，X[i:j]=sequence                                      |
| __delitem__                                  | 索引和分片删除     | del X[key]，del X[i:j]                                       |
| __len__                                      | 长度               | len(X)                                                       |
| __bool__                                     | 布尔测试           | bool(X)                                                      |
| __lt__, __gt__,__le__, __ge__,__eq__, __ne__ | 特定的比较         | 依次为X<Y，X>Y，X<=Y，X>=Y，X==Y，X!=Y注释：（lt: less than, gt: greater than,le: less equal, ge: greater equal,eq: equal, ne: not equal） |
| __radd__                                     | 右侧加法           | other+X                                                      |
| __iadd__                                     | 实地（增强的）加法 | X+=Y(or else __add__)                                        |
| __iter__, __next__                           | 迭代               | I=iter(X), next()                                            |
| __contains__                                 | 成员关系测试       | item in X(X为任何可迭代对象)                                 |
| __index__                                    | 整数值             | hex(X), bin(X), oct(X)                                       |
| __enter__, __exit__                          | 环境管理器         | with obj as var:                                             |
| __get__, __set__,__delete__                  | 描述符属性         | X.attr, X.attr=value, del X.attr                             |
| __new__                                      | 创建               | 在__init__之前创建对象                                       |

接下来，以比较运算符来阐述运算符比较的意义和必要性。

## 美元与人民币

定义两个类

```python
class Dollar:
    def __init__(self, value):
        self.value = value
        self.rate = 1       # 与美元的汇率


class Rmb:
    def __init__(self, value):
        self.value = value
        self.rate = 0.1431      # 1元人民币等于0.1431美元


d = Dollar(0.95)
r = Rmb(6.7)
```

现在请你写代码来判断d是否比r大，在不使用运算符重载的情况下，只能这样来写

```text
d = Dollar(0.95)
r = Rmb(6.7)

def ge(dollar, rmb):
    return dollar.value > rmb.value*rmb.rate

print(ge(d, r))
```

你不可以直接用表达式 d > r来进行判断，因为这样比较的是两个对象的内存地址。编写一个函数来判断两个货币的大小虽然在技术上可行，但从工程上看，着实是一种丑陋的办法。现在，我通过重载__gt__方法来实现两个对象的比较，为了完成这个事情，还需要做一些额外的操作，让这两种货币拥有共同的父类

```python
class Money:
    def __init__(self, value):
        self.value = value

    def __gt__(self, other):
        return self.value*self.rate > other.value*other.rate


class Dollar(Money):
    def __init__(self, value):
        super().__init__(value)
        self.rate = 1       # 与美元的汇率


class Rmb(Money):
    def __init__(self, value):
        super().__init__(value)
        self.rate = 0.1431      # 1元人民币等于0.1431美元


d = Dollar(0.95)
r = Rmb(6.7)

print(d > r)
```

只需要在父类中重载__gt__方法，就可以直接使用 > 运算符比较d 和 r的大小，这比编写一个函数要简单方便的多。
