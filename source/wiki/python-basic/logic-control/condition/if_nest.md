---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.4 if嵌套
order: 3
---

# if嵌套

## 1. if嵌套

最简单的if条件语句结构如下

```text
if condition:
    block
```

当表达式condition成立时，程序进入到if 下面的代码块block执行，在block中，还可以写if条件语句么?答案是肯定的，这样就形成了一个if嵌套，在某个条件下，继续针对不同的条件进行控制，嵌套的层次不受限制，只要你自己能理清思路就行。

使用input函数接收用户的输入，用户输入一个正整数num，对num进行判断，如果num小于50 ，那么则继续判断它能否被3整除，如果能，输出“可以被3整除”，否则输出“不能被3整除”， 如果numd大于等于50，则继续判断能否被5整除，如果能，输出“可以被5整除”，否则输出“不能被5整除”

直接给出示例代码

```python
num = int(input("输入一个数字："))
if num < 50:
    if num % 3 == 0:
        print('可以被3整除')
    else:
        print('不能被3整除')
else:
    if num % 5 == 0:
        print("可以被5整除")
    else:
        print("不能被5整除")
```

![程序流程图](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822172907331.jpeg)

## 2. 随堂练习

### 2.1 题目要求

使用input函数接收用户的输入，如果输入的数据不可以转换成int类型数据，则输出"无法使用int函数转换"，如果可以，则将用户的输入转成int类型数据并继续判断。

如果输入数据是奇数，则将其乘以2并输出，如果是偶数，则判断是否能被4整除，如果可以则输出被4整除后的值，若不能被4整数，则判断是否大于20，如果大于20则输出与20的差值，如果小于等于20，则直接输出该值

### 2.2 思路分析

使用字符串的isdigit()方法可以判断一个字符串能否使用int函数转换为int类型数据

### 2.3 示例代码

```python
value = input("请输入一个整数:")
if not value.isdigit():
    print('无法使用int函数转换')
else:
    i_value = int(value)
    if i_value % 2 == 1:
        print(i_value*2)
    elif i_value % 4 == 0:
        print(i_value / 4)
    elif i_value > 20:
        print(i_value - 20)
    else:
        print(i_value)
```

### 2.4 程序流程图

![程序流程图](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822172907766.jpeg)
