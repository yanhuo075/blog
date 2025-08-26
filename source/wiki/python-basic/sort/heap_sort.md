---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.9 堆排序
order: 8
---

# 堆排序

## 堆的概念

堆是一种数据结构，分最大堆和最小堆，最大（最小）堆是一棵每一个节点的键值都不小于（大于）其孩子（如果存在）的键值的树。大顶堆是一棵完全二叉树，同时也是一棵最大树。小顶堆是一棵完全完全二叉树，同时也是一棵最小树。

我们用list来描述它

```python
lst = [9, 6, 8, 3, 1, 4, 2]
```

你看不出这个lst有什么特别，别着急，再介绍两个概念给你

### 父节点，子节点

列表中的每一个元素都是一个节点，以lst[0] 为例，他的子节点分别是lst[1],lst[2]，同时我们也说lst[1]的父节点是lst[0]

我们可以计算每一个节点的子节点，假设当前节点的序号是i，那么它的左子节点则是 i*2 +1,右子节点则是i*2 + 2

### 最大堆，最小堆

所谓最大堆就是指每一个节点的值都比它的子节点的值大，最小堆就是指每一个节点的值都比它的子节点的值小

现在，我们再来看上面给出的列表

lst[0] = 9,它的子节点分别是lst[1]=6,lst[2]=8

lst[1] = 6,它的子节点分别是lst[3]=3,lst[4]=1

lst[2] = 8,它的子节点分别是lst[5]=4,lst[6]=2

lst[3] = 3,它的子节点分贝是lst[7]和lst[8]，但这两个节点是不存在的

后面的也就不用再看了，这个列表符合最大堆的要求，父节点的值大于两个子节点的值，而且最重要的一点，堆中任意一颗子树仍然是堆

## 如何建立一个堆

关于堆的应用，非常多，比如堆排序，在应用之前，我们必须先建立一个堆，刚才给出的列表，恰好是一个堆，如果不是堆呢，我们需要将其变成堆，例如下面这个列表

```python
lst = [3, 9, 2, 6, 1, 4, 8]
```

这个列表里的元素和上一个列表里的元素是一样的，只是顺序不同，建立堆的过程，就是调整顺序的过程，使其满足堆的定义

### 不是所有节点都有子节点

如果当前节点的位置是i,那么子节点的位置是i*2 + 1 和 i*2 +2 ，因此，不是所有节点都有子节点，假设一个堆的长度为n，那么n/2 - 1 及以前的节点都是有子节点的，这是一个非常简单的算数题，你稍微用脑就能理解。

那么建立堆的过程，就是从n/2 - 1 到0 逐渐调整的过程，如何调整呢？

每个节点都和自己的两个子节点中最大的那个节点交换位置就可以了，这样，节点值较大的那个就会不停的向上调整

### 示例代码

```python
def adjust_heap(lst, i, size):
    lchild = 2 * i + 1      # 计算两个子节点的位置
    rchild = 2 * i + 2
    max = i
    if i < size // 2:
        if lchild < size and lst[lchild] > lst[max]:
            max = lchild
        if rchild < size and lst[rchild] > lst[max]:
            max = rchild

        # 如果max != i成立,那么就说明,子节点比父节点大,要交换位置
        if max != i:
            lst[max], lst[i] = lst[i], lst[max]
            # 向下继续调整,确保子树也符合堆的定义
            adjust_heap(lst, max, size)

def build_heap(lst):
    for i in range((len(lst)//2)-1, -1, -1):
        adjust_heap(lst, i, len(lst))
        print lst   # 此处输出,可以观察效果

lst = [3,9,2,6,1,4,8]
build_heap(lst)
print lst
```

## 最大堆，最小堆

关于最大堆，最小堆，我们只要掌握一点就好了，对于最大堆，堆定的元素一定是整个堆里最大的，但是，如果我们去观察，整个堆并不呈现有序的特性，比如前面建立的堆

```text
[9, 6, 8, 3, 1, 4, 2]
```

堆顶元素为9，是最大值，但是从0到最后一个元素，并不是有序的

### 堆排序的思路

lst = [9, 6, 8, 3, 1, 4, 2]

（1）将lst[0] 与 lst[6]交换，交换后为[2, 8, 6, 4, 3, 1, 9],现在，这个堆已经被破坏掉了

（2）那么我们可以利用adjust_heap函数重新把它调整成一个堆，adjust_heap(lst,0,6)，这样调整后，lst=[8, 6, 4, 3, 1, 2, 9]

注意看lst[0]到lst[5],这个范围内的数据被调整成了一个堆，使得lst[0]也就是8是这个范围内的最大值

我们只需要重复刚才的两个步骤，就可以将堆的大小逐步缩小，同时，从后向前让整个lst变得有序

### 示例代码

```python
def adjust_heap(lst, i, size):
    lchild = 2 * i + 1      # 计算两个子节点的位置
    rchild = 2 * i + 2
    max = i
    if i < size // 2:
        if lchild < size and lst[lchild] > lst[max]:
            max = lchild
        if rchild < size and lst[rchild] > lst[max]:
            max = rchild

        # 如果max != i成立,那么就说明,子节点比父节点大,要交换位置
        if max != i:
            lst[max], lst[i] = lst[i], lst[max]
            # 向下继续调整,确保子树也符合堆的定义
            adjust_heap(lst, max, size)

def build_heap(lst):
    for i in range((len(lst)//2)-1, -1, -1):
        adjust_heap(lst, i, len(lst))
        print lst   # 此处输出,可以观察效果


lst = [3,9,2,6,1,4,8]
def heap_sort(lst):
    size = len(lst)
    # 建立一个堆,此时lst[0]一定是最大值
    build_heap(lst)
    print '*'*20
    for i in range(size-1, -1, -1):
        # 将lst[0] 与lst[i] 交换,这样大的数值就按顺序排到了后面
        lst[0], lst[i] = lst[i], lst[0]
        # 交换后,重新将0到i这个范围内的数据调整成堆,这样lst[0]还是最大值
        adjust_heap(lst, 0, i)
        print lst,i  # 此处输出可以查看效果

heap_sort(lst)
print '*'*20
print lst
```
