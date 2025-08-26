---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.2.6 循环控制练习题
order: 6
---

# 循环控制练习题

本文共3个python for循环, while循环练习题，分别是判断回文，整数翻转， 寻找缺失数，通过这3个练习题，可以强化提升你对for循环和while循环的理解和运用。

## 1. 判断回文

### 1.1 题目要求

使用input函数接收用户的输入，用户输入一个字符串，请判断这个字符串是不是回文，回文是一种对称的字符串，从左向右看和从右向左看是一样的，比如abcba

### 1.2 思路分析

解决问题的关键在于回文的对称性，假设字符串是string，那么先比较string[0] 和 string[-1]，如果他们相等，则继续比较string[1]和string[-2]，这就需要对字符串进行遍历，由于回文的对称性，只需要遍历到一半就可以了

### 1.3 示例代码

```python
value = input("请输入一个字符串:")
for i in range(len(value)//2):
    if value[i] != value[-1-i]:
        print('你输入的{value}不是回文'.format(value=value))
        break
else:
    print('你输入的{value}是回文'.format(value=value))
```

这段代码里用到了for else 语法，当for循环正常结束时，就会执行else子句，如果for循环是由于break语句导致结束的，那么就不会执行else子句。如果你不习惯这种偏高级点的语法，可以参考下面这段代码

```python
value = input("请输入一个字符串:")
is_plalindrome = True

for i in range(len(value)//2):
    if value[i] != value[-1-i]:
        is_plalindrome = False
        break

if is_plalindrome:
    print('你输入的{value}是回文'.format(value=value))
else:
    print('你输入的{value}不是回文'.format(value=value))
```

## 2. 整数翻转

### 2.1 题目要求

使用input函数接收用户的输入，用户输入一个整数，请使用while循环获得整数翻转后的结果，比如用户输入12345，程序最后输出54321，不能借助列表。

### 2.2 思路分析

如果借助列表来实现这个算法，简直不要太简单

```python
string = "12345"
lst = list(string)
lst = lst[::-1]

print(''.join(lst))
```

在不借助列表的情况下，处理整数的翻转，可以使用// 和 % 运算符， 整数除以10，整体向右偏移，整数对10取模，可以得到个位数。

### 2.3 示例代码

```python
value = input("请输入一个整数:")
value = int(value)

reverse_num = 0
while value > 0:
    last_num = value % 10
    value //= 10
    reverse_num = reverse_num*10 + last_num

print(reverse_num)
```

## 3. 寻找缺失数

### 3.1 题目要求

有一组数字，从0到10, 中减少了一个数，顺序也被打乱，放在一个列表里，请找出丢失的数字。

```python
lst = [4, 3, 1, 2, 0, 8, 5, 9, 6, 10]
```

### 3.2 解法1 利用递增序列求和

把7放回序列中，那么整个序列的和就是(n*(n+1))/2 ,n = 10, 结果为 55，现在序列里没有7 ，剩余的数加在一起是48, 55-48恰好就是缺失的数字7

示例代码：

```python
lst = [4, 3, 1, 2, 0, 8, 5, 9, 6, 10]
max_value = max(lst)
sum_value = max_value*(max_value+1)//2
miss_value = sum_value - sum(lst)
print(miss_value)
```

### 解法2 利用索引

示例中的序列有10个数字，算上丢失的有11个数字，创建一个长度为11的标识列表，元素都初始化为0，遍历序列，找到每个值在标识列表里的对应位置，修改元素值为1，哪个位置没有被修改为1，那个位置的索引就是丢失的值

```python
lst = [4, 3, 1, 2, 0, 8, 5, 9, 6, 10]
tag_lst = [0]*11

for item in lst:
    tag_lst[item] = 1

for index, item in enumerate(tag_lst):
    if item == 0:
        print(index)
```

### 解法3 利用亦或运算

0 与任意一个数做亦或操作还是自身，一个数与自己做亦或操作则等于0。先不考虑缺失了哪个数字，用一个变量res 存储 0^1^2...^n，然后再用res逐个与序列里的值进行亦或操作，这样，没有缺失的值会与自己进行亦或操作得到0，而缺失的那个数字则最终被留存下来

```python
lst = [4, 3, 1, 2, 0, 8, 5, 9, 6, 10]
res = 0
for i in range(1, len(lst)+1):
    res ^= i

for item in lst:
    res ^= item

print(res)
```
