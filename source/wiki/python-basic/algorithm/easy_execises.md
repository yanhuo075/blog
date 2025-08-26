---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 14.2  简单练习题
order: 1
---

# 简单练习题

# python练习题---求三位数组合

```python
lst = [3, 6, 2, 7]
```

这四个数字能组成多少个互不相同且无重复数字的三位数？比如362算一个，326算一个，请逐个输出他们

## 思路分析

从4个数字里挑出来，组成一个3位数，就算法而言，最方便的做法是用一个3层嵌套循环，分别从lst取数，取出来的数值组成一个3位数，题目要求无重复数字，这就要求，取出来的3个数字互不相等

## 示例代码

```python
lst = [3, 6, 2, 7]

for a in lst:
    for b in lst:
        for c in lst:
            if a != b and b != c and a != c:
                print(a*100 + b*10 + c)
```

如何判断3个数值互不相等，还有一个更简单的办法

```python
if a not in (b, c) and b != c:
```



# 生成矩阵

## 题目要求

已知两个列表

```text
lst_1 = [1, 2, 3, 4]
lst_2 = ['a', 'b', 'c', 'd']
```

请写算法，将两个列表交叉相乘，生成如下的矩阵

```text
[['1a', '2a', '3a', '4a'],
 ['1b', '2b', '3b', '4b'],
 ['1c', '2c', '3c', '4c'],
 ['1d', '2d', '3d', '4d']]
```

## 思路分析

观察生成的矩阵，可以得出这样的结论，lst_1的长度决定了矩阵有多少列，lst_2的长度决定了生成的矩阵有多少行。

既然是交叉相乘，那么可以写两个for循环，嵌套遍历这两个列表，对lst_2的遍历放在外层，对lst_1的遍历放在内层。

## 示例代码

```python
import pprint


lst_1 = [1, 2, 3, 4]
lst_2 = ['a', 'b', 'c', 'd']

lst = []
for item2 in lst_2:
    tmp = []
    for item1 in lst_1:
        tmp.append(str(item1) + item2)
    lst.append(tmp)

pprint.pprint(lst)
```



# python练习题-列表偏移

## 题目要求

lst = [1,2,3,4,5]，列表向右偏移两位后，变成lst = [5,4,1,2,3]

## 思路分析

先说简易思路，以向右偏移2位做例子，直接对列表进行切片操作，然后再讲切片后的列表连接在一起

```python
lst = [1,2,3,4,5]

lst = lst[len(lst)-2:] + lst[:len(lst)-2]
print(lst)
```

如果是初学者这样做，是完全可以接受的，但如果想更上一层楼，则需要更加合理的解决方法

首先，偏移的位数可能会超过列表的长度，比如向右偏移47位，此时，就要考虑用47%5 = 2，向右偏移47位等价于向右偏移2位

直接对列表进行切片操作，终究还是用了列表的原生功能，既然是练习题，且偏向于算法层面，那么我们应该自己来实现偏移，不生成新的列表，只在原列表上动手脚

不论偏移多少位，在实现时，每一次，我们只偏移1位，先实现偏移1位的情况。

偏移1位，只需要将列表最后一位记录下来，然后其他所有位置上的数值向右移动一位，最后将之前记录下来的列表最后一位放置在列表的开头即可。

实现了偏移1位，剩下的仅仅是写一个for循环，就可以偏移指定的位数

## 示例代码

```python
lst = [1,2,3,4,5]

count = int(input("请输入偏移位数"))
count = count % len(lst)

for i in range(count):
    tmp = lst[-1]
    for j in range(len(lst)-1, 0, -1):
        lst[j] = lst[j-1]

    lst[0] = tmp

print(lst)
```



# python练习题-随机数生成与遍历

## 题目要求

生成有100个随机正整数的列表，随机数小于1000，遍历列表，删除其中的素数，然后输出最大值，将剩余数值降序输出

## 思路分析

随机数的生成需要使用random模块

