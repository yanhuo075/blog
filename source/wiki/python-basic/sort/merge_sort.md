---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.7 归并排序
order: 6
---

# 归并排序

## 合并两个有序集合

有两个有序的序列，分别为 [1,4,7] ,[2,3,5],现在请考虑将这两个序列合并成一个有序的序列。

其实方法真的非常简单

1. 首先创建一个新的序列，分别从两个序列中取出第一个数，1和2，1比2小，把1放到新的序列中
2. 第一个序列中的1已经放到新序列中，那么拿出4来进行比较，2比4小，把2放到新的序列中
3. 第二个序列中的2已经放到新序列中，那么拿出3来进行比较，3比4小，把3放到新的序列中
4. 第二个序列中的3已经放到新序列中，那么拿出5来进行比较，4比5小，把4放到新的序列中
5. 第一个序列中的4已经放到新序列中，那么拿出7来进行比较，5比7小，把5放到新的序列中
6. 最后把7放入到新的序列中

合并的方法就是分别从两个序列中拿出一个数来进行比较，小的那一个放到新序列中，然后，从这个小的数所属的序列中拿出一个数来继续比较

示例代码

```python
def merge_lst(left_lst,right_lst):
    left_index, right_index = 0, 0
    res_lst = []
    while left_index < len(left_lst) and right_index < len(right_lst):
        # 小的放入到res_lst中
        if left_lst[left_index] < right_lst[right_index]:
            res_lst.append(left_lst[left_index])
            left_index += 1
        else:
            res_lst.append(right_lst[right_index])
            right_index += 1

    # 循环结束时,必然有一个序列已经都放到新序列里,另一个却没有
    if left_index == len(left_lst):
        res_lst.extend(right_lst[right_index:])
    else:
        res_lst.extend(left_lst[left_index:])

    return res_lst
```

## 归并排序

归并排序，利用了合并有序序列的思想，把一个序列分成A,B两个序列，如果这两个序列是有序的，那么直接合并他们不就可以了么，但是A，B两个序列未必是有序的，没关系，就拿A序列来说，我把A序列再分一次，分成A1,A2，如果A1，A2有序我直接对他们进行合并，A不就变得有序了么，但是A1，A2未必有序啊，没关系，我继续分，直到分出来的序列里只有一个元素的时候，一个元素，就是一个有序的序列啊，这个时候不就可以合并了

这样一层一层的分组，分到最后，一个组里只有一个元素，终于符合合并的条件了，再一层一层的向上合并

完成示例代码：

```python
def merge_lst(left_lst,right_lst):
    left_index, right_index = 0, 0
    res_lst = []
    while left_index < len(left_lst) and right_index < len(right_lst):
        # 小的放入到res_lst中
        if left_lst[left_index] < right_lst[right_index]:
            res_lst.append(left_lst[left_index])
            left_index += 1
        else:
            res_lst.append(right_lst[right_index])
            right_index += 1

    # 循环结束时,必然有一个序列已经都放到新序列里,另一个却没有
    if left_index == len(left_lst):
        res_lst.extend(right_lst[right_index:])
    else:
        res_lst.extend(left_lst[left_index:])

    return res_lst


def merge_sort(lst):
    if len(lst) <= 1:
        return lst

    middle = len(lst)//2
    left_lst = merge_sort(lst[:middle])
    right_lst = merge_sort(lst[middle:])
    return merge_lst(left_lst, right_lst)


if __name__ == '__main__':
    lst = [19,4,2,8,3,167,174,34]
    print merge_sort(lst)
```
