---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.6 快速排序
order: 5
---

# 快速排序

快速排序的思路可以归结为3个步骤

1. 从待排序数组中随意选中一个数值，作为基准值
2. 移动待排序数组中的元素，是的基准值左侧的数值都小于等于它，右侧的数值大于等于它
3. 基准值将原来的数组分为两部分，针对这两部分，重复步骤1，2， 3

通过分析，可以确定，必然要使用递归算法，遇到递归不要害怕，先把1，2两步分区实现，最后实现第3步的递归

先实现分区

```python
def partition(lst,start,end):
    """
    用lst[start] 做基准值，在start到end这个范围进行分区
    """
    pivot = lst[start]

    while start < end :
        while start < end and lst[end] >= pivot:
            end -= 1
        lst[start] = lst[end]

        while start < end and lst[start] <= pivot:
            start += 1
        lst[end] = lst[start]

    lst[start] = pivot
    return start
```

partition函数返回基准值最后的索引，知道这个索引，才能将之前的待排序数组分为两部分，下面是递归部分的实现

```python
def my_quick_sort(lst,start,end):
    if start>= end:
        return

    index = partition(lst,start,end)
    my_quick_sort(lst,start,index-1)
    my_quick_sort(lst,index+1,end)
```

虽然这两段代码里的逻辑，你还有些不清楚，但整个的分析过程应该说是比较清晰的，先实现分区，然后实现递归，在编写算法时，很忌讳大包大揽的考虑问题，不分层次，不分先后，不分轻重。

分区虽然没有让整个数组变得有序，但是让基准值找到了自己应该在的位置，对左右两侧重复分区动作，每一次分区动作都至少让一个元素找到自己应该在的位置。

验证代码

```python
if __name__ == '__main__':
    lst = [4,3,2,4,1,5,7,2]
    my_quick_sort(lst,0,len(lst)-1)
    print lst
```
