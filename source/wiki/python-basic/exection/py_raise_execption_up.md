---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.10 捕获并再次抛出异常
order: 9
---

# python如何在捕获到一个异常后再次抛出异常

编写python代码时使用try ... except ... 可以捕获到代码中抛出的异常，在对异常进行处理后，有可能需要再次抛出这个异常，让上一级调用者继续处理，对于这样的要求，该怎样编写代码呢？

首先，在except语句块中，应当使用sys.exc_info()的详细信息，然后进行异常的处理，这里的处理完全是由你自己控制的，之后，根据要求，继续抛出这个异常，因为上一级调用者可能有自己对异常处理的逻辑，如果你这里不抛出异常，它那里就不知道有异常发生。

sys.exc_info() 返回的是一个元组，元组中的第一个元素是异常的类型，第二个元素是异常对象，第三个元素是traceback，你可以使用raise 直接抛出第二个元素，考虑到python2 与 3的兼容性问题，你也可以使用six模块的reraise函数抛出异常，示例代码如下

```python
import sys
import six


def raise_exception():
    raise ValueError(3)

def catch_exception():
    try:
        raise_exception()
    except ValueError:
        value = sys.exc_info()
        # do something
        six.reraise(*value)  # 借助six模块抛异常
        # raise value[1]   # 自己抛异常

try:
    catch_exception()
except:
    value = sys.exc_info()
    print(value)
```
