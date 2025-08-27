---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 3.2 变量引用
order: 1
---

# 变量引用

## 1. 变量不能独立存在

在C++等语言中，变量的声明和赋值是可以分开的

```cpp
int a;
a = 343;
```

而在python中却不行，在声明python变量的同时必须进行赋值操作

```python
a = 343
```

如果你直接使用一个不存在的变量，就会发生错误,NameError: name 'b' is not defined

## 2. 变量是内存中数据的引用

![python变量引用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091528436.jpeg)
a = 343 这样代码被执行时，首先要在内存中创建出343这个对象，然后让a指向它，这便是引用。

此后，我们在程序中使用变量a时，其实都是在使用343，python可以通过a找到343， 这是对引用最通俗的解释。

赋值语句执行过程中，有一点极容易被忽略掉，那就是这个过程中，在内存中创建了新的数据

```python
a = [1]

b = [1]

print(a == b)
print(a is b)
```

两行赋值语句，分别将列表[1] 赋值给a 和 b，表达式 a == b 的结果是Ture，因为他们的内容的确相同，但表达式a is b的结果是False, 因为这两行赋值语句执行过程中，一共创建了两个列表，他们只是内容相同而已，但内存地址绝对不相同，下图是这两个变量的内存描述示意图
![python变量引用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091529549.jpeg)

## 3. 变量的内存地址与类型

### 3.1 内存地址与类型

下面的代码会输出变量的内存地址与类型

```python
a = 343
print(id(a))
print(type(a))
```

程序输出为

```text
4325858928
<class 'int'>
```

虽然标题是“变量的内存地址和类型”，但从本质上来说，内存地址是343的内存地址，类型是343的类型。

### 3.2 改变指向

如果将a 的指向改变，即将另外一个值赋值给a， 那两行print输出的内容就会发生变化

```python
a = 343
print(id(a))
print(type(a))

print('改变a的指向')
a = 'python'
print(id(a))
print(type(a))
```

程序输出

```text
4325858928
<class 'int'>
改变a的指向
4339812760
<class 'str'>
```

这究竟是怎么一回事呢，下图展示了变量a改变指向的过程
![改变python变量指向](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091530093.jpeg)
变量a最终指向了字符串'python'， 而343由于没有变量指向它，因此引用数量从1变成了0。

python的变量，总是要指向一个内存中的数据才行，因此，它不能孤立的存在。

### 3.3 多个变量指向同一个数据

```python
a = 343
b = a

print(id(a))
print(id(b))
```

程序输出为

```text
4325858928
4325858928
```

变量a的内存地址和变量b的内存地址一样，下图是这种情况的示意图
![多个python变量指向同一个对象](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091528874.jpeg)
这两个变量都指向了 343， 因此343的引用数量是2

## 4. 引用数量

当一个数据没有变量指向它时，这个数据的引用数量就变成了0，python会销毁掉这种对象，这就是GC（垃圾回收），你可以通过sys.getrefcount() 方法查看一个数据的引用数量。

下面这段代码，一定要在python交互式解释器下执行，因为其他环境下，会做优化处理，导致实际结果与理论结果不符。

```python
Python 3.6.3 (v3.6.3:2c5fed86e0, Oct  3 2017, 00:32:08)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> a = 343
>>> b = a
>>> sys.getrefcount(a)
3
```

- a 指向了343，引用次数加1
- b 也指向了343，引用次数加1
- a作为实参传入getrefcount函数中，会进行一次参数复制，引用次数加1

所以，最终343的引用数量是是3
