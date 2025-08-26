---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.6 常见的错误和异常
order: 5
---

# 19个常见的python错误和异常

错误总是不可避免，尤其是在初学阶段，本文收集整理了1个常见的python错误

## 1. 忘记添加:

在if, elif, else, for, while, class，或者使用def定义函数的名称后面忘记添加:，就会引发 SyntaxError

```python
if 3 > 4
    print('ok')
```

这是一种非常明显的错误，大多数编辑器在你写代码的时候就会用红色的波浪线提示你

## 2. 误将 = 当做 ==

```python
a = 4
if a = 4:
    print('ok')
```

即便是多年编程经验的老手也会偶尔犯这种错误，写的太着急了，就少打了一个=，得到的自然也是SyntaxError

## 3. 错误的缩进空格数量

```python
if 1==1:
    print('1')
   print('2')
```

这段代码将引发错误“IndentationError: unindent does not match any outer indentation level” ，一次缩进是4个空格，这一点务必要牢记

## 4. 错误使用range函数

```python
lst = [1, 2, 3]
for i in range(lst):
    print(lst[i])
```

这段代码的本意是通过索引来遍历列表，但错误的使用了range函数，引发了错误“TypeError: 'list' object cannot be interpreted as an integer”, 正确的做法如下

```python
lst = [1, 2, 3]
for i in range(len(lst)):
    print(lst[i])
```

## 5. 尝试修改字符串的内容

字符串是不可变对象，无法修改字符串里的内容，下面的代码将会引发错误“TypeError: 'str' object does not support item assignment”

```python
s = "i like python"
s[0] = 'I'
print(s[0])
```

## 6. 尝试将非字符串数据与字符串连接

```python
print('I have ' + 3 + " books")
```

上面的代码尝试将字符串与int类型数据连接在一起，由于他们类型不同，会导致错误“TypeError: must be str, not int”

## 7. 字符串缺少引号

```python
print('hello world)
```

创建字符串可以是用一对单引号，或者一对双引号，或者一对"""， 上面的代码会引发错误“SyntaxError: EOL while scanning string literal”

## 8. 使用未定义的变量

```python
sname = "lilei"

print('my name is ' + name)
```

print语句中需要用到的变量name事先并没有被定义，就会引发错误“NameError: name 'name' is not defined”

## 9. 调用对象没有的方法

```python
string = 'PYTHON'
# 经过一些操作后,string变成了None
string = None
print(string.lower())
```

string原本是字符串，但进过一些操作后，变成了其他对象，可能是int，或者None，不论变成什么，总是它都不再是字符串，没有了lower方法，这时你再去调用lower方法就会报错“AttributeError: 'NoneType' object has no attribute 'lower'”

## 10. 访问不存在的索引

```python
lst = [1, 2, 3]

print(lst[6])
```

列表最大的索引是2，代码里尝试访问索引6就会引发索引错误“IndexError: list index out of range”

## 11. 使用一个不存在的key

需要通过key来操作字典，如果key不存在，就会引发错误“KeyError”

```python
dic = {
    'name': 'lili',
    'age': 14
}

print(dic['sex'])
```

## 12. 使用保留字做变量

```python
class = 'python'
print(class)
```

python的保留字不能作为变量，上面的代码会引发错误“SyntaxError”

## 13. 使用不存在的内置函数

```python
lst = [1, 2, 3]
print(avg(lst))
```

这个错误与第8个错误相似，都是使用了一个不存在的对象

## 14. 在函数内修改全局不可变对象

这是一个比较复杂的错误，先来看下面的代码

```python
a = 10

def func():
    print(a)

func()
```

程序正常执行，可以输出10，对代码稍作修改

```python
a = 10

def func():
    print(a)
    a = 20

func()
```

多了一行a = 20后，再次运行代码就会报错“UnboundLocalError: local variable 'a' referenced before assignment”，为什么会这样呢?a原本是一个全局变量，在第一段代码里可以正常访问，但是第二段代码里尝试对变量a进行修改，一旦有了修改这个动作，解释器就认为变量a是一个局部变量，而不在是全局变量。那么在a = 20这条语句之前尝试输出a的内容就会报错因为在执行print(a)时，局部变量a还不存在。

## 15. 修改range的返回值

```python
lst = range(10)
lst[0] = 20
print(lst)
```

range函数创建一个整数序列，但这个序列并不是列表，而是一个迭代器，无法使用索引来进行任何操作，否则就会引发错误"TypeError: 'range' object does not support item assignment"

## 16. 使用 ++ 或 --操作

```python
a = 0
a++
print(a)
```

很多语言都支持 ++ 操作，但很遗憾python并不支持，上面的代码会报错“SyntaxError: invalid syntax”，上面的代码可以修改成这样

```python
a = 0
a += 1
print(a)
```

## 17. 函数调用时参数数量错误

错误1

```python
def func(a, b):
    return a + b

print(func(4))
```

错误2

```python
def func(a, b):
    return a + b

print(func(4, 4, 5))
```

函数需要两个参数，错误1里在调用函数时只提供了1个参数，错误2里提供了3个错误，都会引发TypeError，错误内容分别是“TypeError: func() missing 1 required positional argument: 'b'” 和 “TypeError: func() takes 2 positional arguments but 3 were given”

## 18. 缺少安装包

```python
import requests
```

如果你并没有安装requests库，程序在执行时就会报错“ImportError: No module named requests”，不要慌，使用pip安装就好了

## 19. 文件路径错误

```python
f = open('a.txt')
print(f.read())
```

如果根本不存在a.txt这个文件，那么就会报错“FileNotFoundError: [Errno 2] No such file or directory: 'a.txt'”， 打开一个不存在的文件，就会引发FileNotFoundError。这个错误，对于初学者来说来烦恼了，尤其是windows用户，他们有时候会信誓旦旦的说，这个文件存在啊！

程序是不会骗人的，它说不存在，就一定是不存在，你的文件地址一定是错了，检查一下路径里究竟用的是/ 还是 \\， 另外检查一下是不是隐藏了文件的后缀。
