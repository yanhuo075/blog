---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.3 捕获异常
order: 2
---

# 捕获异常

一旦发生异常，程序就会终止，这是非常糟糕的事情，这种糟糕体现在两方面

1. 即便发生了异常，业务上可以忽略它，那么程序应当继续执行
2. 程序终止，使得异常的信息没有被保留下来，不利于问题的分析和总结

为了提高程序的健壮性和解决问题，可以将异常捕获，根据业务要求来做对应的处理

## 1. try

python中，捕获异常使用try ... except ...这种语法来捕捉异常，下面是一个异常捕获的示例

```python
def test(a, b):
    try:
        print(a/b)
    except ZeroDivisionError:
        print("0不能作分母")

if __name__ == '__main__':
    test(10, 5)
    test(10, 0)
```

那些你担心不安全的代码，就可以放在try子句中，也就是try和except之间。

## 2. except

except 可以指定想要捕获的异常，比如上面的例子中，代码想捕获ZeroDivisionError 异常，如果try子句中发生了别的异常，这个except 就不会捕捉。

这样做，是希望能够针对不同的异常做不同的处理，你可以一次性指定想要捕获的所有异常，比如下面的代码

```python
def test(a, b):
    try:
        print(a/b)
    except (ZeroDivisionError, ValueError):
        return None

if __name__ == '__main__':
    test(10, 5)
    test(10, 0)
```

也可以逐个捕捉

```python
def test(a, b):
    try:
        print(a/b)
    except ZeroDivisionError:
        print('0不能做分母')
    except ValueError:
        print("类型错误")
    else:
        print('什么异常都没发生')

if __name__ == '__main__':
    test(10, 5)
    test(10, 0)
```

try ... except ... else这种语法，当没有异常发生时，就会执行else语句块里的代码。

如果你对于异常还不熟悉，不知道该捕获哪些异常，则可以用下面的写法

```python
def test(a, b):
    try:
        print(a/b)
    except:
        print('发生异常')

if __name__ == '__main__':
    test(10, 5)
    test(10, 0)
```

如果你什么异常都不指定，那么任何异常都会被捕捉，但这不是一个好的写法，因为这意味着，你根本不了解你的程序，因为不了解，你连可能发生的异常是什么都不清楚，这是很致命的，你不了解你自己写出来的程序，还怎么指望它能正常工作？

## 3. finally

你可以在finally语句块里做清理操作，因为不论try子句里是否发生异常，也不论你在except语句块里做了什么操作，总之，最终一定会执行finally语句块里的代码，这就保证了这里的代码最后一定会被执行，所以，清理收尾的工作一定会进行。

你可以在这里输出日志，你可以做任何你想做的事情。

```python
def divide(x, y):
    try:
        result = x / y
    except ZeroDivisionError:
        print("division by zero!")
    else:
        print("result is", result)
    finally:
        print("executing finally clause")

if __name__ == '__main__':
    divide(10, 5)
    divide(10, 0)
```
