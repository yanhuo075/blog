---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.10 函数杂谈
order: 9
---

# 函数杂谈

# 不要用可变类型对象做函数默认参数

## 1. 可变对象做默认参数

内置数据类型int，float，bool，str，tuple 是不可变对象， 字典，集合，列表是可变对象。

在定义python函数时，千万不要使用可变类型对象作为函数的默认参数，那样会引发意想不到的错误，下面的代码向你展示这种错误

```python
def add_int(value, data=[]):
    data.append(value)
    return data

result = add_int(3)
print(result)  # [3]
```

我定义了一个add_int函数，默认参数data是一个列表，程序执行结果是[3], 看起来没有什么问题，但是在第2次执行add_int函数时，就会出现意想不到的情况

```python
def add_int(value, data=[]):
    data.append(value)
    return data

result = add_int(3)
print(result)

result = add_int(4)
print(result)
```

程序输出结果是

```text
[3]
[3, 4]
```

第2次输出的结果并不是预期的[4]， 而是包含了第一次调用函数add_int函数时传入的参数3，这究竟是怎么一回事呢？

## 2. 函数默认参数

在定义函数时，默认参数只会被计算一次，而不是每次调用函数的时候才计算。python脚本在执行过程中，遇到函数定义的代码，会进行编译，创建一个代码对象，完成编译后，python会将默认参数存储在函数对象之中，准确的说是存储在__defaults__属性中，此后每次调用函数所使用的默认参数都是最开始计算的那一个，而由于这个默认参数是可变对象，所以会导致上面所描述的诡异问题。

```python
def add_int(value, data=[]):
    data.append(value)
    return data

print(add_int.__defaults__)
```

通过输出__defaults__， 我们可以参看函数的默认参数，程序输出结果是一个元组([],) ， 刚刚定义完的函数里，默认参数data还是一个空列表，执行过一次以后，就发生变化了

```python
def add_int(value, data=[]):
    data.append(value)
    return data

print(add_int.__defaults__)

add_int(3)
print(add_int.__defaults__)
```

程序输出结果

```text
([],)
([3],)
```



# 追踪函数调用关系

在我们执行拥有复杂函数调用关系的一段python程序时，我们希望能够清楚的知道他们之间的调用关系以及在调用过程中传入的参数信息和返回值，这些信息对于我们分析程序的行为和bug会很有帮助。

我希望能实现一个装饰器，用于追踪函数的调用关系，下面是这个装饰器的简单实现

```python
import sys
from functools import wraps


class FuncTrace:
    indent = 0
    def __init__(self, stream=sys.stdout, indent_step=2, is_show_res=True):
        self.steam = stream
        self.indent_step = indent_step
        self.is_show_res = is_show_res

    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            curr_indent = ' '*FuncTrace.indent
            # 参数信息
            arg_lst = [repr(item) for item in args]
            kwarg_lst = ['{key}={value}'.format(key=key, value=value) for key, value in kwargs.items()]
            arg_str = ','.join(arg_lst + kwarg_lst)
            msg = "{curr_indent}{func_name}({arg})\n".format(curr_indent=curr_indent, func_name=func.__name__, arg=arg_str)
            self.steam.write(msg)

            # 下一次调用时, 增加缩进
            FuncTrace.indent += self.indent_step
            result = func(*args, **kwargs)
            FuncTrace.indent -= self.indent_step    # 调用结束后减少缩进

            if self.is_show_res:
                msg = "{indent}return {result}\n".format(indent=curr_indent, result=result)
                self.steam.write(msg)

            return result
        return wrapper
```

调用函数时的参数信息十分关键，因此将args和kwargs拼接在一起输出，不足之处在于，如果参数里的数据内容较多，比如一个参数是列表，列表里有超过100个元素，那么现有的这种写法会将这100个元素都输出，很影响输出结果的展示，这是一个可以优化的点。

通过在输出信息中增加缩进，来表现函数之间的调用关系，在函数执行前增加缩进，函数执行结束后减少缩进。

默认输出流是sys.stdout， 你也可以将它设置为文件或者log， 下面是这个装饰器的用法展示

```python
def func_c(number):
    if number % 3 == 0:
        return number**2

    return number*2

@FuncTrace()
def func_b(number):
    if number % 2 == 0:
        return number*2
    else:
        return func_c(number)

@FuncTrace()
def func_a(lst):
    result = []
    for i in lst:
        result.append(func_b(i))

    return result


lst = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
func_a(lst)
```

程序输出结果

```text
func_a([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  func_b(1)
    func_c(1)
    return 2
  return 2
  func_b(2)
  return 4
  func_b(3)
    func_c(3)
    return 9
  return 9
  func_b(4)
  return 8
  func_b(5)
    func_c(5)
    return 10
  return 10
  func_b(6)
  return 12
  func_b(7)
    func_c(7)
    return 14
  return 14
  func_b(8)
  return 16
  func_b(9)
    func_c(9)
    return 81
  return 81
  func_b(10)
  return 20
return [2, 4, 9, 8, 10, 12, 14, 16, 81, 20]
```



