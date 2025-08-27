---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 2.4 匹配
order: 3
---

# python正则表达式精讲---match

## 1. match的一般用法

match方法的作用是从字符串起始位置开始匹配一个模式，如果模式不匹配，则返回None

咱们用一个简单的例子来熟悉一下match的用法

```python
import re

pattern = re.compile('www')
res = pattern.match('www.coolpython.net')
if not res is None:
    print(res.span())  # 输出匹配的起始位置和结束位置
```

compile方法对正则表达式进行编译，生成一个正则表达式对象，使用这个对象调用match方法。

除了这种写法之外，你可以使用另一种写法，他们最终的效果是相同的

```python
import re


res = re.match('www', 'www.coolpython.net')
if not res is None:
    print(res.span())  # 输出匹配的起始位置和结束位置
```

这两种写法，本质上是相同的，从源码就可以看得出来，re.match方法的源码如下

```python
def match(pattern, string, flags=0):
    """Try to apply the pattern at the start of the string, returning
    a match object, or None if no match was found."""
    return _compile(pattern, flags).match(string)
```

先调用_compile函数进行编译，之后调用正则表达式对象的match方法，而第一段代码里使用的compile方法源码则是这样的

```python
def compile(pattern, flags=0):
    "Compile a regular expression pattern, returning a pattern object."
    return _compile(pattern, flags)
```

返回的正是_compile所编译的正则表达式对象，在本教程中，均使用第一种写法，预先编译好正则表达式对象。

## 2. 从起始位置匹配模式，并提取子表达式内容

上面的例子实在是过于简单了，让你怀疑match方法的作用和字符串的startswith方法是一样的啊。看似一样，但很不相同，startswith方法纯粹的判断一个字符串是否以某个字符串开头，而match则是从起始位置匹配模式，关键在于模式二字。

假设有这样一个列表

```python
lst = [
    '小明的学号是3918324， 小班',
    '小红的学号是39384344， 中班',
    '小刚的生日是7月15日， 大班'
]
```

现在需要你使用正则表达式，将学生的姓名和学号提取出来，观察一下数据，很容易发现规律，符合提取要求的字符串都是以某人的姓名开头，后面紧跟”的学号是“，学号部分都是纯数字，数字的个数是不固定的，我们可以使用正则表达式，匹配这部分，并且提取姓名和学号

```python
import re

lst = [
    '小明的学号是3918324， 小班',
    '小红的学号是39384344， 中班',
    '小刚的生日是7月15日， 大班'
]

pattern = re.compile('(.+)的学号是(\d+)')
for item in lst:
    res = pattern.match(item)
    if res:
        print(res.groups())
        print(res.group(1), res.group(2))
```

使用match方法，可以匹配更为复杂的模式，这一点是字符串startswith方法所不能比拟的。在正则表达式中，有两个小括号，看过前面的教程，你应该知道这叫分组，也叫子表达式，使用match方法，返回的结果如果不是None，则可以使用groups方法获得所匹配到的子表达式里的内容，你也可以使用group方法，指定组号，需注意的是，组号要从1开始，使用组号0得到的是所匹配的整段字符串。
