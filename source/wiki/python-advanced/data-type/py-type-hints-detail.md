---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.6 类型标注详解
order: 5
---

# 细致讲解python怎么做类型标注

- 细致讲解python怎么做类型标注

  - 1. 为变量做类型标注

    - 1.1 简单的数据类型
    - 1.2 使用 Optional
    - 1.3 使用Union
    - 1.4 为容器类型做标注
      - 1.4.1 为集合做标注
      - 1.4.2 为列表做标注
      - 1.4.3 为元组做标注
      - 1.4.4 为字典做标注
    - 1.4.5 容器类型标注总结

  - 2. 为函数做标注类型

    - 2.1 对形参和返回值进行标注
    - 2.1 对可变参数进行标注
    - 2.3 callable对象做参数

  - 3. 标注自定义类

    - 3.1 自定义类实例
    - 3.2 标注类属性

  - 4. 不常见的类型标注

    - 4.1 迭代器
    - 4.2 字典的items(), keys(), values()返回值
    - 4.3 Sequence

  - 5. 泛型和TypeVar工厂函数

尽管pyhton从3.5版本开始就引入了类型系统，但到目前为止，接受程度不是特别的高，很多的开源库仍然没有使用类型标注。

究其原因，我认为最主要的一条是类型标注不是必须的，且python解释器并不检查你所做的类型标注，那么大家在代码里添加类型标注的动力也就不大。

一项好的技术，没有被推广，单纯的说它不是必须的，不能解释所有的问题，另一个影响类型标注广泛使用的原因，我认为是类型标注有一点难度，官方文档没有尽全力为大家解释该如何使用。这使得很多想要使用类型标注的人望而却步，毕竟，即便费力的做了，也没有很明显很直观的收益。

本文将通过实际的例子为你展示如何在python代码里做类型标注。

## 1. 为变量做类型标注

我们先来通过最简单的情况，为变量做类型标注，变量可以有如下类型：

1. int
2. float
3. bool
4. str
5. bytes
6. None
7. list
8. tuple
9. set
10. dict

### 1.1 简单的数据类型

int, float, bool, str, None，bytes 这些都是最简单的数据类型，他们的类型标注也是最简单的，创建脚本typehint.py

```python
from typing import Optional, Union, Any

a: int = 8
b: bool = True
c: str = 'ok'
d: None = None
e: float = 9.8
f: bytes = b'32'
```

使用mypy对类型标注进行检查

```text
mypy typehint.py
```

检查结果

```text
Success: no issues found in 1 source file
```

这说明我们对这4种变量的类型标注是正确的，但上面的代码存在严重的缺陷，变量d我为它标注为None，那么d这个变量就永远只能为None了，如果我将其赋值为其他类型，类型标注检查就会报错, 修改代码

```text
from typing import Optional, Union, Any

a: int = 8
b: bool = True
c: str = 'ok'
d: None = None
e: float = 9.8
f: bytes = b'32'

d = 5
```

使用mypy检查结果

```text
typehint.py:9: error: Incompatible types in assignment (expression has type "int", variable has typ
e "None")
Found 1 error in 1 file (checked 1 source file)
```

修改后的代码可以正常执行，因为python解释器才不管类型标注呢，但是将5赋值给d就不符合类型标注的要求了。

**类型标注的意义是标注一个变量的数据类型，此后的代码都应当遵守对这个变量的类型标注，这就要求我们，不能随意的修改变量的数据类型。**

### 1.2 使用 Optional

在1.1 的例子中，d变量别标注为None类型，可一个变量始终赋值为None是毫无意义的事情，你只是在最初的时候不想给它一个明确的值才赋值为None的，后面的代码一定会修改变量d的值的。

假设你对变量d的使用是希望为它赋值一个int类型的数据，那么在类型标注的时候，就应当做好准备

