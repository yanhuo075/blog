---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.6 递归函数
order: 5
---

# 递归函数

## 1. 什么是递归函数

递归函数是一种特殊的函数，函数内部会调用函数自身，现在，我将《函数的参数》这篇教程中的一个函数例子改造成递归函数，你可以对比他们之间的区别。

```python
def my_print(content, count):
    for i in range(count):
        print(content)

my_print('ok', 2)
```

改造成递归函数后

```python
def my_print(content, count):
    print(content)
    if count == 1:
        return
    my_print(content, count-1)

my_print('ok', 2)
```

在改造后的递归函数中，会再次调用my_print函数自身。你不禁会担忧，这样自身调用自身，岂不是要形成死循环？没错，会形成死循环，因此递归函数一定要有一个递归终止的条件，在本示例中，递归终止的条件是count=1，当这个条件满足时，就会执行return 语句，函数退出。

## 2. 4个步骤，写出一个递归函数

递归函数对初学者来说，是无法回避的噩梦。刚刚对程序顺序执行有了一定理解后，突然间告诉你函数可以调用函数自身，近似无限循环的递归调用让你深陷其中无法自拔。

编程，既是一门知识，也是一门手艺，你可以自学知识，但手艺很难自学，没有长时间的经验积累，没有一次次的试错和总结，你的技能就无法得到提升，所以，手艺，需要有人传帮带。那么今天，就让专业的python工程师教你如何用4个步骤写出一个递归函数。

咱们以最简单的计算阶乘为例，将完成递归函数归纳为4个步骤

1. 定义函数功能
2. 找到终止条件
3. 甩锅
4. 反甩锅

### 2.1 第一步，定义函数功能

不管最终能不能写出完整的递归函数，你至少应该有能力完成函数定义啊，这就好比数学考试，题不会做，"解" 字你总会写吧

```python
def recursion(number):
    """
    计算number的阶乘
    :param number:
    :return:
    """
```

### 2.2 第二步，找到终止条件

所有的递归函数，一定会有终止条件，如果不符合这个要求，那么就会进入死循环。函数的功能是计算阶乘，阶乘的终止条件是什么呢？ 是1啊，1的阶乘是1，你不能计算0的阶乘，1是最小的可以求阶乘的整数，现在，假设调用函数传参number为1，代码该怎么写呢？

```python
def recursion(number):
    """
    计算number的阶乘
    :param number:
    :return:
    """
    if number == 1:
        return 1
```

函数的功能是计算number的阶乘，如果number等于1，直接返回1就好了

### 2.3 第三步，甩锅

如果number不是1，该怎么办呢？答案很简单，甩锅，这就是说，现在你搞不定这个事情了，那你可以把锅帅给别人啊，你找到小明，告诉他： 老师让我计算number的阶乘，你现在计算number-1的阶乘，把结果告诉我，要快，不然老师生气了。

你看，把锅甩给了小明，小明如果能算出来number-1的阶乘，你把他的答案乘以number，不就是number的阶乘了么，如果小明算不出来，你对老师也能有交代，是小明的过错，这就叫甩锅，这就是函数的递归调用

```python
def recursion(number):
    """
    计算number的阶乘
    :param number:
    :return:
    """
    if number == 1:
        return 1

    next_recursion = recursion(number-1)   # 等待小明的结果
```

### 2.4 第四步，反甩锅

你把锅甩给小明，小明也不傻，他又把锅甩给了小刚，小刚表示很无辜，但事情还是要做，于是又甩给了小红，就这样，一个接着一个的甩，但是要注意，总会有甩不出去的时候，最后，锅甩到了小芳这里，小芳是个好姑娘，聪明乖巧，到她这里，需要计算1的阶乘，小芳心想，甩不出去了，而且也不用甩啊，1的阶乘就是1啊，多简单。

于是，开始了第四个步骤，反甩锅，1的阶层等于1，小芳心想，我已经告诉你答案了，剩下的事情你们处理吧，这口锅沿着一开始的路线反向的甩了回来，每个人都得到了之前自己甩锅的人的答案，现在小明把number-1的阶乘告诉给你，你应该怎么办呢，你应该把结果乘以number，然后return这个结果，最终完成了老师的要求

```python
def recursion(number):
    """
    计算number的阶乘
    :param number:
    :return:
    """
    if number == 1:
        return 1

    next_recursion = recursion(number-1)
    return next_recursion*number


if __name__ == '__main__':
    print(recursion(4))
```

上面的代码，没有一个字符超出你的学习范围，就是知识；以怎样的逻辑组织代码，理解代码结构，这就是手艺
