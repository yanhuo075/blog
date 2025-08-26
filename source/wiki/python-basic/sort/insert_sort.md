---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.3 插入排序
order: 2
---

# 插入排序

插入排序就是每一步都将一个待排数据按其大小插入到已经排序的数据中的适当位置，直到全部插入完毕。

咱们举一个例子，你就能明白插入排序的精髓所在

```python
lst = [1, 2, 6, 7, 5]
```

lst是一个待排序的列表，你仔细观察不难发现，这个列表里的前4个数据已经是有序的了，现在，只需要把最后一个元素5插入到一个合适的位置就可以了。

从7开始向左遍历，比5大的数向右移动，当遇到一个小于等于5的数就停下来，这个位置就是5应该在的位置。当7向右移动时，占据了5的位置，因此，程序里需要一个变量把5保存下来，还需要一个变量把向左遍历时的索引记录下来，最后这个索引就是5应该在的位置。

```python
def insert(lst, index):
    """
    列表lst从索引0到索引index-1 都是有序的
    函数将索引index位置上的元素插入到前面的一个合适的位置
    :param lst:
    :param index:
    :return:
    """
    if lst[index-1] < lst[index]:
        return

    tmp = lst[index]
    tmp_index = index
    while tmp_index > 0 and lst[tmp_index-1] > tmp:
        lst[tmp_index] = lst[tmp_index-1]
        tmp_index -= 1
    lst[tmp_index] = tmp


if __name__ == '__main__':
    lst = [1, 2, 6, 7, 5]
    insert(lst, 4)
    print(lst)
```

函数默认列表lst从0到index-1都是有序的，现在只需要将索引index位置上的数据插入到合适的位置就可以了。

你可能已经产生了疑惑，咱们是要排序啊，你这函数默认前面index-1个数据都是有序的，这不合理啊，前面index-1个数据如果是无序的，你怎么办呢？

看下面的代码，你就全明白了

```python
def insert(lst, index):
    """
    列表lst从索引0到索引index-1 都是有序的
    函数将索引index位置上的元素插入到前面的一个合适的位置
    :param lst:
    :param index:
    :return:
    """
    if lst[index-1] < lst[index]:
        return

    tmp = lst[index]
    tmp_index = index
    while tmp_index > 0 and lst[tmp_index-1] > tmp:
        lst[tmp_index] = lst[tmp_index-1]
        tmp_index -= 1
    lst[tmp_index] = tmp


def insert_sort(lst):
    for i in range(1, len(lst)):
        insert(lst, i)


if __name__ == '__main__':
    lst = [1, 6, 2, 7, 5]
    insert_sort(lst)
    print(lst)
```

第1个元素单独看做一个数列，它本身就是有序的，那么只需要执行insert(lst, 1)，就可以保证前两个数据变成有序的，然后执行insert(lst, 2)，此时，从索引0到索引1是有需的，只需要将索引为2的数据插入到合适的位置就可以了。