```python
lst = [random.randint(2, 999) for i in range(100)]
```

素数，是大于1的数，除了1和自身，没有其他的因数，大白话就是除了1和自身，除以其他的数都得不到整数结果

假设一个数值是M，那么从2开始到M//2 ，用M逐个对这些数取模，如果得到0，就说明可以整除，M就不是素数

为啥只遍历到M//2 呢，因为大于M//2的数，除了M意外，其他的肯定不能被整除，简单的数学知识，不必再解释

```python
def is_prime(number):
    for i in range(2, number/2):
        if number % 2 == 0:
            return False

    return True
```

想要删除素数，千万不能这么写

```python
for number in lst:
    if is_prime(number):
        lst.remove(number)
```

在使用for循环对列表进行遍历时，本质上是在根据数据的索引获取数据，如果使用remove方法删除的某个数据，那么被删除的数据后面的数据的索引都发生了改变，会影响for循环，导致紧挨着你删除的数据的下一个数据无法遍历到，下面的这个例子可以很清楚的观察到这个情况

```python
lst = [2, 4, 6, 8, 10, 12]
for number in lst:
    print(number)
    if number % 2 == 0:
        lst.remove(number)

print(lst)
```

不能用remove,咋办呢，可以使用列表推导式，也可以使用一个新的列表来存储非素数

```python
lst = [item for item in lst if not is_prime(item)]
```

或者

```python
new_lst = []
for item in lst:
    if not is_prime(item):
        new_lst.append(item)
```

最后找出列表最大值，可以使用max内置函数，也可以对列表进行排序，从大到小

## 完整示例代码

```python
import random

# 生成100个随机正整数
lst = [random.randint(1, 1000) for i in range(100)]


def is_prime(number):
    """
    判断一个正整数是否为素数
    :param number:
    :return:
    """
    for i in range(2, number//2):
        if number % i == 0:
            return False

    return True

# 过滤素数
lst = [item for item in lst if not is_prime(item)]

# 从大到小排序
lst.sort(reverse=True)
print(lst[0])  # 最大值

# 倒序输出
for item in lst:
    print(item)
```



# python练习题-寻找列表里第二大的数

## 题目要求

实现算法，查找列表里第二大的数

```python
lst = [3, 6, 7, 9, 2]
```

## 思路分析

最直接的方法是使用列表的sort方法，从小到大排序，lst.sort()，lst[-2] 就是列表里第2大的数。

能想到这个处理方法，对初学者来说，很好了，如果要求算法复杂度为O(n) 呢？ 显然，排序算法都不符合要求。

找列表里最大值，只需要遍历一遍列表就可以实现，算法复杂度是O(n)， 能否以此为基础，顺带手的找出列表里第2大的数呢？

设置两个变量，max_value,sec_max_value，分别表示列表里的最大值和第2大的值，遍历列表，分别于他们比较并进行替换，

## 示例代码

```python
lst = [3, 6, 7, 9, 2]

max_value = lst[0]
sec_max_value = lst[1]

# 先假设列表里前两个数据是最大的和第二大的
if sec_max_value > max_value:
    max_value, sec_max_value = sec_max_value, max_value

# 遍历列表
for item in lst:
    # 小于等于第2大的数
    if item <= sec_max_value:
        continue

    # 大于第2大的数，小于等于最大的数
    if sec_max_value < item <= max_value:
        sec_max_value = item

    # 大于最大的数
    if item > max_value:
        sec_max_value = max_value
        max_value = item

print(max_value, sec_max_value)
```



# python练习题-startswith

## 题目要求

实现函数is_startswith，如果字符串source是以substr开头的，则函数返回True,反之返回False

```python
def is_startswith(source, substr):
    """
    判断字符串source是否以substr开头
    :param source:
    :param substr:
    :return:
    """
    pass
```

## 思路分析

函数首先要判断传入的参数是否合法，这里默认传入的都是字符串，那么我们要需要判断字符串是否有空串的情况

如果substr的长度大于source的长度，直接返回False

