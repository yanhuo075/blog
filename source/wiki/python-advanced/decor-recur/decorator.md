---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 6.1 装饰器
order: 0
---

# 装饰器导读

python的装饰器本质上是一个函数，得益于一切皆对象的语言设计理念，在python中，函数也是对象，因此可以在函数内部定义一个函数并将其返回，这便是装饰器的基本原理。

想要掌握装饰器，必须先理解变量作用域的概念，将函数视为与int, float相同级别的对象，当我们使用装饰器去修饰一个函数时，其实就是在装饰器中把目标函数当成int，float类型的数据来使用，理解到这里，结合实际装饰器的例子，你就能明白装饰器的基本原理了。

除了可以使用函数来实现装饰器，也可以使用类来实现装饰器，基于类的装饰器本质上还是使用了类的__call__方法，因此，并没有超出装饰器就是一个函数的基本设定。

编写装饰器，的确不是一件容易的事情，本章节最后会向你介绍一个第三方模块decorator，使用这个模块，可以非常轻松的实现一个装饰器，这要比你从头编写要简单很多。



# 变量作用域

python变量的作用域取决于变量被赋值的位置，python中，只有当变量在模块，类，函数中定义的时候才会有作用域的概念。一共有四种作用域: 局部作用域, 嵌套作用域, 全局作用域, 内置作用域

理解变量作用域是学习装饰器的关键之一，理解了作用域，也为理解更高级的语法与技术奠定基础。

## 1 变量的作用域

### 1.1 局部作用域

在函数中创建的变量，是局部变量，当函数被执行时，会创建一个局部作用域，这些局部变量只能在这个作用域内使用，出了这个作用域就无法使用了

```python
def test():
    value = 100
    print(value)

print(value)
```

在函数外，就不能再访问value这个变量了，上面的代码执行时会报错

```text
NameError: name 'value' is not defined
```

原因在于最后一行print语句是在全局作用域里执行的，但这个作用域里没有value这个变量。

### 1.2 嵌套作用域

```python
def test():
    value = 100
    def test_2():
        value2 = 99
        print(value2, value)
    return test_2

a = test()
a()
```

如果一个函数的内部又定义了一个函数，那么这样就产生了嵌套作用域，其实嵌套作用域是一个相对概念。变量value存在于test函数所产生的局部作用域，变量value2存在于test_2函数所产生的局部作用域，value所在的作用域相对于value2来说就是一个嵌套作用域。

在函数test_2中可以访问test函数中的变量，但反过来就不行, 只允许向上访问。函数最后返回值是test_2，它是一个函数，赋值给a以后，a()就是在执行test_2函数

### 1.3 全局作用域

每个模块都是一个全局作用域，你也可以简单的理解为每个脚本就是一个全局作用域。在模块中创建的变量，这个模块里的任何地方都可以访问得到
示例1

```python
value = 100

def test():
    print(value)

test()
```

在局部作用域里，你可以使用这个变量，但你不能修改这个变量的值
示例2

```python
value = 100

def test():
    value = 99
    print(value)

test()
print(value)
```

程序的输出结果是

```python
99
100
```

道理很简单，当你执行value = 99时，其实是在局部作用域里创建了一个变量value，这个value和全局的那个value根本就不是同一个变量。

那么为什么，在示例1中，就可以访问到全局的那个value呢，这是因为在搜索变量时是按照顺序进行的

局部作用域 > 嵌套作用域 > 全局作用域 > 内置作用域

python会按照这个顺序搜索变量，示例1中现在局部作用域搜索，没有找到后去全局作用域，这时找到了。

如果想在函数里修改全局变量，必须使用global关键字

```python
value = 100

def test():
    global value
    value = 99
    print(value)

test()
print(value)
```

### 1.4 内置作用域

系统内固定模块里定义的变量，如预定义在builtin 模块内的变量

## 作用域所产生的诡异行为

```python
a = 3
b = 5
sum = a + b
print(sum)

lst = [1, 2, 3]
print(sum(lst))
```

上面这段代码执行会报错

```text
TypeError: 'int' object is not callable
```

这是一个非常诡异的错误，很多初学者都曾经遇到过，看代码，看不出有任何问题，可就是报了一个奇怪的错误。问题出在代码的第三行 sum = a + b。 sum 是一个变量，它的值等于a + b，sum这个变量存在于全局作用域里，最后一行的代码试图通过内置函数sum求列表里的元素之和，内置函数的作用域是内置作用域，python在搜索变量时是按照

1. 局部作用域
2. 嵌套作用域
3. 全局作用域
4. 内置作用域

的顺序进行的，最后一行代码想要使用内置函数sum，但是由于全局作用域里已经有了一个sum变量，就不会去内置作用域里继续寻找了。全局作用域里sum的值 a+b = 8，是一个int类型数据，int类型数据当然不是callable的。



