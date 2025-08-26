---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 4.8 成员运算符
order: 7
---

# 成员运算符

| 运算符 | 描述                                                    | 实例                                 |
| ------ | ------------------------------------------------------- | ------------------------------------ |
| in     | 如果在指定的序列中找到值返回 True，否则返回 False。     | 3 in [1, 2, 3] 表达式结果为True      |
| not in | 如果在指定的序列中没有找到值返回 True，否则返回 False。 | 3 not in [1, 2, 3] 表达式结果为False |

python的成员运算符用于判断一个对象是否是另一个对象的成员，成员运算符只两个：in 和 not in。成员运算符in 后面的对象必须是可迭代对象，例如列表，元组，字典，集合，字符串。

```python
>>> 'a' in 'abc'
True
>>> 1 in [1, 2, 3]
True
>>> 2 in (1, 2, 3)
True
>>> 3 in {1, 2, 3}
True
>>> 'key' in {'key': 'value'}
True
```

如果你掌握了in 这个运算符，那么not in 也就顺带手的理解了，not in 就是对in 的一个否定判断。