从索引0开始，遍历substr,从source上获得相同索引的字符，两者进行比较，只要有一个字符不相同，则可以立即返回False

## 示例代码

```python
def is_startswith(source, substr):
    """
    判断字符串source是否以substr开头
    :param source:
    :param substr:
    :return:
    """
    if not source or not substr:
        return False

    if len(substr) > len(source):
        return False

    for index, item in enumerate(substr):
        if item != source[index]:
            break
    else:
        return True  # 如果for循环不是因为break结束的，就会进入到else语句块

    return False


if __name__ == '__main__':
    print(is_startswith("python", 'py'))
```



# python练习题-统计字符数量

## 题目要求

使用input函数接收用户的输入，统计出其中英文字母、空格、数字和其它字符的个数，例如用户输入字符串"sdfijer384323js",程序最终输出信息

```python
{'s': 2, 'd': 1, 'f': 1, 'i': 1, 'j': 2, 'e': 1, 'r': 1, '3': 3, '8': 1, '4': 1, '2': 1}
```

## 思路分析

input函数接收用户的输入，得到的是字符串，遍历字符串，统计字符串中每一个字符出现的次数，这种信息自然是用字典来存储，其他容器类型数据列表，元组，集合，都不适合存储这种key-value形式的数据

## 示例代码

```python
string = input("请输入一个字符串:")
info = {}
for item in string:
    info.setdefault(item, 0)
    info[item] += 1

print(info)
```



# 忽略大小比较字符串是否相等

## 题目要求

比较两个字符串，忽略大小写，比如字符串"abc"和字符串"ABC",在忽略大小写的情况下是相等的，实现函数

```python
def is_same_ignore_case(str1, str2):
    """
    忽略大小写比较两个字符串是否相等
    :param str1:
    :param str2:
    :return:
    """
    pass
```

要求，不能使用字符串的lower方法和upper方法

## 思路分析

题目要求不能使用字符串的lower方法和upper方法，之所以这样要求，是希望你能从更基础的编程做起，培养更深入的思考。

要解这个练习题，你需要对ASCII码表有一定的了解，在ASCII码表中，大小写字母的十进制编码相差32。

通过for循环，遍历str1,同时获取与之相对应索引上的字母，因为要忽略大小写，因此，将他们都转换成ASCII码表里的十进制数值，并且统一转成小写字母的十进制数值。

在进行正式比较前，严谨的判断逻辑应该是先判断传入的参数str1和str2是否有效，然后判断这两个字符串的长度是否相等，这些都是从程序的健壮性和效率上考虑的。

enumerate函数是一个非常方便且有用的函数，在for循环中，既能获得遍历的元素，又能获得该元素的索引。

## 示例代码

```python
def get_ord_value(string):
    value = ord(string)
    if 65 <= value <= 90:
        value += 32

    return value


def is_same_ignore_case(str1, str2):
    """
    忽略大小写比较两个字符串是否相等
    :param str1:
    :param str2:
    :return:
    """
    if not isinstance(str1, str) or not isinstance(str2, str):
        return False

    if len(str1) != len(str2):
        return False

    # 统一转成小写
    for index, item in enumerate(str1):
        value1 = get_ord_value(item)
        value2 = get_ord_value(str2[index])

        if value1 != value2:
            return False

    return True


if __name__ == '__main__':
    print(is_same_ignore_case('ABC', 'abc'))
    print(is_same_ignore_case('ABC', 'abd'))
```



# 计算三角形的周长和面积

## 题目要求

写一段程序，让用户输入三角形的三条边长，如果三条边长不能构成三角形，则提示用户重新输入

如果可以构成三角形，则计算周长和面积

## 思路分析

对于用户的输入，首先要约定格式，这里简单的约定为每个边长之间用空格间隔

在获得用户的输入以后，要对输入进行检查，有两点需要检查

（1） 检查是不是输入了三条边的边长，输入2个或者4个都是错误的

