---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.4 主动抛异常
order: 3
---

# 主动抛异常

## 1. 抛异常

有时，程序需要主动抛出异常，因为某些情况下，你需要反馈消息给更上层的调用者，告诉它有一些异常情况发生，而你抛出异常的地方，没有能力处理它，因此需要向上抛出异常。

这种情况为什么不让系统自己抛出异常呢？一个原因是上层的调用者本身就希望能够捕获有别于系统异常的自定义异常，二来，有些情况下，程序的逻辑是没有异常的，但是，从业务角度考虑，的确是一个不寻常的情况，因此需要我们主动抛出异常。

下面是抛出异常的一个例子

```python
def divide(x, y):
    if y == 0:
        raise ZeroDivisionError("0不能做分母")
    return x/y

if __name__ == '__main__':
    divide(10, 5)
    divide(10, 0)
```

抛出异常时，你可以指定抛出哪个异常,如果你不想指定，那么可以抛出异常Exception， 它是所有异常的父类

```python
def divide(x, y):
    if y == 0:
        raise Exception("0不能做分母")
    return x/y

if __name__ == '__main__':
    divide(10, 5)
    divide(10, 0)
```

## 2. 自定义异常类

在程序里引入自定义的异常类，可以让代码更具可读性，同时对异常的划分更加精细，那么在处理异常时也就更加具有针对性，自定义异常继承自Exception，或者那些类本身就继承自Exception

```python
import requests


class HttpCodeException(Exception):
    pass

def get_html(url, headers):
    res = requests.get(url, headers=headers)
    print(res.status_code)
    if res.status_code != 200:
        raise HttpCodeException

    return res.text

try:
    text = get_html("http://www.coolpython.net", {})
    print(text)
except HttpCodeException:
    print("状态码不是200")
```
