---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.9 理解python的赋值语句
order: 8
---

# 正确理解python的赋值语句

- 正确理解python的赋值语句
  - [1. a = a + 1](http://www.coolpython.net/python_primary/function/py-assign-statement.html#toc_1)
  - [2. 等号右侧是函数](http://www.coolpython.net/python_primary/function/py-assign-statement.html#toc_2)
  - [3. 等号右侧是包含函数的表达式](http://www.coolpython.net/python_primary/function/py-assign-statement.html#toc_3)
  - [4. 总结](http://www.coolpython.net/python_primary/function/py-assign-statement.html#toc_4)

在我教授学生学习函数章节时，学生们突然对python的赋值语句产生了不同程度的困惑，对于函数调用后的返回值赋值给某个变量这件事情，似乎总是不能做到清晰的理解，因此我决定专门写一篇教程。

普通的赋值语句是极容易理解的，比如下面的这些示例

```python
a = 5
b = [4, 5, 8]
```

赋值语句的左侧是变量，右侧是具体的数值，这是最简单的赋值语句，也是你最初接触变量这个概念时所学习的，然而随着难度的增加，渐渐的，有些赋值语句让你的思维变得混乱，下面我举几个具体的例子，来强化你对赋值语句的理解

## 1. a = a + 1

我们从最简单的开始

```python
a = 3
a = a + 1
```

a 最后的值为4， 这个结果不难理解，难在深刻的理解，第一行代码，我们将3这值赋值给a， 第二行代码，是将a + 1 的结果赋值给a， 对于赋值语句，我们永远先关注等号的右侧，不要让等号的左侧影响到我们的思考。等号的右侧是 a + 1，它是一个简单的表达式，不懂表达式的朋友，请移步我的python基础教程[表达式](http://www.coolpython.net/python_primary/expression/expression.html)。表达式必然有结果，a + 1 = 3 + 1 = 4，等号右侧表达式的结果是4，最后将4赋值给a， 这就是代码a = a + 1的执行过程，**先计算等号右侧的表达式，然后将表达式的结果赋值给等号左侧的变量**

## 2. 等号右侧是函数

接下来，举一个稍微复杂一点点的例子

```python
def add(x, y):
    return x + y
    
a = add(3, 4)
print(a)        # 7
```

变量a 的值等于7， 不论等号右侧是什么，我们都要先计算等号右侧，在这个例子中，等号右侧是什么呢？等号的右侧调用执行函数add，函数都是有返回值的，因此等号右侧是函数add被调用后的返回值。函数add传入的参数是3 和 4 ，函数的返回值是7 ，等号右侧执行函数，得到的返回值是7，将7赋值给变量a

## 3. 等号右侧是包含函数的表达式

不要觉得简单，就走马观花，你还嫩的很，接下来，看下面稍复杂一点点的例子

```python
def add(x, y):
    return x + y
    
a = add(3, 4) + 7
print(a)        # 14
```

和前面的例子一样，这一次，仍然先看等号右侧，等号右侧是add(3, 4) + 7， 函数都有返回结果，只要看到函数被调用，被执行，你就必须想到它有返回结果，并且计算出它的返回结果，add(3, 4) = 7, add(3, 4) + 7 = 7 + 7 = 14，将14赋值给变量a。

是不是很轻松，再看下面的例子

```text
def add(x, y):
    return x + y
    
a = 3
a = add(a, 4) + 7
print(a)        # 14
```

还是先看等号右侧，add(a, 4) + 7， 先计算add(a, 4) = add(3, 4) = 7 , add(a, 4) + 7 = 7 + 7 = 14， 最后将14赋值给变量a

## 4. 总结

回顾本篇要点

1. 赋值语句，我们只考虑等号右侧，将等号右侧的计算结果赋值给等号左侧的变量
2. 不要被等号左侧的变量干扰了思维，我在文章里从来不提等号左侧的变量
3. 函数都有返回值，这是你必须关注的，你看到函数被执行，就一定要想到它有返回值，而且一定要计算出这个返回值，拿到这个返回值，我们才能去计算表达式的结果