（2） 检查输入的内容是不是数值型，如果输入的是字母，那根本驴唇不对马嘴

以上两点是编程时要考虑的，经过上面的分析你应该有所体会，编程，并不是你掌握一门语言然后用它在计算机上做各种操作

编程，是对问题的思考，我这里约定让用户一次性输入三条边的边长，中间用空格隔开，你也可以让用户输入三次，也可以让用户输入一次但用别的字符做间隔，这些都是没有定论的，完全取决于你的思考

对于输入内容检查，你可能会以为python会自己完成，但其实不会，input获得的就是字符串，你必须理解什么是字符串，必须清楚的知道input的作用，这些都是最最基础的内容，如果你不掌握这些，那你就无法思考

根本不存在一种方法或者操作可以满足你的要求，可以解决实际的问题，编程就是分析问题然后你自己解决问题的过程

```python
def get_edge(line):
    """
    根据用户的输入获得三条边
    用户输入三条边的长度,用空格隔开 类似于  3 4 5
    :param line:
    :return:
    """
    edge_lst = line.split(" ")
    # 如果输入条数不是3
    if len(edge_lst) != 3:
        return False, (0, 0, 0)
    try:
        # raw_input 获得用户的输入,得到的是字符串,这里把字符串转成float数值
        edge_lst = [float(item) for item in edge_lst]
    except:
        return False, (0, 0, 0)

    return True, (edge_lst[0], edge_lst[1], edge_lst[2])


def is_able_triangle(a, b, c):
    """
    判断能否构成三角形
    :param a:
    :param b:
    :param c:
    :return:
    """
    return (a + b > c) and (a + c > b) and (c + b > a)


def triangle_func():
    while True:
        line = input('输入三角形的三个边长,用空格隔开,退出请输入q:')
        if line == 'q':
            break
        input_correct, edges = get_edge(line)
        if not input_correct:
            print('输入错误')
            continue

        if not is_able_triangle(edges[0], edges[1], edges[2]):
            print('不能构成三角形')
            continue

        perimeter = sum(edges)
        half_perimeter = perimeter/2
        area = (half_perimeter*(half_perimeter-edges[0])*(half_perimeter-edges[1])*(half_perimeter-edges[2])) ** 0.5

        print('周长: {perimeter}  面积:{area}'.format(perimeter=perimeter, area=area))

if __name__ == '__main__':
    triangle_func()
```



# 打印杨辉三角

## 题目要求

给定一个正整数N,打印杨辉三角的前N行
杨辉三角形态如下

```text
          1
        1   1
      1   2   1
    1   3   3   1
  1   4   6   4   1
1   5   10  10  5   1
```

杨辉三角的每一行第一个和最后一个元素都是1
中间的元素,由上一行的两个元素相加得到,第N行的第index元素
是由第N-1 行的第index-1元素和第index相加得到的

## 思路分析

- 杨辉三角的每一行的第一个元素和最后也元素都是1
- 中间的元素可以由上一行的元素两两相加得到

第N行元素，可以由第N-1 行变化而来，这正是递归算法的精髓，把新的问题转化成老的问题，而老的问题转化成更老的问题，最终必然有一个老问题给出答案，然后整个逻辑链条上的问题都有了答案

## 代码示例

```python
def print_yanghui(n):
    """
    递归算法打印杨辉三角
    :param n:
    :return:
    """
    if n == 1:
        print([1])
        return [1]
    elif n == 2:
        print_yanghui(1)
        print([1, 1])
        return [1, 1]  # 这里如果不返回,第三行就无法生成
    else:
        lst = [1]  # 第一个元素
        pre_lst = print_yanghui(n - 1)  # 得到上一行的元素
        # 根据第n -1 行的元素,生成第n行的中间元素
        for i in range(len(pre_lst) -1):
            lst.append(pre_lst[i] + pre_lst[i+1])
        lst.append(1)  # 最后一个元素
        print(lst)
        return lst

if __name__ == '__main__':
    print_yanghui(6)
```
