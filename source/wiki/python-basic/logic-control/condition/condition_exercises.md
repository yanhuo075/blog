---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.9 条件控制练习题
order: 8
---

# 条件控制练习题

## 1.判断日期是否合法

### 1.1 题目要求

使用input函数接受用户输入，用户输入月份和日期，比如“3月15日”，请写程序判断这个日期是否合法，为了简化编程难度， 只考虑非闰年的情况，默认月份总是正确。以下为输入示例

示例1

```text
输入: 3月15日
输出: 合法
```

示例2

```text
输入: 5月32日
输出: 不合法
```

### 1.2 思路分析

1，3，5，7，8，10，12月有31天，非闰年2月份有28天，其他月份有30天。

输入的数据是一个字符串，需要把月份和日期提取出来，提取思路如下

1. 使用字符串replace方法将“日”替换成空字符串
2. 使用split方法，“月”作为分割符对字符串进行分割，得到列表
3. 将列表里的数据转换成int类型数据

### 1.3 示例代码

```python
date_time = input("请输入月份和日期：")
date_time = date_time.replace("日", '')
time_lst = date_time.split('月')
month, day = int(time_lst[0]), int(time_lst[1])

if month in (1, 3, 5, 7, 8, 10, 12):
    if 1 <= day <= 31:
        print("合法")
    else:
        print("不合法")
elif month == 2:
    if 1 <= day <= 28:
        print("合法")
    else:
        print("不合法")
else:
    if 1 <= day <= 30:
        print("合法")
    else:
        print("不合法")
```

### 1.4 程序流程图

![python判断日期是否合法](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822173231086.jpeg)

### 1.5 更简洁的写法

上面的代码虽然逻辑上没有问题，但过于臃肿，不符合python一贯简洁的风格，下面的代码相比较而言，代码量更好，更利于阅读，逻辑更加清晰，这是你需要努力的方向

```python
date_time = input("请输入月份和日期：")
date_time = date_time.replace("日", '')
time_lst = date_time.split('月')
month, day = int(time_lst[0]), int(time_lst[1])
min_day = 1
max_day = 30

if month in (1, 3, 5, 7, 8, 10, 12):
    max_day = 31
elif month == 2:
    max_day = 28

if min_day <= day <= max_day:
    print("合法")
else:
    print("不合法")
```

## 2. 判断闰年

### 2.1 题目要求

使用input函数接收用户的输入，用户输入一个年份，请写代码判断这一年是否为闰年，如果是闰年，则输出“xxx 是闰年”,反之输出“xxx 不是闰年”, 闰年的规则如下:

1. 不能被4整除的一定不是闰年
2. 如果年份能被4整除且不能被100整除，则是闰年
3. 如果年份能被4整除，同时又能被100整除，那么要判断它能否被400整除，如果可以则是闰年，否则不是

### 2.2 示例代码

```python
year = int(input("请输入一个年份："))

if (year % 4) == 0:
    if year % 100 == 0:
        if year % 400 == 0:
            print("{year} 是闰年".format(year=year))
        else:
            print("{year} 不是闰年".format(year=year))
    else:
        print("{year} 是闰年".format(year=year))
else:
    print("{year} 不是闰年".format(year=year))
```

### 2.3 程序流程图

![python判断闰年](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822173239526.jpeg)

## 3. 比较两个数大小

### 3.1 题目要求

使用input函数接收用户输入，用户会输入两个整数，中间用空格分割，程序输出两个数中最大的一个。

### 3.2 思路分析

input函数返回的数据是一个字符串，使用字符串split方法可以将字符串解析成一个列表，列表里有两个字符串，将他们转换成int类型数据，分别赋值给变量a,b，对这两个数值进行比较，输出最大的一个

### 3.3 示例代码

```python
value = input("请输入两个整数,中间用空格分开:")
lst = value.split()
a, b = int(lst[0]), int(lst[1])
if a >= b:
    print(a)
else:
    print(b)
```

### 3.4 程序流程图

![python比较两个数大小流程图](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822173232224.jpeg)

## 4. 比较三个数大小

### 4.1 题目要求

使用input函数接收用户输入，用户会输入三个整数，中间用空格分割，程序输出三个数中最大的一个。

### 4.2 思路分析

这个题目，我们借助程序流程图来梳理思路
![python比较3个数大小流程图](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822173230585.jpeg)

### 4.3 示例代码

```python
value = input("请输入两个整数,中间用空格分开:")
lst = value.split()
a, b, c = int(lst[0]), int(lst[1]), int(lst[2])

if a >= b:
    if a >= c:
        print(a)
    else:
        print(c)
else:
    if b >= c:
        print(b)
    else:
        print(c)
```

### 4.4 另一种算法

4.3 的代码，并不是一种好的算法，逻辑处理过于繁琐，作为训练使用还可以，若是用于实际工作，空拍就要被笑话了,下面提供一种更好的算法

```python
value = input("请输入两个整数,中间用空格分开:")
lst = value.split()
a, b, c = int(lst[0]), int(lst[1]), int(lst[2])

max_value = a
if b >= max_value:
    max_value = b

if c >= max_value:
    max_value = c

print(max_value)
```

还有更加简单的写法，在初学阶段，我们不在进行讨论。
