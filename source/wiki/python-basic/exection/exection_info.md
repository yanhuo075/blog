---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.5 异常信息分析与收集
order: 4
---

# 异常信息分析与收集

## 1. 异常信息分析

程序异常崩溃时会提供非常详细的错误信息，掌握正确的分析方法，就可以快速定位问题并解决问题，下面这段代码会引发异常导致程序终止

```python
def func_tet():
    func_sum('4', 3)


def func_sum(a, b):
    value = a + b
    return value


func_tet()
```

运行这段程序，异常信息如下

![python异常信息](http://www.coolpython.net/pictures/python_primary/exection/exection_info-1583242480-0.jpg)

我将异常信息分为两部分，分析时，先关注最后一行绿色框内的信息，这里的信息明确的指明了异常的类型和异常的解释信息，这是我们分析问题的第一步，随着经验的积累，你很容就能通过异常信息分析出为何会发生异常。



异常信息的第二部分，就是蓝色框内的内容，是调用堆栈信息，详细的记录了程序的执行路径，最后一行正是错误发生的位置。

现在，既有出错代码的位置，又有错误的类型与解释，如果还是不能找出问题，那么，可以百度了，百度时将最后一行绿色框里的内容作为搜索词进行搜索，可以找到大量文章。

## 2. 异常信息收集

traceback模块可以准确获得有关异常的信息，这在稍大点的项目里非常有用。

在大一点的项目里，需要记录程序运行的日志，那些异常日志非常非常重要，将有助于你发现系统的问题。

```python
import traceback
def divide(x, y):
    try:
        return x/y
    except:
        msg = traceback.format_exc()
        print(msg)

if __name__ == '__main__':
    #print(divide(10, 5))
    print(divide(10, 0))
```

traceback.format_exc() 可以获取有关异常的详细信息，这些信息就是平时你的程序遇到bug时报出来的那些异常信息

```text
Traceback (most recent call last):
  File "/Users/kwsy/PycharmProjects/pythonclass/mytest/test.py", line 4, in divide
    return x/y
ZeroDivisionError: division by zero
```

有了这些信息，你就清楚的知道错误发生在哪里，异常的类型是什么，你甚至可以在日志里将函数的入参写入到日志中，这样，更加有助于你解决问题。
