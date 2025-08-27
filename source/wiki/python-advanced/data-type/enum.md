---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.1 python3枚举
order: 0
---

# python 枚举（enum）

## 1. 枚举模块enum

从3.x开始python提供了enum模块来提供枚举的功能，在使用时通过from enum import Enum 来引入。开发人员需要自己定义一个继承Enum的类来实现枚举类型对象。python的枚举是使用类来实现的，类属性是枚举名称，属性值对应枚举值。Enum的使用有如下特点：

1. 枚举类不允许定义相同枚举名称，但不同的枚举名称可以有相同的值，后者相当于前者的别名。
2. 枚举值不能被修改，枚举值一旦被修改，就会引发AttributeError异常。
3. 两个不同的枚举类，枚举名称和枚举值即便相同，在比较时也是不相等的。
4. 枚举类的一个枚举有name（标签）和value（枚举值）两个属性，使用枚举值时，务必通过value获取枚举值。

## 2. 枚举应用场景

枚举有什么作用呢？当一个变量有几种固定的取值时，通常我们喜欢将它定义为枚举类型，枚举类型用于声明一组命名的常数，使用枚举类型可以增强代买的可读性。

假设有这样一个函数

```python
def print_color(color_code):
    if color_code == 1:
        print('红色')
    elif color_code == 2:
        print('蓝色')
    elif color_code == 3:
        print('黑色')
```

参数color_code取值有3种，1表示红色，2表示蓝色，3表示黑色。color_code是表示颜色的代码，只有3种取值，这种情形下就适合使用枚举类型来表示，在python没有枚举类型之前，可以使用类来定义枚举类型。

```python
class ColorCode:
    RED = 1
    BLUE = 2
    BLACK = 3

def print_color(color_code):
    if color_code == ColorCode.RED:
        print('红色')
    elif color_code == ColorCode.BLUE:
        print('蓝色')
    elif color_code == ColorCode.BLACK:
        print('黑色')
        
print_color(1)
```

函数里不再用color_code和1，2，3这些整数值进行比较，而是与ColorCode的类属性进行比较，代码可阅读性更好，因为只看1，2，3，你无法理解这些数字所代表的含义。虽然使用类可以模拟枚举类型，但这种技术有一个缺点，类属性可以随意修改

```python
ColorCode.RED = 4   
```

枚举类型要求一旦完成定义，就不能再修改，否则使用枚举的地方将由于枚举值的改变出现不可知的问题。

## 3. Enum使用示例

python3 提供了enum模块，定义类时继承enum.Enum，可以创建一个枚举类型数据，除此以外还可以继承enum.IntEnum，枚举值只能是int。

**下面的代码演示如何通过继承Enum定义一个枚举类**

```python
import enum


class ColorCode(enum.Enum):
    RED = 1
    BLUE = 2
    BLACK = 3


def print_color(color_code):
    if color_code == ColorCode.RED.value:
        print('红色')
    elif color_code == ColorCode.BLUE.value:
        print('蓝色')
    elif color_code == ColorCode.BLACK.value:
        print('黑色')

print_color(1)
```

看上去和第2小节里的代码没有什么大的区别，但由于继承了enum.Enum，ColorCode的类属性将无法修改，如果执行

```python
ColorCode.RED = 4
```

将会引发错误

```text
raise AttributeError('Cannot reassign members.')
AttributeError: Cannot reassign members.
```

枚举值不能被修改，是使用枚举类型进行编程的最重要的目的之一，假设枚举值可以被修改，那么也就没有必要提供enum这个模块了，我们使用自定义类和类属性就能够替代enum模块。

## 4. 枚举值

### 4.1 枚举值唯一

枚举值理论上是允许重复的，如果不希望出现枚举值重复的情况，可以使用enum模块提供的unique装饰器

```python
import enum
from enum import unique

@unique
class ColorCode(enum.Enum):
    RED = 1
    BLUE = 1
    BLACK = 3
```

如果枚举值出现了重复的情况，由于有unique装饰器修饰，在执行时会报错。

### 4.2 枚举值遍历

使用for循环可以对枚举值进行遍历，枚举有name和value两个属性，name就是枚举类的类属性，value则是类属性的值。

```python
import enum
from enum import unique

@unique
class ColorCode(enum.Enum):
    RED = 1
    BLUE = 2
    BLACK = 3


for color in ColorCode:
    print(color.name, color.value)
```

程序输出结果

```text
RED 1
BLUE 2
BLACK 3
```

### 4.3 枚举值比较

两个枚举值之间只支持身份运算符is和比较运算符==进行比较。以4.2 小结的代码为例演示枚举值如何进行比较。

```python
print(ColorCode.RED == ColorCode.RED)       # True
```

这看起来没有什么特别，但如果使用枚举和对应的值进行比较，就会产生一些出乎意料的结果

```python
print(ColorCode.RED == 1)       # False
```

这是非常容易出错的地方，很多人相当然的认为ColorCode.RED与1是相等的，但真实的结果却是False，RED是一项枚举，枚举有name和value两个属性，必须通过value才能获得真实的枚举值

```python
print(ColorCode.RED.value == 1)       # True
```

或者换一个思路，将枚举值转成枚举类型

```python
print(ColorCode.RED == ColorCode(1))       # True
```

ColorCode(1)) 的结果正是ColorCode.RED。