# python令人迷惑的形参与实参

你所熟知的各种编程语言，都有形参与实参这个概念，虽然不同语言对于形参和实参的要求是不同的，但他们存在的形式与意义却是一致的。

对于许多初学者来说，形参与实参，着实难以区分和理解，这不怪你，因为计算机里的很多概念，的确是难以理解。我们对于日常生活中许多事物的理解，已经让我们养成了思维上的习惯与定式，而这种定式放在计算机编程上，往往是失效的。原因在于，计算机里的许多概念，都是没有实体的。

比如说，内存，你不能指着某样东西说“看，这是内存”， 充其量，你拆开电脑，指着一块硬件说“看，这是内存条”，然尔内存与内存条是两个完全不同的事物。所以，我们要放弃自然科学中那种对事物进行精确定义和描述的习惯。

看下面这段代码

```python
def add(x, y):
    return x + y
```

x, y ,就是函数add的形参，形参这个概念，侧重于函数的定义，这段代码里，没有实参，因为实参侧重于函数的调用，看下面这段代码

```python
def add(x, y):
    return x + y

add(3, 4)
```

3 和 4 就是调用函数add时传入的实参，这句话强调了调用函数和传入两个动作，只有当这两个动作发生时，才有实参的概念，或者说，实参的概念才有意义。

那么，return x + y 这行代码里，x 与 y 是实参，还是形参呢？ 我个人的见解是，在函数里再去辨别他们已经没什么意义，如果一定需要一个答案，那么，我认为它们是形参，而不是实参。实参，是一个只在函数调用时存在的概念，具体到函数内部执行时，我们已经不关心实参这个事情了。

如果你对变量作用域理解的不深刻，那么下面这段代码很容易让你困惑

```python
x = 3
y = 4

def add(x, y):
    return x + y

add(x, y)
```

现在再来看，return x + y 这行代码里，x y 到底是实参还是形参？答案是形参。实参，不是一个具体的事物，它是一个概念，这个概念只在调用函数传入参数时才有意义。

为什么要有形参和实参这两个概念呢？形参，规定了函数的样式，是一种形式的约定，强类型语言还会约定一个形参的类型。实参，是实际调用时传入函数的数据，因此叫实参。

实参与形参，是两个维度的事物，一个强调形式，一个强调实际数值，因此我说，他们是两个维度的事物，形参是静态的概念，实参是动态的概念，我执行add(3, 5)， 函数的实参就是3, 5。下一次执行add(7, 8)，那么函数的实参就是7和8，但形参始终都是x和y。

在函数内，形参的值，是由实参决定的，在函数执行之前，你不知道x 和 y 的值是什么，只有函数实际被执行，被调用，在函数内，你才知道形参的实际数值是什么。



# python函数的强制关键字参数

如果你希望函数的某些参数强制使用关键字参数进行传递，那么在定义函数时，可以在位置参数结束后添加一个*， *后面的参数就是强制关键字参数。

```python
def func(name: str, age: int, *, address='上海') -> str:
    return f"{name}今年{age}岁，住在{address}"


print(func("小明", 10, address='北京'))
```

- name, age 是位置参数，在age后面的* 并不是参数，它的作用是标识后面的参数都是强制关键字参数
- 调用函数时，address必须使用关键字参数进行传递

使用强制关键字参数在某些情况下会带来非常大的好处，比如一个函数拥有几十个参数的情况，如果都使用位置参数，那么函数在调用时，你很难准确的理解某一位置的参数含义和作用，毕竟参数多大几十个，你无法记住每一个位置参数的作用。但使用强制关键字参数，在函数调用时，就必须使用关键字参数，就像上面的示例，你必须在函数调用时传入关键字参数address，这样做表意更加清晰。

在实际应用中，uvicorn的run函数就使用强制关键字参数

```python
def run(
    app: ASGIApplication | Callable[..., Any] | str,
    *,
    host: str = "127.0.0.1",
    port: int = 8000,
    uds: str | None = None,
    fd: int | None = None,
    loop: LoopSetupType = "auto",
    ****中间省略
    )
```

在你调用uvicorn.run函数时，除了app这个位置参数外，其他的都是强制关键字参数，你必须准确的理解你所传入的参数的含义和作用，在python引入强制关键字参数之前，上面的函数可能会这样定义

```python
def run(
    app: ASGIApplication | Callable[..., Any] | str,
    **kwargs
    )
```

**kwargs 是可变参数，两者在调用时，其形式是一样的，都可以写成如下的形式

