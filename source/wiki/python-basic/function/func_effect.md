---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.7 函数存在的意义
order: 6
---

# 函数存在的意义

花草鱼鸟，都是自然造物，很难说哪一个事物的存在是必须的，哪一个事物的存在是没有必要的。而对于人造物，比如一门编程语言，它的语法和设计都是必须的，对于没有存在意义的设计，一定会被剔除掉。所有的主流编程语言，都有函数，这说明函数的存在是不可缺少的。

## 1. 代码重用

函数存在的一个非常明显的作用和意义就是代码重用。没有代码重用，编程人员就会被活活累死，费尽千辛万苦写出来的代码只能使用一次，有类似的功能需要完成时，不得不重头开始写起。下面就用一个特别简单的例子来向你阐明代码重用的重要性。

```python
import os

file_lst = os.listdir('/Users/kwsy/kwsy/coolpython')
py_file_lst = []

for file_name in file_lst:
    if file_name.endswith('.py'):
        py_file_lst.append(file_name)

print(py_file_lst)
```

上面这段代码可以获取指定文件夹下的所有文件名称，我使用一个for循环挑选出所有以.py结尾的文件，这样就获取到了指定目录下的所有python脚本。仅从功能实现上来看，代码没有任何问题，问题出在这段代码无法被重用。如果下一次你希望获取另一个文件夹下的python脚本，该怎么办呢，修改这段代码？ 如果下一次你想获取指定目录下的所有word文档，又该怎么办呢？

你应该注意到，至少有两个维度在发生变化时会导致这段代码无法被重用，这时，你应该想到函数

```python
import os


def get_file(file_path, suffix):
    file_lst = os.listdir(file_path)
    target_file_lst = []

    for file_name in file_lst:
        if file_name.endswith(suffix):
            target_file_lst.append(file_name)

    return target_file_lst


py_file_lst = get_file('/Users/kwsy/kwsy/coolpython', '.py')
print(py_file_lst)
```

一旦使用了函数，上面的问题就迎刃而解了，不管是文件夹发生了变化，还是想要获取的文件类型发生了变化，函数体里的代码都不需要修改，只需要在调用函数时传入你期望的参数就可以了。你写好了这段代码，其他人就可以通过引入模块的方式来调用你的代码。

## 2. 有助于我们思考

在函数设计上有一个原则，叫做单一职能原则，意思是说，一个函数只完成一个特定的功能。我以冒泡排序来向你解释什么叫做到单一职能原则，并向你展示函数是如何帮助我们思考问题的。

### 2.1 冒泡排序算法

冒泡排序是最简单的排序算法，它可以让一个无序的列表变得有序。

冒泡排序的核心思想是相邻的两个数据进行比较，假设数列A有n个数据，先比较第1个和第2个数据，如果A1 > A2,则交换他们的位置，确保较大的那个数在右侧。

接下来比较A2和A3，采用相同的规则，较大的数向右移动，最后会比较An-1 和An的大小，如果An-1 > An，那么交换他们的位置，这时，An就是数列中的最大值。

### 2.2 在指定范围内移动

你应该已经发现，经过一轮比较后，数列仍然是无序的，但是没有关系，你已经找到了最大值An，而且它在列表的末尾，我们定义个简单的函数

```python
def move_max(lst, max_index):
    """
    将索引0到max_index这个范围内的最大值移动到max_index位置上
    :param lst:
    :param max_index:
    :return:
    """
    for i in range(max_index):
        if lst[i] > lst[i+1]:
            lst[i], lst[i+1] = lst[i+1], lst[i]
```

在0到max_index这个范围内，函数move_max实现了冒泡排序的基本操作，到目前为止，我们还没有完整的实现冒泡排序，但是我们实现了冒泡排序的一个基本操作。这个函数的功能非常单一，单一的你都不知道这个函数究竟和冒泡排序有什么关系，别着急，继续往下看。

### 2.3 重复简单

在2.2 中，我们已经实现了一个简单的函数，咱们用它来做一个小实验

```python
lst = [4, 2, 1, 6, 3]
```

这有一个列表，长度是5，最大索引是4，执行一次move_max(lst, 4)后，列表的内容发生了变化

```python
lst = [2, 1, 4, 3, 6]
```

列表的最大值被移动到了最大索引4的位置上，OK，最关键，最核心的地方到了，现在，让我们忘记6，忘记lst[4]， 执行函数调用move_max(lst, 3)， lst[4]已经是列表里的最大值了，如果此时我调用执行move_max(lst, 3)，那么lst[3]不就变成了列表里第2大的数了么，沿着这个思路继续向，move_max(lst, 2)， move_max(lst, 1)，经过这一番操作，不就让列表变得有序了么？

```python
def pop_sort(lst):
    """
    实现冒泡排序
    :param lst:
    :return:
    """
    for i in range(len(lst)-1, 1, -1):
        move_max(lst, i)


def move_max(lst, max_index):
    """
    将索引0到max_index这个范围内的最大值移动到max_index位置上
    :param lst:
    :param max_index:
    :return:
    """
    for i in range(max_index):
        if lst[i] > lst[i+1]:
            lst[i], lst[i+1] = lst[i+1], lst[i]


if __name__ == '__main__':
    lst = [4, 2, 1, 6, 3]
    pop_sort(lst)
    print(lst)
```

函数的单一职能原则有助于我们抽丝剥茧，从混乱复杂的事物中找到解决问题的一小步，然后重复这一小步；或者找到多个一小步，之后组合这些小的步骤构建出一大步来解决更大的问题。