# 装饰器作用与原理

装饰器是python的一个神器，它赋予函数在不修改自身代码的情况下改变自身行为的能力，装饰器的基本原理是python函数可以接受一个函数作为参数并返回一个函数。

## 1. 装饰器的作用

当你编写了一段程序，发现程序运行的很慢，调试过程中，你需要知道程序执行过程中每一个函数的执行时间，以此来确定哪里需要优化性能，通常的做法是在函数开始的时候记录一次时间，在函数结束的时候记录一次时间，这两个时间的差值就是函数的执行时间。

这样做的确可以实现目标，但却有一个缺点。你不得不修改函数，如果函数调用很多，你不得不一一修改，结束优化后，你不得不删除掉那些添加的用来记录时间的代码。

下面是一个普通的函数

```python
def test_decorator():
    time.sleep(1)
    print("执行函数test_decorator")
```

如果按照刚才讲的方法，你需要把函数修改成这样

```python
def test_decorator():
    t1 = time.time()
    time.sleep(1)
    print("执行函数test_decorator")
    t2 = time.time()
    print("执行耗时" + str(t2 - t1))
```

修改以后，代码变得臃肿，而且结束优化后，你要删除掉刚才所添加的3行代码

## 2. 用装饰器来改变函数行为

现在，我们写一个简单的装饰器，改变函数test_decorator的行为，让它具备记录自身执行时间的能力，装饰器代码如下

```python
def cost(func):
    def warpper():
        t1 = time.time()
        res = func()
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper
```

cost 是一个函数，入参func是一个函数，返回值warpper是一个函数， 使用非常简单

```python
def test_decorator():
    time.sleep(1)
    print("执行函数test_decorator")


test = cost(test_decorator)
print(test)
test()
```

输出结果为

```text
<function cost.<locals>.warpper at 0x118a31840>
执行函数test_decorator
test_decorator执行耗时1.0051071643829346
```

将test_decorator作为参数传给cost函数，cost函数返回的warpper也是一个函数，而这个函数里执行了test_decorator并且记录了函数的执行时间，对于test_decorator这个函数，我们没有对它做任何改变，但却得到了它的执行时间。

上面的实例，并不是工作中使用的方法，我创建了一个test变量，并将cost的返回值赋值给他，是为了让你清晰的看到装饰器是如何工作的，装饰器的工作原理就是将被装饰的函数放入到一个新的函数中执行，这个新的函数是你自己编写的，因此，你可以做任意你想做的事情来实现自己想要的功能却不需要改变被装饰的函数。

实际工作中，我们这样使用装饰器

```python
@cost
def test_decorator():
    time.sleep(1)
    print("执行函数test_decorator")


test_decorator()
```

使用@cost装饰一个test_decorator等价于test = cost(test_decorator)，好处时无需再定义一个变量test，当你不需要记录函数的执行时间时，只需要移除@cost即可



# 完善装饰器细节

## 1. 传递参数

上一篇实现了一个监测函数执行时间的装饰器，但这个装饰器存在缺陷，它只能装饰那些没有参数的函数，下面的用法就是有问题的

```python
import time


def cost(func):
    def warpper():
        t1 = time.time()
        res = func()
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper


@cost
def test(sleep_time):
    time.sleep(sleep_time)


test(1)
```

程序报错，原因在于test函数有一个sleep_time参数，但是在执行res = func()时，却没有传递这个参数，对装饰器稍作修改，修改后代码如下

```python
def cost(func):
    def warpper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper
```

在函数warpper中增加可变参数，在调用执行被装饰的函数时传递这些可变参数，这样就不会有问题了，现在，这个装饰器可以装饰任何你想装饰的函数

## 2. 修复函数

### 2.1 自省信息

函数被装饰以后，一些原本属于自己的自省信息会丢失，先来看装饰前的样子

```python
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```

执行输出结果

```text
test

    测试装饰器
    :param sleep_time:
    :return:
```

### 2.2 自省信息丢失

程序会输出函数的名称和函数的注释doc信息，但是被装饰以后，这些信息就会丢失

```python
@cost
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```

程序输出结果为

```text
warpper
None
```

### 2.3 修复丢失信息

可以看到，实际输出的都是warpper的自省信息，这是我们不希望看到的，为此，我们要使用一项修复技术

```python
import time
from functools import wraps


def cost(func):
    @wraps(func)
    def warpper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper

@cost
def test(sleep_time):
    """
    测试装饰器
    :param sleep_time:
    :return:
    """
    time.sleep(sleep_time)


print(test.__name__)
print(test.__doc__)
```

