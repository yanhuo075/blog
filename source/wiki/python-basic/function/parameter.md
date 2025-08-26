---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.4 函数的参数
order: 3
---

# python函数的4类参数

python函数的参数可以分为位置参数，默认参数，关键字参数，可变参数，函数可以没有参数，也可以有多个参数，参数是传递给函数的命名变量。

## 1. 形参和实参

所谓形参，就是函数定义中的参数，形参在函数体内可以使用

而实参，则出现在调用过程中，这两个概念不必刻意去记忆

```python
def print_diamond(count):
    for i in range(count):
        if i <= count//2:
            print(" "*(count//2-i)  + "*"*(2*i + 1))
        else:
            print(" "*(i - count//2)  + "*"*((count-i)*2-1))

print_diamond(11)
```

上面的代码中，函数定义中的count就是形参，而最后一行的代码中，11就是实参，形参代表一种形式，实参，是实际的具体的起作用的数据。

## 2. 必传参数(位置参数)

```python
def my_print(content, count):
    for i in range(count):
        print(content)

my_print('ok', 2)
```

上面定义的my_print函数里有两个参数，content 和count，他们都是必传参数，在调用函数时，你必须传入两个值，否则就会报错，下面演示一种错误的调用方法

```python
def my_print(content, count):
    for i in range(count):
        print(content)

my_print('ok')
```

报错内容为

```text
Traceback (most recent call last):
  File "/Users/kwsy/kwsy/coolpython/test.py", line 5, in <module>
    my_print('ok')
TypeError: my_print() missing 1 required positional argument: 'count'
```

从最后的TypeError的内容来看，缺失了一个位置参数（positional argument），我想，必传参数更能准确的描述这个参数的性质。 在函数my_print内部，用到了count这个形参，可是调用函数时，只传了content, 却没有传count，最终报错。

## 3. 默认参数

my_print函数根据count参数输出指定次数的content，我希望在两次输出的空档可以暂停一段时间，因此增加一个sleep参数

```python
import time


def my_print(content, count, sleep=1):
    for i in range(count):
        print(content)
        time.sleep(1)

my_print('ok', 2)
```

在调用函数时，我并没有传sleep这个形参，程序并没有报错，这是因为在定义函数的时候，给了sleep形参默认值1，这意味着，如果调用函数时没有传这个参数，python会用默认值来执行函数。默认值参数非常用有用，假设你在系统里的很多地方都使用最开始定义的my_print函数

```python
def my_print(content, count):
    pass
```

开始某一天，你需要增加一个参数sleep， 如果你将sleep定义为必传参数，那么所有用到函数my_print的地方都必须修改调用时的代码来增加这个入参，但如果你把sleep定义为默认参数，sleep已经默认等于1，除非必须传入其他的数值，否则就不需要这么做，之前的代码仍然可以使用。

定义函数是时，如果有多个默认参数，他们必须放置在参数列表的最后，不允许在中间放置一个必传参数。

## 4. 关键字参数

关键字参数不是一个出现在函数定义时的概念，而是一个出现在函数调用时的概念。

```python
import time


def my_print(content, count, sleep=1):
    for i in range(count):
        print(content)
        time.sleep(1)

my_print(count=2, content='关键字参数', sleep=2)
```

以第3小节中定义的函数my_print为例，在调用函数时，我使用了key=value的形式来传递参数，不仅如此，还打乱了顺序，先传入了count，后传入了content，关键字参数允许你以任何顺序传递参数，只要必传参数都以key=value的形式传值即可。现在，你应该明白我前面所讲的，关键字参数是出现在函数调用时的概念。

使用关键字参数，可以让参数传递更加明确，让调用方清楚的知道每个参数的传值情况。

## 5. 可变参数

可变参数分为两种：

1. *args 接受任意多个实际参数
2. **kwargs接收任意多个以关键字参数赋值的实际参数

### 5.1 *args

在定义函数时，有时候你并不希望参数的个数是固定的，这种设计在实际工作中很常见。

```python
def func(*args):
    print(args, type(args))
    sum_res = 0
    for item in args:
        sum_res += item

    return sum_res

print(func(2))
print(func(2, 3, 4))
```

你可以看到，我在定义func时，使用*args， args只是个名字，你可以随意修改，关键是前面有一个星。有了这个星，函数调用时传递多少个参数就变成了一个很随意的事情，所有传入的参数将被放入到一个元组中，因此args的类型是tuple元组。

### 5.2 **kwargs

```python
def print_score(**kwargs):
    print(kwargs, type(kwargs))
    for course, score in kwargs.items():
        print(course, score)


print_score(yuwen=89, shuxue=94)
```

在调用函数时，以关键字参数的形式进行参数传递，最终这些key-value对将组装成字典，kwargs的类型是dict。个人理解，**kwargs就是一种为了偷懒而做的设计，当函数需要传入很多参数(多达10几个)时，使用**kwargs定义函数是非常方便的。