```python
uvicorn.run(app, host="0.0.0.0", port=8800)
```

看似没有区别，但实际操作时你会明白，run函数使用强制关键字参数比使用可变参数更能提高编程体验，如果使用了强制关键字参数，编辑器例如pycharm可以提示run函数有哪些关键字参数可以传递，而如果使用可变参数，pycharm则无法提供这样的信息，你只能通过阅读源码中函数的文档来了解可变参数**kwargs可以传入哪些关键字参数名称。



# python的函数也可以实现重载

## 1. 借助functools.singledispatch实现重载

python的函数是一等公民，原则上或者说实质上并不像C++和java一样支持函数重载，但是可以借助functools.singledispatch装饰器来实现类似于函数重载的功能。functools.singledispatch 是Python的一个装饰器，它允许你将一个普通的函数变成一个单分派泛函数（generic function）。这意味着基于第一个参数的类型，你可以为同一个函数定义多个实现。它是Python对单分派泛型函数的支持，这种函数可以根据单个参数的类型来决定调用哪个具体的实现。

在讲解如何实现python函数的重载之前，有必要先解释重载所能解决的问题，下面的函数将根据参数的类型来实现不同的逻辑

```python
def calc(x:Any, y:Any) -> Any:
    if isinstance(x, int):
        print("x和y都是int")
    elif isinstance(x, float):
        print("x和y都是float")
    else:
        raise NotImplementedError(f"calc not implemented for {type(x)} and {type(y)}")
    return x + y
```

在函数内部，通过对参数x的类型进行判断来决定走哪种处理逻辑，随着x类型的增加，if 语句的逻辑分支也就会增多，函数的复杂度会提升，降低了代码的可维护性，重载就能很好的解决这个问题，重载后的函数可以根据参数的类型自动决定调用某个具体的实现。

下面是一个示例

```python
from functools import singledispatch
from typing import Any



@singledispatch
def calc(x: Any, y: Any) -> Any:
    raise NotImplementedError(f"calc not implemented for {type(x)} and {type(y)}")


@calc.register
def calc_int(x: int, y: int) -> int:
    print("x和y都是int")
    return x + y


@calc.register
def calc_float(x: float, y: float) -> float:
    print("x和y都是float")
    return x + y


calc(1, 2)

calc(3.4, 5.6)
```

- 首先，我用singledispatch装饰器装饰函数calc，calc函数有两个参数，类型我标注为Any
- 使用calc.register装饰器装饰函数calc_int 和 calc_float
- 实际调用时，不是调用calc_int 或者 calc_float，而是调用calc， 具体实际执行哪个函数由传入的参数来决定，执行calc(1, 2) 时，调用的是calc_int函数，而执行calc(3.4, 5.6) 时调用的是calc_float函数，注意，是根据第一个参数的类型

如果传入的参数与注册的函数都不匹配，则会执行calc函数

```python
calc('', 3)
```

程序会抛异常

```plain_text
NotImplementedError: calc not implemented for <class 'str'> and <class 'int'>
```

前面的示例中，被calc.register装饰的函数的参数都有清晰的类型标注，如果你不喜欢编写类型标注，也可以退而求其次，在register中约定类型，代码可以改写成下面的样子

```python
from functools import singledispatch
from typing import Any



@singledispatch
def calc(x: Any, y: Any) -> Any:
    raise NotImplementedError(f"calc not implemented for {type(x)} and {type(y)}")


@calc.register(int)
def calc_int(x, y) -> int:
    print("x和y都是int")
    return x + y


@calc.register(float)
def calc_float(x, y) -> float:
    print("x和y都是float")
    return x + y


calc(1, 2)

calc(3.4, 5.6)
```

对于被calc.register装饰的函数，函数名称可以是相同的，因此你也可以写成下面的样子

```python
@calc.register(int)
def _calc(x, y) -> int:
    print("x和y都是int")
    return x + y


@calc.register(float)
def _calc(x, y) -> float:
    print("x和y都是float")
    return x + y
```

## 2. 使用开源库multipledispatch

functools.singledispatch只能基于被装饰函数的第一个参数的类型来实现简单的派发，还不能算是真正的重载，想要根据函数的所有参数进行派发，可以使用开源库multipledispatch

```plain_text
pip install multipledispatch
```

使用示例如下

```python
from multipledispatch import dispatch

@dispatch(int, int)
def calc(x, y) -> int:
    print("x和y都是int")
    return x + y


@dispatch(float, float)
def calc(x, y) -> float:
    print("x和y都是float")
    return x + y

@dispatch(float, int)
def calc(x, y):
    print("x是float，y是int")
    return x + y

calc(1, 2)

calc(3.4, 5.6)

calc(3.3, 5)
```

执行calc(3.3, 5) 时，会调用最后一个版本的calc的实现，singledispatch 无法实现这样的功能。



