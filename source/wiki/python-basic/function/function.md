---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.2 函数的定义与调用 
order: 1
---

# python函数的定义与调用

在python中 ，函数是一个组织好的 ，可以重复使用的代码段 ，函数可以提高代码的重复利用率 ，原则上一个函数只实现一个单一的功能 ，这样能增强程序的模块性， python有许多的内置函数可供你使用， 你也可以自己定义函数，这通常被称之为自定义函数

## 1. 函数的定义

```text
def 函数名（参数列表）:
    函数体
```

先看几个示例

```python
def hello_word():
    pass

def print_diamond(count):
    pass

def get_max(a, b, c):
    pass
```

关于上面这段代码，你要记住下面3个结论

1. 函数可以没有参数
2. 函数的参数可以是一个，也可以是多个
3. pass的作用相当于占位符，没有实际作用

## 2. 函数的调用

示例1， 定义并调用没有参数的函数

```python
def hello_word():
    print('hello world')

hello_word()
```

示例2， 定义并调用有一个参数的函数

```python
def print_diamond(count):
    """
    输出菱形
    """
    for i in range(count):
        if i <= count//2:
            print(" "*(count//2-i)  + "*"*(2*i + 1))
        else:
            print(" "*(i - count//2)  + "*"*((count-i)*2-1))

print_diamond(11)
```

示例3， 定义并调用有三个参数的函数

```python
def get_max(a, b, c):
    max_number = a
    if b > max_number:
        max_number = b

    if c > max_number:
        max_number = c

    return max_number


max_num = get_max(22, 23, 21)
print(max_num)
```

当你使用def 定义了一个函数后，只有调用这个函数才能让这个函数运行。

## 3. 函数的返回值

在第2小节中的示例3中，这个函数的功能是找出a, b, c三个数据中的最大值，获得最大值后，使用return 语句将max_number返回，max_number就是函数get_max的返回值，这个返回值最终赋值给变量max_num。

return的作用是退出函数并返回函数的返回值，任何时候，只要执行了return语句就一定会退出函数。你可能已经注意到第2小结中的示例1和示例2的函数里并没有return语句，那么这样的函数有返回值么？我们来做一下实验

```python
def hello_word():
    print('hello world')

res = hello_word()
print(res)
```

尽管函数hello_word没有使用return语句，但仍然会有返回值，这种情况下，函数默认返回None, 关于None，我会专门写一篇教程。

python的函数允许你一次返回多个结果

```python
def return_tuple():
    return 1, 2

res = return_tuple()
print(res, type(res))
```

程序输出结果

```text
(1, 2) <class 'tuple'>
```

函数一次返回多个结果时，是以元组的形式返回的。

如果函数里没有任何数据需要返回，但需要提前结束，也可以使用return，这种用法我会在递归函数中做讲解。
