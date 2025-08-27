---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.6 with语句
order: 5
---

# 深入理解python with 语句

- 深入理解python with 语句

  - 1. 使用with打开文件

  - 2. 上下文管理器和with 语句有关的概念

    - 2.1 上下文管理协议
    - 2.2 上下文管理器
    - 2.3 运行时上下文
    - 2.4 上下文表达式
    - 2.5 语句体

  - 3. 使用with语句控制线程锁的释放

  - 4. 同时打开多个文件

  - 5. 自定义上下文管理器

python中with 语句作为try/finally 编码范式的一种替代, 适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的”清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等

## 1. 使用with打开文件

你应该见过下面这种打开文件的方式

```python
with open('data', 'r', encoding='utf-8') as f:
    data = f.readlines()
```

上面的写法，与下面的写法在最终效果上是一致的

```text
f = open('data', 'r', encoding='utf-8')
try:
    data = f.readlines()
except:
    pass
finally:
    f.close()
```

对比两段代码不难发现，使用with语句时，代码更加简洁，而且不用主动关闭文件，在with语句体退出时，会自动关闭文件，即便with语句体中发生了异常。

## 2. 上下文管理器和with 语句有关的概念

想要理解with语句，就必须先理解以下几个概念

### 2.1 上下文管理协议

简单来说，就是实现两个方法，__enter__() 和__exit__()

### 2.2 上下文管理器

实现了__enter__() 和__exit__()的对象就是上下文管理器

### 2.3 运行时上下文

由上下文管理器创建，在with语句体代码执行前，通过__enter__()进入，语句体代码执行结束后，通过__exit__()退出

### 2.4 上下文表达式

在with关键字后面的表达式，表达式返回上下文管理器对象

### 2.5 语句体

with语句包裹起来的代码

## 3. 使用with语句控制线程锁的释放

使用with不仅能够自动的关闭打开的文件对象，还可以自动的释放线程锁，这样可以避免死锁的发生，在[python多线程---线程锁](http://www.coolpython.net/python_senior/concurrent/multithreading_lock.html)一文中，为避免多个线程同时对一个变量对象进行修改，在关键语句上加了线程锁

```text
def worker():
    time.sleep(1)
    global a
    for i in range(100000):
        m_lock.acquire()    # 加锁
        a += 1
        m_lock.release()    # 释放锁
```

如果你忘记了写m_lock.release() 对锁进行释放，那么这将导致其他线程永远也无法获取到线程锁，这样就形成了死锁，上面的代码在acquire之后，使用release释放所，使用with语句，可以更加优雅的实现加锁和释放锁的操作。

```python
def worker():
    time.sleep(1)
    global a
    for i in range(100000):
        with m_lock:
            a += 1
```

## 4. 同时打开多个文件

许多人都不知道，with语句可以同时打开多个文件，这样做可以减少代码的缩进，让代码的编写更加容易，两个open语句之间用逗号分隔即可。

```python
with open('a1', 'w')as f1, open('a2', 'w')as f2:
    f1.write('a')
    f2.write('b')
```

## 5. 自定义上下文管理器

在调试程序性能时，如果只是想知道某个函数的执行时长，可以使用一个可以统计函数运行时长的装饰器进行处理，但程序往往很复杂，一段代码里，要做很多操作，不只是调用了一个函数，也可能存在循环，因此，单纯的知道某个函数的执行时长，不能帮助我们更好的了解程序的性能。

我们需要针对某个代码段进行时间统计，知道这一段代码的执行时长对我们很有帮助。你可以使用time.time()方法在代码段开始时获取到时间，在结束时再次获取到时间，两个时间做差就可以得到这个代码段的运行时长，这种操作方式写起来很麻烦，如果有多处代码段需要统计，就得写多次，很不方便。

下面是一个可以统计代码段运行时长的上下文管理器

```python
import time


class ProTime(object):
    def __init__(self, tag=''):
        self.tag = tag

    def __enter__(self):
        self.start_time = time.time()

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end_time = time.time()
        time_diff = self.end_time - self.start_time
        msg = "代码段{tag}运行时长{time_diff}".format(tag=self.tag, time_diff=time_diff)
        print(msg)


with ProTime('first') as pt:
    # 这里是你要统计运行时长的代码块
    time.sleep(1)

with ProTime('second') as pt:
    # 这里是你要统计运行时长的代码块
    time.sleep(2)
```

理解这段代码的关键之处，在with语句所包裹的语句体执行之前，先要执行__enter__方法，语句体执行结束之后，不论是否有异常，都要执行__exit__，在__exit__方法里，三个参数提供了异常的全部信息，如果你想处理异常，可以在这个方法里做处理。

__init__ 方法有一个tag参数，设置这个参数的目的，是为了在输出信息里区分多个代码块，如果不想设置这个tag，可以考虑对这个上下文管理器进行修改，通过调用栈获得调用信息，准确的指出是哪个代码段的执行时长。

修改后的上下文管理器如下

```python
import time
import sys


class ProTime(object):
    def __init__(self, tag=''):
        frame = sys._getframe()
        tag_frame = frame.f_back
        self.lineno = tag_frame.f_lineno
        self.filename = tag_frame.f_code.co_filename
        self.tag = tag

    def __enter__(self):
        self.start_time = time.time()

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end_time = time.time()
        time_diff = self.end_time - self.start_time
        if self.tag:
            msg = "代码段{tag}运行时长{time_diff}".format(tag=self.tag, time_diff=time_diff)
        else:
            msg = "文件{filename} 第 {lineno} 行代码块执行时长{time_diff}".format(filename=self.filename, lineno=self.lineno, time_diff=time_diff)
        print(msg)


with ProTime('first') as pt:
    # 这里是你要统计运行时长的代码块
    time.sleep(1)

with ProTime() as pt:
    # 这里是你要统计运行时长的代码块
    time.sleep(2)


def test():
    with ProTime() as pt:
        # 这里是你要统计运行时长的代码块
        time.sleep(1)

test()
```
