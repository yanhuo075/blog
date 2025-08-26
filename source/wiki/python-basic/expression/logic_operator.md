---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 4.5 逻辑运算符
order: 4
---

# 逻辑运算符

python的逻辑运算符只有3个：and，or, not ，对应着逻辑运算中的与，或，非。逻辑运算符将多个条件连接在一起，结合小括号设置运算优先级构成更加复杂的逻辑判断，多用于if语句和while循环语句。

| 运算符 | 逻辑表达式 | 描述                                                  | 实例                                                         |
| ------ | ---------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| and    | x and y    | x和y都为True时，结果为True,否则为False                | True and True 结果为True, 有一个为False，表达式结果即为False |
| or     | x or y     | x和y只要有一个为True，则表达式结果为True              | True or False 结果为True                                     |
| not    | not x      | x 为Ture, not x 则为False, x为False, not x 结果为True | not (3 > 4 and 5 > 3) 结果为True                             |

and 是布尔与运算，表示并且的意思，并且想要成立，就要求并且两边的表达式都为True，有一个是Flase，表达式为False

or 是布尔或运算，表示或的意思，只有一个条件成立了，整个表达式的结果即为True

not 是布尔非运算，表示相反的意思，就是把True和False颠倒过来。

实际使用中，表达式看起来会很复杂，甚至会使用小括号，那么优先计算小括号里的表达式结果

```python
True and (3 > 2 or 2 > 100)  # 结果True
(5 > 4 and 3%2 == 1) or (4 == '' and True)  # 结果为True
4 and 0 # 结果为False
```

- 不论多复杂的逻辑表达式，有小括号，先计算小括号里的结果
- 在逻辑表达式中，0，None, 空字符串，空列表，空字典，空集合等价于False， 其余的数据等价于True
