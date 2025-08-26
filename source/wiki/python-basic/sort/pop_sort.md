---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.2 冒泡排序
order: 1
---

# 冒泡排序

冒泡排序的核心思想是相邻的两个数据进行比较，假设数列A有n个数据，先比较第1个和第2个数据，如果A1 > A2,则交换他们的位置，确保较大的那个数在右侧。

接下来比较A2和A3，采用相同的规则，较大的数向右移动，最后会比较An-1 和An的大小，如果An-1 > An，那么交换他们的位置，这时，An是数列中的最大值。

你肯定已经发现，经过这一轮比较后，数列仍然是无序的，但是没有关系，我们已经找到了最大值An，而且它在队列的末尾。

接下来要做的事情，就是简单的重复之前的过程，整个数列，先暂时把An排除在外，这n-1个无序的数，仍然可以采用之前的方法，找出n-1个数当中的最大值，这样An-1就是第2大的数，继续对n-2个数做相同的事情

为了让你更容易理解冒泡排序，我们先实现一个简单的函数

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


if __name__ == '__main__':
    lst = [7, 1, 4, 2, 3, 6]
    move_max(lst, len(lst)-1)
    print(lst)
```

这个函数只完成一个简单的功能，它将列表从索引0到max_index之间的最大值移动max_index索引上，这正是冒泡排序的核心思想。

当我们完成这一步，剩下的事情，就是不断的重复这个过程。

```python
def pop_sort(lst):
    for i in range(len(lst)-1, 0, -1):
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
    lst = [7, 1, 4, 2, 3, 6]
    pop_sort(lst)
    print(lst)
```