```python
from typing import Optional, Union, Any

a: int = 8
b: bool = True
c: str = 'ok'
d: Optional[int] = None
e: float = 9.8
f: bytes = b'32'

d = 5
```

Optional表示可选，那么d就可以被赋值成int类型，此外也可以是None。

### 1.3 使用Union

d 能赋值成int，也可能被赋值成float， 这种情况，要结合Optional 和
Union

```python
from typing import Optional, Union, Any

a: int = 8
b: bool = True
c: str = 'ok'
d: Optional[Union[int, float]] = None
e: float = 9.8
f: bytes = b'32'

d = 5
d = 9.8
d = None
```

Union表示或的意思，d变量的类型，可以是None，也可以是int或者float。

接下来，你可能会问，可不可以将a变量的类型标注设置为Union[int, float]， 让a以赋值成int也可以赋值成为float？ 从纯粹的技术实现上讲这样做没有问题

```python
from typing import Optional, Union, Any

a: Union[int, float] = 8    # 坚决反对你这样做
b: bool = True
c: str = 'ok'
d: Optional[Union[int, float]] = None
e: float = 9.8
f: bytes = b'32'

d = 5
d = 9.8
d = None

a = 8.9
```

但从工程实践的角度来看，这种做法简直就是脱裤子放屁，多此一举。我们为变量进行类型标注的目的就是为了防止变量在使用过程中由于缺乏类型检查导致类型变来变去，你这样不就是又回到了之前的状态了么，那做类型标注还有什么意义呢，还不如不做。

d变量与其他几个变量不同，d变量初始值赋值为None，我们心里很清楚，它的值一定会被改变的，不然留着它毫无意义， 而一旦改变，就必然导致数据类型发生变化，因此才需要我们使用Optional。其他变量呢，值改变了，数据类型可以不发生变化，如果类型发生了变化，说明你的操作就违背了类型标注的初衷。

### 1.4 为容器类型做标注

list, tuple, dict, set， 为这4个容器类型数据做标注，要稍微麻烦一点点, 先来看最简单的set，

#### 1.4.1 为集合做标注

在使用set时，我们默认只会向集合中添加相同数据类型的值，但你要明确一点，集合可以存储不同类型的数据。

```python
from typing import Optional, Union, Any, Set

s: Set[int] = {1, 2, 3}
```

这段代码可以通过mypy的检查， 接下来看列表如何做标注

#### 1.4.2 为列表做标注

```python
from typing import Optional, Union, Any, Set, List, Tuple

s: Set[int] = {1, 2, 3}
l: List[int] = [1, 2, 3]
```

列表标注的方式与集合是一样的，但我们都清楚，列表里存储的数据往往都是类型不相同的，比如下面的列表

```python
 [1, 2, 3, 'a', 'b', True]
```

对这种情况，就需要使用1.3小节所介绍的Union

```python
from typing import Optional, Union, Any, Set, List, Tuple

s: Set[int] = {1, 2, 3}
l: List[Union[int, str, bool]] = [1, 2, 3, 'a', 'b', True]
```

#### 1.4.3 为元组做标注

为元组做标注，不能使用和列表相同的办法，而是要逐个索引位置进行标注

```python
from typing import Optional, Union, Any, Set, List, Tuple

t: Tuple[int, str, bool] = (3, 'ok', True)
```

#### 1.4.4 为字典做标注

先来看最简单的，字典的key都是字符串，value都是int

```python
from typing import Optional, Union, Any, Set, List, Tuple, Dict

d: Dict[str, int] = {'ok': 4}
```

这是最理想的情况，但实际情况往往更复杂，字典的key可以有str类型，也可以有int类型，当类型不确定的时候，我们就可以使用Union

```python
from typing import Optional, Union, Any, Set, List, Tuple, Dict

d: Dict[str, int] = {'ok': 4}
d1: Dict[Union[str, int], Union[str, int, float]] = {'ok': 4, 3: 'ok', 4: 3.2}
```

