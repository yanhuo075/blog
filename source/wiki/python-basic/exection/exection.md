---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.2 错误和异常
order: 1
---

# 错误和异常

## 1. 错误

你经常会遇见的错误是语法错误，初学者掌握知识不扎实，经常会写出一些语法错误而不自知，一旦报错，就慌的一笔，下面是一个常见的错误语法示例

```python
a = “python”
print(a)
```

如果你经常犯语法错误，那么要认真的复习一下基础语法了，语法并不是需要动脑子思考的知识，你只需记住它就好。

不要老是说理解就好了，编程需要的是准确无误，每一行代码都不能出错，需要你做到精确的理解和记忆，这是所有工科专业的特点，文科专业，你大概理解一些，然后考试的时候自由发挥，自圆其说就好，但工科专业要的是准确无误的答案，可供你自由发挥的地方并不多，即便是实际应用时，你所谓的灵活也必须建立在准确无误的基础语法之上。

语法错误，在程序解析阶段就会被发现。

## 2. 异常

你的程序没有语法错误，但在运行期间仍然报错了，这种在运行期间发生的错误就是异常。

下面是几个比较典型的异常
示例1，ZeroDivisionError

```python
def test(a, b):
    print(a/b)

if __name__ == '__main__':
    test(10, 5)
    test(10, 0)
```

示例2, UnboundLocalError

```python
def test(a, b):
    if a > 5:
        c = 10
    print(b / c)


if __name__ == '__main__':
    test(6, 3)
    test(3, 2)
```

示例3 ，TypeError

```python
def test(a, b):
    print(a+b)


if __name__ == '__main__':
    test(1, 3)
    test('1', 3)
```

### 2.1 异常在什么时候发生？

仔细观察上面的三个异常示例，你会发现一个有趣的现象，异常并不总是发生，每一个例子中，第一次调用函数的时候，函数是可以正确执行的，但是第二次调用时，由于各种各样的原因，都发生了错误。

和语法错误不同，在程序解析阶段，异常不会发生，因为此时，程序还没有正式运行，而在运行时，也不一定就会报错，只有遇到特定情况时才会发生错误，所以，才叫它异常，大部分情况下，程序都是可以正常执行的，偶尔，由于某个特殊原因导致程序发生了错误。

一个程序员，在写程序的时候，要考虑很多，基于过往的经验，他会刻意的处理一些边界情况或者特殊情况，以避免发生错误。

## 3. 异常类继承关系

对于错误和异常，不要过于担心，他们并不是无穷无尽的，python的异常类是有限的，发生错误时耐心分析，认真总结，错误就会越来越少

```text
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
       |     +-- FloatingPointError
       |     +-- OverflowError
       |     +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
           +-- ModuleNotFoundError
      +-- LookupError
       |     +-- IndexError
       |     +-- KeyError
      +-- MemoryError
      +-- NameError
       |     +-- UnboundLocalError
      +-- OSError
       |     +-- BlockingIOError
       |     +-- ChildProcessError
       |     +-- ConnectionError
       |      |     +-- BrokenPipeError
       |      |     +-- ConnectionAbortedError
       |      |     +-- ConnectionRefusedError
       |      |     +-- ConnectionResetError
       |     +-- FileExistsError
       |     +-- FileNotFoundError
       |     +-- InterruptedError
       |     +-- IsADirectoryError
       |     +-- NotADirectoryError
       |     +-- PermissionError
       |     +-- ProcessLookupError
       |     +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
       |     +-- NotImplementedError
       |     +-- RecursionError
      +-- SyntaxError
       |     +-- IndentationError
       |          +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
       |     +-- UnicodeError
       |          +-- UnicodeDecodeError
       |          +-- UnicodeEncodeError
       |          +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```
