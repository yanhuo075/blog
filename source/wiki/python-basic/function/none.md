---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.3 python中的None
order: 2
---

# python中的None

## 1. None的类型

None在python中是一个特殊的对象，它表示空值，其类型为NoneType

```text
>>> type(None)
<class 'NoneType'>
```

## 2. 只存在一个None

None只存在一个，python解释器启动时创建，解释器退出时销毁

```text
>>> a = None
>>> b = None
>>> a == b
True
>>> a is b
True
```

由于内存None只有一个，所以a is b的结果为True

## 3. None 的运算

None不支持任何运算，也没有内建方法，除了表示空以外，什么都做不了。

如果要判断一个对象是否为None,使用is身份运算符

```text
>>> a = None
>>> a is None
True
```

## 4. None的使用

如果一个函数，没有显式return任何数据，则默认返回None。

在判断语句中，None等价于False

```text
>>> a = None
>>> not a
True
```