还有更复杂的情况

```python
from typing import Optional, Union, Any, Set, List, Tuple, Dict

dic: Dict[str, Union[Tuple[int, int], Dict[int, int]]] = {
    'ok': (1, 2),
    'dic': {5: 9}
}
```

字典里的value，可以是元组，也可以是字典，字典嵌套了字典，在做类型标注的时候，也就需要以嵌套的形式进行标注。对于这种复杂的字典，我的建议就是简化处理

```python
from typing import Optional, Union, Any, Set, List, Tuple, Dict

dic: Dict[str, Union[Tuple, Dict]] = {
    'ok': (1, 2),
    'dic': {5: 9}
}
```

value可以是元组，也可以是字典，我只要标注到这个程度就可以了，不再继续详细的进行标注，不然单单一个类型标注就把代码搞的难以理解了。

### 1.4.5 容器类型标注总结

容器类型标注，可以粗略的进行标注，也可以详细的进行标注，这完全取决于你的想法，我的观点是，在不影响代码可阅读性的前提下详细标注，反之则粗略标注

```python
from typing import Optional, Union, Any, Set, List, Tuple, Dict

l: List = [1, 2, ['2', '3']]            # 粗略标注
l2 : List[Union[int, List[str]]] = [1, 2, ['2', '3']]   # 详细标注
```

## 2. 为函数做标注类型

### 2.1 对形参和返回值进行标注

为函数做标注类型，需要对每一个形参做类型标注，同时还要对函数的返回值做类型标注

```python
def add(x: int, y: int) -> int:
    return x + y

print(add(2, 5))
```

形参的变量类型，我们事先是清楚的，因此你只需要按照第一节里的讲解对形参进行标注就可以了，函数的返回值在函数定义时进行标注，在有括号后面紧跟着进行标注，注意需要用到“->”。

如果返回值的类型可能是int，也可能是None，该怎么标注呢？其实这种情况完全可以参考对变量的标注

```python
from typing import Optional


def add(x: Optional[int], y: int) -> Optional[int]:
    if not isinstance(x, int):
        return None
    return x + y

add(3, 4)
add(None, 4)
```

看到这里你应该明白，对函数参数及返回值的标注，完全遵守对变量的标注规则，唯一需要区别对待的是函数的返回值。

### 2.1 对可变参数进行标注

python的可变参数一个是*args， 一个是**kwargs，从函数的视角来看，args的类型是元组，kwargs的类型是字典，先来看args

```python
def add(*args: int) ->int:
    sum_value = sum(args)
    return sum_value

print(add(1, 2, 3))
```

我很确定args里的元素都是int类型，那么直接标注为int就可以了，如果还有其他类型，那么就需要使用Union

```python
from typing import Optional, Union


def add(*args: Union[str, int, float]) -> float:
    sum_value = sum([float(item) for item in args])
    return sum_value

print(add(1, '2', 3.8))
```

传入的可变参数可以是str,int，float中的任意一个，args虽然是元组，但是我们不是按照元组来进行标注，标注的是对这些参数的期望值，再来看**kwargs

```python
from typing import Any, Union


def add(**kwargs: Union[int, str, float]) -> None:
    print(kwargs)

dic = {
    'a': 3,
    'b': '5',
    'c': 9.3
}

add(**dic)
add(a=3, b='5', c=9.3)
```

关键字参数的值，有int，str，float三个类型，我们要标注的是这些参数的值，而不是字典。

### 2.3 callable对象做参数

在python中，函数也是对象，也可以作为函数的参数

```python
from typing import Callable, Any, Union
import time
from functools import wraps


def cost(func: Callable):
    @wraps(func)
    def warpper(*args: Any, **kwargs: Any):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper

@cost
def test(sleep_time: Union[float, int]) -> None:
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


test(1)
```

当形参是函数对象时，使用Callable进行标注。

## 3. 标注自定义类

