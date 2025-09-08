---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.10 python函数练习题
order: 9
---

# python函数练习题

python函数练习题所练习的不仅仅是函数如何使用，还包括基础知识的运用，尽管所要实现的算法在不使用函数的情况下也都能实现，而且很多算法在前面都已经学习实践过。但对初学者而言，十分有必要通过编写函数来理解掌握函数的定义，调用，传参，返回值等概念，进而提高自己的逻辑思考能力。

## 1. 求列表最大值

### 1.1 题目要求

```python
def get_max(lst):
    """
    返回列表lst的最大值
    :param lst:
    :return:
    """
    pass


lst = [4, 2, 1, 6, 7, 9]
max_value = get_max(lst)
print(max_value)
```

实现函数get_max，函数最终返回列表lst的最大值,不要使用max函数

### 1.2 示例代码

```python
def get_max(lst):
    """
    返回列表lst的最大值
    :param lst:
    :return:
    """
    if not lst:
        return None
    max_value = lst[0]
    for item in lst:
        if item > max_value:
            max_value = item

    return max_value


lst = [4, 2, 1, 6, 7, 9]
max_value = get_max(lst)
print(max_value)
```

## 2. 求学生最高分数科目和分数

### 2.1 题目要求

```python
def get_max_socre(score_dic):
    """
    返回学生考试成绩的最高分的科目和分数
    :param score_dic:
    :return:
    """
    pass

dic = {
    '语文': 90,
    '数学': 97,
    '英语': 98
}

course, score = get_max_socre(dic)
print(course, score)
```

实现函数get_max_socre， 执行程序，最终输出： 英语 98

### 2.2 示例代码

```python
def get_max_socre(score_dic):
    """
    返回学生考试成绩的最高分的科目和分数
    :param score_dic:
    :return:
    """
    max_score = 0
    max_score_course = ''
    for course, score in score_dic.items():
        if score > max_score:
            max_score = score
            max_score_course = course

    return max_score_course, max_score

dic = {
    '语文': 90,
    '数学': 97,
    '英语': 98
}

course, score = get_max_socre(dic)
print(course, score)
```

## 3. 判断回文

### 3.1 题目要求

```python
def is_palindrome(string):
    """
    判断字符串string是回文
    :param string:
    :return:
    """
    pass


print(is_palindrome('abcddcba'))
print(is_palindrome('pythonohtyp'))
print(is_palindrome('bookkob'))
```

实现函数is_palindrome，判断参数string是否为回文，所谓回文，就是左右对称的字符串

### 3.2 示例代码

```python
def is_palindrome(string):
    """
    判断字符串string是回文
    :param string:
    :return:
    """
    for index in range(len(string)//2):
        if string[index] != string[len(string)-index-1]:
            return False

    return True


print(is_palindrome('abcddcba'))
print(is_palindrome('pythonohtyp'))
print(is_palindrome('bookkob'))
```

## 4. 打印九九乘法表

### 4.1 题目要求

在屏幕上输出九九乘法表

```text
1*1 = 1
1*2 = 2  2*2 = 4
1*3 = 3  2*3 = 6  3*3 = 9
1*4 = 4  2*4 = 8  3*4 = 12  4*4 = 16
1*5 = 5  2*5 = 10  3*5 = 15  4*5 = 20  5*5 = 25
1*6 = 6  2*6 = 12  3*6 = 18  4*6 = 24  5*6 = 30  6*6 = 36
1*7 = 7  2*7 = 14  3*7 = 21  4*7 = 28  5*7 = 35  6*7 = 42  7*7 = 49
1*8 = 8  2*8 = 16  3*8 = 24  4*8 = 32  5*8 = 40  6*8 = 48  7*8 = 56  8*8 = 64
1*9 = 9  2*9 = 18  3*9 = 27  4*9 = 36  5*9 = 45  6*9 = 54  7*9 = 63  8*9 = 72  9*9 = 81
```

### 4.2 思路分析

题目乍看起来挺难的，但其实非常简单，前提是你知道如何分析。

