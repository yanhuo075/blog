---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.8 运行时类型检查
order: 7
---

# python运行时类型检查

在上一篇教程[《使用mypy对python程序进行静态检查》](http://www.coolpython.net/python_senior/data_type/mypy.html)中，你已经学会了如何结合类型标注和mypy工具对代码进行静态类型检查，这种检查发生在程序执行之前，程序运行期间，仍然有可能发生实际传参的类型与预期不相符的情况，对此，你需要一种能在程序运行期间进行类型检查的方法，本文将教你如何编写一个类型检查的装饰器

## 1. 参数类型检查装饰器设计

请紧跟我的思路来设计这个可以检查函数参数类型的装饰器

### 1.1 装饰器带参数

首先，这个装饰器必须允许传入参数，这样，我们才能在使用装饰器修饰函数的时候指定参数的类型

```python
@typecheck(int, int)
def add(x, y):
    return x + y
```

### 1.2 获得被装饰的函数的形参列表

其次，必须在函数参数和我们所规定的参数之间建立起一个映射关系，指明参数x的类型是int， 参数y的类型是int, 在装饰器里，我们必须能够拿到被装饰函数的参数，这个倒是不难，使用inspect模块的signature方法就可以

```python
from inspect import signature

def add(x, y):
    return x + y

sig = signature(add)
print(sig, type(sig))
```

程序输出结果

```text
(x, y) <class 'inspect.Signature'>
```

### 1.3 建立映射关系

接下来，要在函数参数列表和我们装饰器里传入的参数之间建立映射关系， 这个可以使用bind_partial方法

```python
sig = signature(add)
print(sig, type(sig))

bound_types = sig.bind_partial(int, int).arguments
print(bound_types)
```

程序输出结果

```text
OrderedDict([('x', <class 'int'>), ('y', <class 'int'>)])
```

### 1.4 获取函数的实参列表

已经知道了函数的参数的约定类型，在实际执行代码时，还有能够获得函数实际传入的参数数值，也就是实参列表，这个可以通过bind方法来获得

```python
from inspect import signature

def add(x, y):
    sig = signature(add)
    bound_values = sig.bind(x, y)
    print(bound_values)


add(1, 4)
```

程序输出结果

```text
<BoundArguments (x=1, y=4)>
```

### 1.5 检查参数类型

1.3 获取到参数的约定类型，1.4获取函数执行时实际传入的实参，那么就可以进行类型检查了，当实际传入参数类型与约定不符时则抛出异常

## 2. 装饰器实现与使用

前面的设计，每一步都只解决一个小问题，现在，综合所有已经探索的技术来完成这个装饰器

```python
from inspect import signature
from functools import wraps


def typecheck(*type_args, **type_kwargs):
    '''
    类型检查装饰器, type_args和type_kwargs都是装饰器的参数
    :param type_args:
    :param type_kwargs:
    :return:
    '''
    def decorator(func):
        sig = signature(func)
        # 建立函数参数与装饰器约定参数类型之间的映射关系
        bound_types = sig.bind_partial(*type_args, **type_kwargs).arguments

        @wraps(func)
        def wrapper(*args, **kwargs):
            # 获得函数执行时实际传入的数值
            bound_values = sig.bind(*args, **kwargs)
            # 进行类型检查
            for name, value in bound_values.arguments.items():
                if name in bound_types:
                    if not isinstance(value, bound_types[name]):
                        raise TypeError(
                            'Argument {} must be {}'.format(name, bound_types[name])
                            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@typecheck(int, int)
def add(x, y):
    return x + y

add(3, 4.5)
```

使用该装饰器指定参数类型非常灵活，可以使用关键字参数

```python
@typecheck(int, z=float)
def test(x, y, z):
    print(x, y, z)

test(1, '2', 5.4)
test(1, 2, 5.4)
```

只用位置参数约定了函数的第一个参数类型必须是int，使用关键字参数z=float约定了参数z的类型必须是float,对于参数y没有任何要求，那么参数传入任何类型的数据都可以，装饰器不会进行检查