### 3.1 自定义类实例

在程序里自定义了一个类，对于这个类的实例，我们也可以标注

```python
class Demo():
    pass

d : Demo = Demo()

def test(demo: Demo):
    pass

test(d)
```

### 3.2 标注类属性

类属性可以使用ClassVar进行标注，标注后，如果实例尝试修改类属性，mypy在检查时会报错，但python解释器可以正常执行程序，原因前面已经强调过，解释器不受类型标注影响

```python
from typing import ClassVar

class Demo():
    count: ClassVar[int] = 0

d: Demo = Demo()

print(d.count)
d.count = 20  # mypy 检查会报错
```

## 4. 不常见的类型标注

有些对象的类型不如基础数据类型那样常见，我这里做一个总结并一一举例说明

### 4.1 迭代器

```python
from typing import Iterator

def my_generator(n: int) -> Iterator:
    index = 0
    while index < n:
        yield index
        index += 1

generate = my_generator(5)
```

my_generator是生成器函数，它的返回值是一个generator类型的对象，是一个迭代器，的返回值就可以标注为Iterator

### 4.2 字典的items(), keys(), values()返回值

字典的items()，keys()，values()三个方法分别返回字典的key-value对，所有的key和所有的values，标记他们类型的方法如下：

```python
from typing import ItemsView, KeysView, ValuesView


def test_1() -> ItemsView:
    dic = {'name': 'python'}
    return dic.items()


def test_2() ->KeysView:
    dic = {'name': 'python'}
    return dic.keys()


def test_3() ->ValuesView:
    dic = {'name': 'python'}
    return dic.values()
```

### 4.3 Sequence

Sequence 可以用来标记任何序列对象，比如列表，元素，字符串，字节串，他们都是序列，如果你对变量的类型不是很确定，但可以肯定它一定是一个序列，那么就可以使用Sequence

```python
from typing import Sequence, List


lst: Sequence[int] = []
name: Sequence = 'python'
tup: Sequence = (1, 2, 4.5)
bstring: Sequence = b'sds'
```

## 5. 泛型和TypeVar工厂函数

泛型和TypeVar工厂函数，都是为了更方便的进行类型标注而存在的。假设你现在要定义一个栈，Stack类，你需要一个列表来存储数据，此时，你会遇到一个难处，如果这个栈只允许int类型数据入栈，那么你就只能这样定义

```python
from typing import List

class Stack():
    def __init__(self):
        self.data: List[int] = []
```

但如果这个栈只允许float类型的数据入栈，你就只能这样来定义

```python
class Stack():
    def __init__(self):
        self.data: List[float] = []
```

这样就有点犯难了，两个存储不同数据类型的栈就需要两个定义，但这两个类的代码是完全一致的，只是类型标注不同，有没有什么办法，可以用一套代码实现不同类型的标注呢？

这就要用到泛型和TypeVar函数

```python
from typing import TypeVar, Generic, List


T = TypeVar('T')


class Stack(Generic[T]):
    def __init__(self):
        self.data: List[T] = []

    def push(self, item: T):
        self.data.append(item)

    def pop(self) -> T:
        return self.data.pop(-1)

    def top(self) -> T:
        return self.data[-1]

    def size(self) -> int:
        return len(self.data)

    def is_empty(self) -> bool:
        return len(self.data) == 0

stack = Stack[int]()
stack.push(3)
stack.push(5)
print(stack.pop())
```

我定义一个泛型，所谓泛型，就是先不明确它的类型，那么什么时候明确它的类型呢，等到实际调用的时候，比如

```text
stack = Stack[int]()
```

我在创建stack对象时来确定泛型T的数据类型，如果你希望栈只存储float类型数据，你就可以这样来写

```text
stack = Stack[float]()
```

使用泛型，相当于创建了一个模板，在调用模板前，来确定泛型的数据类型，一套代码，实现了多套数据类型标注，岂不美哉。
