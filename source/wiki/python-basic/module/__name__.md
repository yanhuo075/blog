---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 8.2 模块引入与执行
order: 1
---

# if __name__ == '__main__'

```python
def func():
    print('ok')


if __name__ == '__main__':
    func()
```

这种形式的代码，相信你已经见过很多，在脚本的末尾，出现一个if条件判断，这个if条件判断的作用是什么呢， __name__ 事先并没有定义，为什么可以直接使用呢，它从哪里来？ 回答这些问题，必须先了解模块属性

## 1. 模块属性

一个python脚本（以.py 结尾的文件）就是一个模块，模块自身有若干属性，其中比较常用的是如下两个

1. __name__ 模块的名称
2. __file__ 脚本的完整文件路径

在任意一个python脚本里，你都可以输出这两个属性

```text
print(__name__)
print(__file__)
```

得到结果

```text
__main__
/Users/zhangdongsheng/experiment/test/test.py
```

__name__ 的值是 __main__ ，这表示模块的名称是"__main__", __file__ 是文件的完整路径

虽然弄清楚了__name__ 是怎么一回事，但也带来了新的疑问，明明__name__ 就等于 __main__ ，为何还要做if条件判断呢？显然是存在 __name__ 不等于__main__ 的情况。

## 2. 直接执行与其他模块引入

简单的功能，我们可以在一个python脚本里完成，但复杂的功能，我们会写多个python 脚本，比如下面的例子里，有两个脚本，一个是main.py ，做为整个程序的启动脚本，utils.py 提供一些辅助函数，供main.py使用

（1）utils.py

```python
def safe_division(a, b):
    if b == 0:
        return None

    return a/b

print("utils 模块里的__name__ 值为：", __name__)
```

(2)main.py

```python
from utils import safe_division

def func():
    value = input("输入两个整数，中间用空格分开:")
    arrs = value.split()
    a = int(arrs[0])
    b = int(arrs[1])

    result = safe_division(a, b)
    print(result)


if __name__ == '__main__':
    func()
```

接下来，通过两步实验，来理解 __name__ 在不同场景下的取值情况。

第一步，先来执行utils.py文件

```text
python utils.py
```

执行结果为

```text
utils 模块里的__name__ 值为： __main__
```

第二步，执行main.py

```text
python main.py
```

执行结果为

```text
utils 模块里的__name__ 值为： utils
输入两个整数，中间用空格分开:5 2
2.5
```

这里有一个现象，你必须理解其背后的原因，我们明明执行的main.py脚本，但是utils.py脚本里的代码也被执行了，这是因为在main.py脚本里引入了utils.py 这个模块，被引入的脚本里的代码会在引入时执行。

当utils.py 被其他脚本引入时，它的__name__ 就不等于__main__， 而是等于utils,恰好是文件名称去掉.py 剩余的部分。

经过上面的实验，我们可以得出两个结论

1. 当脚本被直接执行时，模块属性__name__ 等于__main__
2. 当脚本被其他模块引入时，模块属性__name__ 等于脚本名称去掉.py 后剩余的部分

3、终极目的---测试模块里函数
由于一个脚本被引入时，自身的代码会被执行，因此我们在每个脚本里都写上一段if __name__ == '__main__': 如果你希望一些代码只有在脚本被直接执行时才执行，那么就把这些代码放入到if 语句块中，最常见的情形就是测试代码,下面我对utils.py 进行修改

```python
def safe_division(a, b):
    if b == 0:
        return None

    return a/b

if __name__ == '__main__':
    print(safe_division(10, 5) == 2)
    print(safe_division(10, 0) == None)
```

我们写完一个函数后，不免要写一些测试的代码，而这些测试的代码我们不希望他们在utils.py被引入时执行，只有当我们主动执行utils.py 进行测试才执行这些测试代码