functools模块有一个wraps函数,它可以避免函数被装饰后丢失自省信息，增加@wraps(func)即可，这样，程序最后输出的就test自身的信息



# 装饰器带参数

装饰器携带参数，可以实现更加强大的装饰器，本篇教程以http请求重试场景来讲解如何编写带参数的装饰器。

## 1. 重试场景

假设这样一个场景，你写了一个函数，用来从一个API接口获取数据，那么你必须考虑网络问题，也许网关解析会出错，也许API接口压力太大，短时间内响应超时，面对这种情况，你的函数应该具备重试功能。

请求失败以后，暂停一定时间，再次发起请求，重试的次数必须是可设置的。你可以把这个功能在函数中实现，但可以肯定，这不是一个好方法，原因在于每一个这样的函数你都需要实现一遍，而且不同的函数访问不同的API，暂停的时间以及重试的次数都有可能不同。

这种请求，就非常适合用装饰器来解决

## 2. 带参数装饰器

```python
from functools import wraps


def retry(retry_count=5, sleep_time=1):
    def wrapper(func):
        @wraps(func)
        def inner(*args, **kwargs):
            for i in range(retry_count):
                try:
                    res = func(*args, **kwargs)
                    return res
                except:
                    time.sleep(sleep_time)
                    continue
            return None
        return inner
    return wrapper
```

这段代码一共嵌套着定义了3个函数，在讲解作用域的时候强调过，嵌套作用域是一个相对概念，在函数inner里可以访问更上层的局部作用域里的数据，retry_count是函数retry的一个参数，在inner里根据retry_count来决定重试的次数，只要发生了异常，就会重试。

## 3. 如何使用

写爬虫时，你不得不考虑网络因素，一次请求可能无法得到所需要的响应数据，这时就需要重试

```python
class HttpCodeException(Exception):
    pass
    
@retry()
def get_html(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.87 Mobile Safari/537.36',
        'Host': 'movie.douban.com',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3',
        'Referer': 'https://movie.douban.com/top250?start=25&filter='
    }

    res = requests.get(url, headers=headers)
    if res.status_code != 200:
        raise HttpCodeException

    return res.text
```

这段代码是我编写过的一个爬虫的一小部分，在其他场景下，只要涉及到重试，都可以使用我写的这个装饰器。



# 基于类的装饰器

## 1. __call__

前面介绍的装饰器都是用函数写的，其实，装饰器也可以用类来定义编写

用类写装饰器，就要用到 __call__ 方法，这个方法，允许你像调用函数一样去调用对象，下面是一个简单的示例来展示这种用法

```python
class TestCall(object):
    def __call__(self, *args, **kwargs):
        print("执行了call方法")

tc = TestCall()
tc()
print(callable(tc))
```

程序执行结果为

```text
执行了call方法
True
```

tc是一个对象，如果没有实现__call__方法，那么tc()这种写法是有问题的，但是由于实现了__call__方法，tc就是一个callable的对象了

这种技术在python的web框架里非常常见，__call__ 赋予了类对象和函数一样的被调用的能力，直白点说，函数和对象可以实现混用，因为他们都能被采用小括号的方式去调用

## 2. 类装饰器

下面是一个类装饰器的简单例子

```python
import time


class Decorator(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        t1 = time.time()
        res = self.func(*args, **kwargs)
        t2 = time.time()
        print("函数执行时长:"+ str(t2 - t1))


@Decorator
def test():
    time.sleep(1.5)

test()
```

为了便于理解，下面的代码不采用@ 这种方式来进行装饰

```python
f = Decorator(test)
f()
```

test函数作为参数初始化Decorator对象，此时，f.func = test, f是一个对象，但是由于实现了__call__方法，因此，可以直接像调用函数那样去调用对象， f(),此时，__call__方法被执行



# 一行代码，轻松编写装饰器

一行代码能轻松编写装饰器么，当然不能，但实际的工作量和一行代码真的没有太多区别，先看一个常规方法编写的装饰器

```python
def cost(func):
    def warpper(*args, **kwargs):
        t1 = time.time()
        res = func(*args, **kwargs)
        t2 = time.time()
        print(func.__name__ + "执行耗时" +  str(t2-t1))
        return res
    return warpper

@cost
def test_cost(sleep):
    time.sleep(sleep)


test_cost(2)
```

很多人被装饰器的闭包概念搞的死去活来，不能理解一切皆对象的概念，别说写一个装饰器，就是看一个装饰器都不能理解其中之意。

得益于python强大的社区，有人写出了一个模块，可以让你轻松写出一个装饰器，该模块安装命令为 pip3 install decorator

使用起来，极为简单