九九乘法表一共有9行，我们先考虑如何输出一行，一行解决了，使用一个for循环，就可以把所有行都输出了，定义函数print_line(line_number)，该函数输出第line_number行。

当line_number等于9的时候，研究一下如何输出第9行。第9行有9个式子，从1到9分别乘以9，看见没，这不就是一个for循环么，使用一个for循环，让变量i从1变化到9，每次都与9相乘，并将结果组装成一个i*9 = xx 的式子，把所有的式子连接在一起，就是第9行的内容

解决了指定行的输出后，只需要一个简单的for循环，从1到9，分别去调用这个函数不就将整个乘法表打印出来了么？

### 4.3 示例代码

```python
def print_line(line_number):
    lst = []
    for i in range(1, line_number+1):
        part = "{index}*{number} = {res}".format(index=i, number=line_number, res=i*line_number)
        lst.append(part)

    print("  ".join(lst))


def print_table():
    for i in range(1, 10):
        print_line(i)


print_table()
```

## 5. 矩阵对角线元素和

### 5.1 题目要求

求一个3*3矩阵中对角线上元素之和
先模拟一个3*3矩阵

3 5 6

4 7 8

2 4 9

### 5.2 思路分析

在C语言里，这种数据要用二维数组来存储，在python里，没有二维数组这个专业用语，概念上，你可以理解为嵌套列表，其定义如下

```text
lst = [
    [3,5,6],
    [4,7,8],
    [2,4,9]
]
```

lst中有3个元素，均是列表，lst[0]是一个列表，该列表里有3个元素。

题目要求计算对角线元素和，也就是lst[0][0] + lst[1][1] + lst[2][2],写一个for循环，用range(3)产生一个从0到2的序列，即可实现这三个元素的相加

### 5.3 实例代码

```python
lst = [
    [3,5,6],
    [4,7,8],
    [2,4,9]
]
sum = 0
for i in range(3):
    sum += lst[i][i]

print sum
```

## 6. 水仙花数

输出所有的水仙花数，所谓水仙花数是指一个三位数，各个位上的数的立方相加在一起等于这个三位数，比如153，1的3次方 + 5的三次方 + 3的三次方 等于153

### 6.1 思路分析

水仙花数是一个3位数，数值范围是从100到999之间，写一个for循环，从100到999进行遍历，逐个判断该数是否为水仙花数即可。

对于一个三位数，获得这个数的每一位也并不难，以153为例

```python
a = 153
while a > 0:
    print(a % 10)
    a //= 10
```

对数字的操作，永远离不开取模运算和整除运算

### 6.3 示例代码

```python
for i in range(100, 1000):
    res = 0
    value = i
    while value > 0:
        res += (value%10)**3
        value //= 10

    if res == i:
        print(res)
```

## 7. 完全平方数

完全平方数，就是可以表示为某个整数的平方的数，例如9，是3的平方，16是4的平方，9和16都是完全平方数，请打印10000以内的完全平方数

### 7.1 思路分析

两个思路，一个思路是从1到10000进行遍历，对每一个数值进行判断，判断其是否为某个整数的平方。

第二个思路，从1到10000进行遍历，计算每一个数值的平方，如果这个平方小于10000，那么这个数值的平方就是完全平方数。

显然，第二个方法更容易一些，毕竟开根号这种事情，不能保证开出来的一定是整数。

### 7.2 示例代码

```python
i = 1
value = 1
while value < 10000:
    print(i, value)
    i += 1
    value = i**2
```

## 8. 翻转列表

### 8.1 题目要求

```text
lst = [1,2,3,4,5]
翻转后
lst = [5,4,3,2,1]
```

### 8.2 思路分析

如果是用列表自带的功能，翻转列表是非常容易的事情

```text
lst = lst[::-1]
```

现在，需要你自己来实现翻转过程，思路很简单，从头开始遍历列表，但只遍历到一半，左右两边对位交换数据即可

### 8.3 示例代码

```python
lst = [1, 2, 3, 4, 5, 6, 7, 8, 9]


length = len(lst)

for i in range(length//2):
    tmp = lst[i]
    lst[i] = lst[length - 1 - i]
    lst[length - 1 - i] = tmp

print(lst)
```
