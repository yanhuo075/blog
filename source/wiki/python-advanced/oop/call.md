---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.5 __call__()
order: 4
---

# python面向对象 __call__方法

python中, 一个类，如果实现了__call__方法，那么这个类的实例就是callable对象（可调用对象）,__call__方法允许你像函数一样被调用, 这在程序设计上会带来非常大的便利性

## 1. 把对象伪装成函数

python中, 一个类，如果实现了__call__方法，那么这个类的实例就是callable对象（可调用对象），下面用一个示例来解释这段话

```python
class Stu:
    def __call__(self, *args, **kwargs):
        print('ok')

stu = Stu()
print(callable(stu))
stu()       # 像调用函数一样使用stu
```

程序输出结果

```python
True
ok
```

如果类Stu没有实现__call__方法，执行stu(), 则会报错

```text
TypeError: 'Stu' object is not callable
```

## 2. 应用场景

使用__call__方法，将类的实例对象伪装成函数，这种技术在python中被广泛使用

### 2.1 基于类实现装饰器

编写基于类的装饰器，就必须用到__call__方法

```python
from functools import wraps

class SafeAdd:
    def __init__(self, func):
        self.func = func       # func是被装饰的函数

    def __call__(self, *args, **kwargs):
        if len(args) == 2:
            left = args[0]
            right = args[1]         # 获取参数并进行类型转换
            if not isinstance(left, (int, float)):
                left = float(left)

            if not isinstance(right, (int, float)):
                right = float(right)

            return left + right

        return None

@SafeAdd
def add(a, b):
    return a + b


print(add(3, 5))
print(add('3', '6'))
```

上面的代码还不足以阐述说明__call__方法在编写装饰器时所产生的作用，换一种写法，虽然我们正式使用装饰器时很少这样写，但更容易理解

```python
def add(a, b):
    return a + b

safe_add = SafeAdd(add)     # add函数作为参数传入SafeAdd的初始化函数\__init__中
# safe_add是类SafeAdd的实例
print(safe_add(3, 5))
print(safe_add('3', '6'))
```

实例对象safe_add由于实现了__init__方法，因此可以像使用函数一样通过一对小括号来调用执行。

### 2.2 实现WSGI协议

我们所使用flask， tornado， django 等web框架，无一例外的遵守WSGI协议，WSGI是一个协议，是一份约定，它规定服务器和应用程序之间如何传递数据，各自应该实现什么样的接口，以便彼此间配合工作。对于应用程序段，它只规定了3个简单的要求

1. 应用程序是一个可调用对象（callable）
2. 可调用对象接受两个参数，分别是environ和start_response
3. 可调用对象需要返回一个可迭代对象

第一条明确规定，应用程序必须是一个可调用对象，下面是一个简单的flask应用示例代码

```python
from flask import Flask
app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```

app就是应用程序，它是类Flask的实例，app是可调用对象，深入Flask类的源码可以发现，Flask类实现了__call__方法

```python
def __call__(self, environ, start_response):
    """The WSGI server calls the Flask application object as the
    WSGI application. This calls :meth:`wsgi_app` which can be
    wrapped to applying middleware."""
    return self.wsgi_app(environ, start_response)
```