```python
import time
from decorator import decorator


@decorator
def cost(func, *args, **kw):
    t1 = time.time()
    res = func(*args, **kw)
    t2 = time.time()

    print(func.__name__ + "执行耗时", t2-t1)
    return res

@cost
def test_cost(sleep):
    time.sleep(sleep)


test_cost(2)
```

cost函数被decorator装饰，cost就变成了一个装饰器，第一个参数必须传入函数，就是你希望被cost装饰的函数，*args 和 **kw 是用来传给func的参数，如果你想写一个带参数的装饰器，同样很简单

```python
import time
from decorator import decorator


@decorator
def cost(func, timelimit=3, *args, **kw):
    t1 = time.time()
    res = func(*args, **kw)
    t2 = time.time()
    # 只打印运行时长超过timelimit的函数
    if (t2-t1) > timelimit:
        print(func.__name__ + "执行耗时", t2-t1)
    return res

@cost
def test_cost(sleep):
    time.sleep(sleep)


test_cost(3)
```



# python使用装饰器实现缓存

本文将和你一同开发一个缓存装饰器，届时，你将了解缓存的一般算法，对于编写装饰器也更加得心应手

## 1. 缓存算法

经典的缓存算法有3个：

1. FIFO算法
2. LFU算法
3. LRU算法

### 1.1 FIFO算法

FIFO（First in First out），先进先出， 该算法的核心原则是： 如果一个数据最先进入缓存中，则应该最早淘汰掉，当缓存容量满了以后，应当将最早被缓存的数据淘汰掉。FIFO算法是一种比较简单的算法，使用队列就可以轻易的实现。

### 1.2 LFU算法

LFU（Least Frequently Used）最近最少使用算法， 这个算法的核心在于：**如果一个数据在最近一段时间内使用次数很少，那么在将来一段时间内被使用的可能性也很小**。

### 1.3 LRU算法

LRU (Least Recently Used)， 最近最久未使用算法，该算法的核心原则是：**如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小**

LFU算法和LRU算法乍看起来是一个意思，但其实很不同，LRU的淘汰规则是基于访问时间，而LFU是基于访问次数的。

一个缓存的数据，一段时间内被命中很多次，这个数据在LFU算法里会被保留，但在LRU算法里则可能被淘汰，虽然这段时间内，比如2分钟内被命中了很多次，可是，这些事情都发生在1分50秒之前的10秒钟里，自那以后就再也没有被命中，LRU算法则可能会将其淘汰。

## 2. 装饰器实现

3种算法里，最容易实现的是FIFO算法，因此这个装饰器使用FIFO算法来实现。

使用一个列表来保存函数的参数，并且规定这个列表的最大长度，缓存不能无限增加。使用一个字典，以参数做key， 以函数返回结果做value。

```python
from functools import wraps
from inspect import signature


def fifo_cache(maxsize=128):
    lst = []
    cache = {}
    def wrapper(func):
        sig = signature(func)       # 获得参数列表
        @wraps(func)
        def inner(*args, **kwargs):
            bound_values = sig.bind(*args, **kwargs)        # 绑定参数
            # print(bound_values)
            key = bound_values.__str__()
            value = cache.get(key)

            if value is not None:
                print('命中缓存')
                return value

            if len(lst) >= maxsize:
                oldkey = lst.pop(0)
                if oldkey in cache:
                    cache.pop(oldkey)

            result = func(*args, **kwargs)
            lst.append(key)
            cache[key] = result
            return result

        return inner

    return wrapper


@fifo_cache()
def test1(x, y):
    return x + y


@fifo_cache()
def test2(x, y, z=20):
    return x + y + z

@fifo_cache()
def test3(*args, **kwargs):
    return 5

print(test1(19, 20))

print(test2(19, 20, 20))
print(test2(19, 20))        # 不会命中缓存

print(test3(4, 2, x=6, y=9))
print(test1(19, 20))
```

代码执行输出结果

```text
39
59
59
5
命中缓存
39
```

在第二次调用test1(19, 20)时，命中了缓存

### inspect.signature

使用inspect的signature函数可以获取函数定义的参数的顺序以及函数注解

### sig.bind

signature返回的sig是类Signature的实例对象，该对象的bind方法返回一个BoundArguments对象，下面是其中一个示例

```text
<BoundArguments (args=(4, 2), kwargs={'x': 6, 'y': 9})>
```

方法在实际传入的参数和函数的参数列表之间建立了一个映射关系

### 装饰器的缺陷

这个装饰器还存在着一个小的缺陷，个别情况下无法命中缓存，函数test2的最后一个参数z是默认参数，默认值是20， test2(19, 20, 20) 与 test2(19, 20) 本质上传入的参数是相同的,但在inner函数里接收到的参数里没有默认参数的数值，这里一定有办法可以解决这个问题，暂且搁置，待我慢慢研究。
