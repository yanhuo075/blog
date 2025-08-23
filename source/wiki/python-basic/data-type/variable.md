---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 3.5 变量的概念
order: 4
---

# python中变量的概念

在python中，变量就是一种标识符， 它是数据的名字，更专业的理解，变量是内存中数据的引用， 编程语言里的变量和初中学习代数时的方程变量很相似。

前面学习数字类型，bool类型时，我们一直在交互式解释器里进行操作，目的是为了让你熟悉交互式解释器，同时作为在编辑器里写代码的一种适应性训练，后续的有关数据类型的文章中，我会两种方法都使用。

关于如何在编辑器里写代码，你可以参考第一章里的[《安装配置pycharm》](http://www.coolpython.net/python_primary/introduction/install_pycharm.html)教程。

## 1. 概念讲解

在本教程基础语法章节里，有一篇名为[《标识符与保留字》](http://www.coolpython.net/python_primary/basic_grammar/identifier.html)的教程，详细阐述了标识符的概念。变量就是一种标识符，程序 = 数据 + 算法，数据有各种类型，我们已经学习了int，float， bool， 学习本篇教程后，你还将明白，数据还有名字。

我们可以简单的认为，变量就是数据的名字，更专业的理解，变量是数据的引用，在交互式解释器里跟随我进行操作

```python
>>> a = 4
>>> b = 5
```

在上面的代码里，我定义了两个变量，a 和 b,并对他们进行赋值，那么这样做究竟有什么用呢？ 如果你想计算4乘以5的结果，你不比再像之前那样写4*5，而是写成

```python
>>> a*b
20
```

如果你希望能有另外一个变量保存20，那么可以写成

```text
>>> c = a*b
>>> c
20
```

## 2. 变量命名规范

1. 变量名只能包含字母、数字和下划线。变量名可以字母或下划线开头，但不能以数字开头，例如，可将变量命名为message_1，但不能将其命名为1_message。
2. 变量名不能包含空格，但可使用下划线来分隔其中的单词。例如，变量名greeting_message可行，但变量名greeting message会引发错误。
3. 不要将Python关键字和函数名用作变量名，即不要使用Python保留用于特殊用途的单词，如print, sum
4. 变量名应既简短又具有描述性。例如，name比n好，student_name比s_n好，name_length比length_of_persons_name好

## 3. 赋值语句

下面是一个赋值语句

```text
>>> a = 343
```

就这么一行简单的代码，背后却隐藏着许多知识

### 3.1 python的变量不能单独存在

在c++中，你可以这样写

```cpp
int a;
a = 4;
```

变量的声明和赋值可以分开，但python中不可以，变量不能只声明而不赋值，必须在声明变量的同时，进行赋值

### 3.2 python的变量是没有类型的

python中，变量是没有类型的，虽然可以用type函数来查看类型，但本质上，变量的类型是由赋值给变量的数据来决定的

```python
>>> a = 343
>>> type(a)
<class 'int'>
>>> a = 34.2
>>> type(a)
<class 'float'>
```

### 3.3 对象的创建

在python中，一切皆对象， a = 343 这样一行简单的代码，其背后就涉及到了343这个对象，int，float,bool类型的数据，都是对象，当a = 343这行代码被执行时，python在内存中创建了一个对象，下面是一个简单的变量与内存中对象关系的示意图
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822151938296.jpeg)

变量a 是内存中343这个对象的引用，关于引用的概念，如果你是初学者，不必太纠结，在进阶教程中，我会详细讲解。
