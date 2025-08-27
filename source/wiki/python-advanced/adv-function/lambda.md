---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.4 lambda
order: 3
---

# python lambda表达式精讲

## 1. lambda表达式究竟是什么

在学习python过程中，lambda表达式是一个令人感到困惑的语法，你在许多源码里可以看到它，但对于它究竟是什么，起到什么功能作用却不甚了解。

下面用一个常见的例子来阐释lambda表达式的功能作用

```python
lst = [('小明', 98), ('小刚', 85), ('小红', 94)]

lst.sort(key=lambda x: x[1])
print(lst)
```

列表lst中存储的是3个人的考试成绩信息，使用sort对列表进行排序。sort方法里的key参数决定了用什么值进行排序，在这个问题中，我们只能使用3个人的分数进行排序，以上是对整个问题的概况了解。

lambda表达式定义创建了一个匿名函数，x是这个匿名函数的参数，x[1] 是这个函数的返回值，当('小明', 98)传入时，返回98，('小刚', 85)传入时，返回85，依次类推。

如果你不想用lambda表达式，可以直接定义一个函数，完成同样的功能

```python
lst = [('小明', 98), ('小刚', 85), ('小红', 94)]


def score(x):
    return x[1]

lst.sort(key=score)
print(lst)
```

## 2. lambda表达式创建的函数与普通函数的区别

lambda表达式创建的是匿名函数，这个匿名函数和我们用def创建的函数有什么不同么？

下表详细展示了两者的区别

| 函数             | 是否有名字 | 是否有函数文档 | 代码行数要求 | 是否自动返回结果 |
| ---------------- | ---------- | -------------- | ------------ | ---------------- |
| def 创建的函数   | 是         | 是             | 无限制       | 否               |
| lambda创建的函数 | 否         | 否             | 一行         | 是               |

### 2.1 匿名函数自身信息

我们用def创建的函数，有自己的名字和函数文档

```python
def score(x):
    """
    返回分数
    :param x:
    :return:
    """
    return x[1]

print(score.__name__)
print(score.__doc__)
```

程序运行结果

```text
score

    返回分数
    :param x:
    :return:
```

lambda创建的函数是匿名函数，既没有名字，也没有函数文档，不然也不会叫它匿名函数，由于lambda没有名字，因此，它只能使用一次，或许你见过这样的代码

```python
add = lambda x, y: x+y
print(add(1, 4))
print(add(1, 5))
```

上面的代码里，将lambda表达式所创建出来的函数赋值了变量add，add使用了两次，似乎和我刚才所讲的匿名函数没有名字，只能使用一次相矛盾,但这只是假象。

如果你执行print(add.__name__) 就会发现，add的函数名字永远是<lambda>, 所有的lambda表达式创建出来的匿名函数的名字都是<lambda>，虽然，你使用这种方法让这个匿名函数执行了多次，但是抱歉，这种做法是PEP8中明确建议禁止的行为。

由于lambda创建的函数没有名字，因此，它只能作为其他函数的参数来使用。

### 2.2 匿名函数特殊语法

我们用def创建的函数，对于函数体内代码行数是没有要求的，但你所看到的lambda创建的匿名函数，只有一行代码，你想写出多行，是不可能的,你甚至不能再lambda表达式中对变量进行赋值。

lambda表达式会自动返回结果，无需return，而def函数如果不使用return返回结果，默认会返回None。

如果一段逻辑，只需要一行代码就可以完成，且只会使用一次，那么就应该用lambda表达式来完成这个功能，用def定义函数当然也是可以的，但很容易产生大量的只有一行代码的函数，关键这些函数只使用一次或者少出几次，为了代码的干净整洁，使用lambda表达式是最合适的，正是为了干净整洁，所以lambda表达式里不需要return。

## 3. lambda表达式应用

要在合适的场景应用lambda表达式

- 逻辑简单
- 只用一次或极少次
- 作为参数

下面这段代码计算列表里所有数据的乘积

```python
lst = [2, 1, 3]
product = 1
for item in lst:
    product *= item

print(product)
```

如果改为reduce算法，则可以这样写

```python
from functools import reduce

lst = [2, 1, 3]
product = reduce(lambda x, y: x * y, lst, 1)
print(product)
```

lambda表达式作为reduce函数的参数来使用，如果你在工作中需要使用pyspark，那么，lambda表达式应用的会更加频繁
