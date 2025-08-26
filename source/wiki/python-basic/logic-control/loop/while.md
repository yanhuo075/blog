---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.2.5 while循环
order: 5
---

# python while循环

python中的while语句用于循环执行程序，当给定的判断条件为True时执行循环体，循环体里是一段需要重复执行的代码。每一次执行完循环体都要重新对判断条件进行计算，只有当判断条件为False的时候才会终止循环， 此外，也可以使用break语句终止循环。

## 1. python while循环的一般形式

```text
while 表达式:
    代码块
```

- 只要表达式的结果是True，就会执行循环体里的代码
- 当循环体里的代码执行结束后，会再次进行条件判断，如果表达式的结果是True，会继续执行循环体里的代码

思考题，下面的代码，会永远执行下去么?

```python
while True:
    print('while')
```

上面的代码会一直执行下去，形成了死循环，除非你有意实现一个死循环，否则，这样的死循环会导致致命的问题

## 2. for循环与while循环对比

for循环，本质上是迭代，在循环之前，你基本上是知道要循环多少次的，但是while循环则不然，有时候你知道要循环多少次，但有些while循环，你根本不知道要循环多少次。

for循环举例子

```python
lst = [1,2,3]
for item in lst:
    print(item)
```

这段代码，你看了，就知道会循环3次，同样是循环遍历lst，用while循环实现一下

```python
lst = [1,2,3]

index = 0
while index < len(lst):
    print(lst[index])
    index += 1
```

看到代码，你也能判断循环会执行多少次，但下面的代码，会执行多少次，你就无法知晓了

```python
while True:
    input_str = input("请输入一个正整数，想退出程序请输入 quit:")
    if input_str == "quit":
        break
    number = int(input_str)

    if number % 2 == 0:
        print("你输入的是一个偶数")
    else:
        print("你输入的是一个奇数")
```

这个循环要执行多少次，你事先是无法知晓的，因为什么时候退出，取决于用户的输入，请思考一下，如果用for循环来实现上面的这个功能，应该怎么做？

尽管在python里，通过技术手段，可以实现for循环的无限循环，但我们不这样做，那些需要无限循环的代码，使用while True 这种形式就可以了。

## 3. 练习题

### 3.1 题目要求

```python
lst = [3, 6, 1, 8, 10, 3, 20, 13]
```

请使用while循环找出列表里的最大值和最小值，并使用print输出他们

### 3.2 思路分析

while循环同样可以用来遍历列表和元组，这两种容器都有索引，初始化变量index = 0， index < len(lst) 作为while循环的条件，在循环体里，让index += 1， 这样就可以实现对列表的遍历

### 3.3 示例代码

```python
lst = [3, 6, 1, 8, 10, 3, 20, 13]


max_value, min_value = lst[0], lst[0]
index = 0
while index < len(lst):
    tmp = lst[index]
    if tmp > max_value:
        max_value = tmp

    if tmp < min_value:
        min_value = tmp

    index += 1      # 这行代码千万别忘了,否则就会死循环

print((max_value, min_value))
```

## 4. while循环存在的意义

for循环同样可以实现死循环，根据前面所举的例子来看，似乎while循环能实现的功能，for循环都可以实现，那么，while循环存在的意义是什么呢？肯定存在某种情况，使用while循环比for循环更好，甚至使用for无法实现相同的功能。

给你一个整数，要求你计算每一位数字的和，比如整数4352，4 + 3 + 5 + 2 = 14,下面使用while循环来实现这个求和的过程

```python
number = 4352
_sum = 0

while number > 0:
    _sum += number % 10
    number //= 10

print(_sum)
```

试想一下，这样的循环，for循环可以实现么，如果你会使用生成器，当然也可以使用for循环来求和，但那样的代码就没有使用while循环来的简单容易。
