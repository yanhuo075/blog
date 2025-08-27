---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.11 自省
order: 10
---

# python的自省能力

## 1. 何为自省

自省是一种能力，检查某些事物以确定它是什么，知道什么以及能做什么。下面介绍python中那些提供强大自省能力的方法。

## 2. dir()

函数原型如下

```python
def dir(p_object=None):
```

dir以列表的形式返回参数p_object对象的属性名称，如果dir不传任何参数，则参数默认为当前所在模块对象。

这个函数在你学习第三方库的时候非常有用，如果第三方库没有详细的文档和示例，你可以阅读源码来了解第三方库的使用方法，也可以尝试用dir来获得第三方模块的方法名。

```python
import requests

print(dir(requests))
```

## 3. getattr, setattr, hasattr

我先用一个小例子来演示这三个内置函数如何使用，假设有一个类，定义了三个属性,此外还有一个request方法，发送http请求获得这三个属性的值，示例代码如下:

```python
class Demo(object):
    def __init__(self):
        self.name = None
        self.count = None
        self.price = None

    def request(self):
        # 假装发送了一个http请求,得到一个json格式的数据
        data = {'name': 'python', 'count': 10, 'price': 100}
        self.name = data['name']
        self.count = data['count']
        self.price = data['price']


demo = Demo()
demo.request()
print(demo.name)
print(demo.count)
print(demo.price)
```

这样写看起来没有任何问题，但现在条件发生了改变，类新增了许多属性，假设属性加起来有20个，那么这样的代码写起来就麻烦多了，此时，使用getattr和setattr就可以完美的解决这个问题。

```python
class Demo():

    def __init__(self):
        self.attrs = ['name', 'count', 'price']
        # 初始化
        for name in self.attrs:
            setattr(self, name, None)

    def request(self):
        # 假装发送了一个http请求,得到一个json格式的数据
        data = {'name': 'python', 'count': 10, 'price': 100}
        for key, value in data.items():
            setattr(self, key, value)


demo = Demo()
demo.request()
for name in demo.attrs:
    print(getattr(demo, name))
```

当Demo类新增属性时，只需要在attrs这个列表里新增属性名称即可，是不是方便快捷许多呢。
这三个函数的功能作用描述如下：

- def setattr(p_object, name, value) 将对象p_object的名为name属性赋值为value
- def getattr(object, name, default=None) 返回对象object 名为name属性
- def hasattr(p_object, name) 判断对象p_object 是否有名为name的属性

## 4. callable

可以用来测试对象是否可调用

```python
def test():
    return [123]

print callable(test)        
print callable(test())
print callable(test().append)
```

程序输出结果为

```text
True
False
True
```

test是函数，自然是可以调用的，test()返回的是列表，列表不可调用，test().append 等价于 [123].append，列表的append方法自然是可以调用的，所谓可以调用，就是指可以用()来表示执行。

## 5. 访问对象的元数据

如果你用dir去查看一个类或是一个函数的属性，那些明显看起来就不是你自己定义的属性，保存了对象的元数据

```python
def test():
    return [123]

print(dir(test))
```

程序输出结果为

```text
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', 
'__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', 
'__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', 
'__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict',
 'func_doc', 'func_globals', 'func_name']
```

这里面最常用的是__name__， 是函数的名称，下面的这段代码展示一个装饰器如何运用函数的__name__属性

```python
import time


def cost(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print('函数{name}执行时间为{seconds}'.format(name = func.__name__, seconds=t2-t1))
        return res

    return wrapper

@cost
def test(x, y):
    return x + y

print(test(4, 6))
```

程序输出结果为

```text
函数test执行时间为1.1920928955078125e-06
10
```

对于这个装饰器来说，需要输出函数的执行时间，那么在输出内容中就要指明，是哪个函数，示例代码中，通过调用__name__属性，就可以顺利的获得函数的名称。

## 6. inspect

inspect 是Python的标准库，提供了更加强大的自省能力，下面的示例代码，只向你展示其中的两个方法，其余方法，我会专门写一篇讲解inspect模块的教程

```python
import inspect

def test():
    print('ok')

print(inspect.getmodule(test))
print(inspect.getsource(test))
```

程序输出结果为

```text
<module '__main__' from '/Users/kwsy/kwsy/coolpython/demo.py'>
def test():
    print('ok')
```
