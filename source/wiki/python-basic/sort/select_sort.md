---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 13.4 选择排序
order: 3
---

# 选择排序

假设有一个序列，a[0],a[1],a[2]...a[n]现在，对它进行排序。我们先从0这个位置到n这个位置找出最小值，然后将这个最小值与a[0]交换，然后呢，a[1]到a[n]就是我们接下来要排序的序列

我们可以从1这个位置到n这个位置找出最小值，然后将这个最小值与a[1]交换，之后，a[2]到a[n]就是我们接下来要排序的序列

每一次，我们都从序列中找出一个最小值，然后把它与序列的第一个元素交换位置，这样下去，待排序的元素就会越来越少，直到最后一个

## 示例代码

```python
def select_sort(lst):
    for i in range(len(lst)):
        min = i
        for j in range(min,len(lst)):
            # 寻找min 到len(lst)-1 这个范围内的最小值
            if lst[min] > lst[j]:
                min = j
        lst[i], lst[min] = lst[min], lst[i]

lst = [2,6,1,8,2,4,9]
select_sort(lst)
print lst
```
