---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.5 zip
order: 4
---

# zip

## 1. zip函数应用示例

zip函数可以将多个可迭代对象作为参数，将对象中对应的元素打包成元组，如果可迭代对象的长度不同，则根据长度最短的那个进行打包，比如下面两个列表

```python
lst1 = [1, 2, 3, 4, 5]
lst2 = ['一', '二', '三', '四', '五']
```

要求你根据这两个列表构建一个字典

```text
{1: '一', 2: '二', 3: '三', 4: '四', 5: '五'}
```

如果使用普通的方法，则需要对两个列表进行遍历，示例代码如下

```python
lst1 = [1, 2, 3, 4, 5, 6]
lst2 = ['一', '二', '三', '四', '五']

info = {}
for index, item1 in enumerate(lst1):
    if index < len(lst2):
        info[item1] = lst2[index]

print(info)
```

这种写法要考虑lst1的长度大于lst2长度的情况，代码看起来也算简洁，但使用zip函数可以写出更简洁的代码

```python
lst1 = [1, 2, 3, 4, 5, 6]
lst2 = ['一', '二', '三', '四', '五']

info = {}
for item1, item2 in zip(lst1, lst2):
    info[item1] = item2

print(info)
```

想要更加简洁，可以使用字典推导式

```python
lst1 = [1, 2, 3, 4, 5, 6]
lst2 = ['一', '二', '三', '四', '五']

info = {item1: item2 for item1, item2 in zip(lst1, lst2)}
print(info)
```

## 2. 自己实现zip函数

如果只是单纯的讲解zip函数的用法，也就不会放在进阶教程里了，既然已经了解了zip函数的功能，那么我们来尝试一下自己实现一个类似功能的函数。

函数接收多个可迭代对象，说明函数定义时使用了可变长参数，在python3中，zip函数不再返回列表，而是一个可迭代对象，我们自己来实现一个可迭代对象，可以使用yield关键字。下面的my_zip是我自己实现的类似zip功能的函数

```python
lst1 = [1, 2, 3, 4, 5, 6]
lst2 = ['一', '二', '三', '四', '五']

def my_zip(*args):
    min_len = min(len(item) for item in args)       # 获得可迭代对象长度最小值
    index = 0
    while index < min_len:
        tmp_lst = [item[index] for item in args]    # 从每个可迭代对象里取出一个值
        index += 1
        yield tuple(tmp_lst)        # 返回元组

for item1, item2 in my_zip(lst1, lst2):
    print(item1, item2)
```
