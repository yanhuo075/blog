---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 7.2 文件读写
order: 1
---

# python文件读写

python读取文件和写文件都要使用内置的open函数，通过传入访问模式来决定读写的方法，与其他语言一样，读写操作的文件对象主要分为文本和二进制两大类。

## 1. 读取文件

### 1.1 打开文件

读取文件时，首先要打开文件，获得文件句柄。

```python
f = open('学生信息.txt', 'r')
```

使用open函数，可以打开一个文件，返回文件句柄，open函数的第一个参数是文件地址，第二个参数是打开模式，打开模式详细介绍如下

| 访问模式 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| r        | 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。 |
| w        | 打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。 |
| a        | 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| rb       | 以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。 |
| wb       | 以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。 |
| ab       | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| r+       | 打开一个文件用于读写。文件指针将会放在文件的开头。           |
| w+       | 打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。 |
| a+       | 打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写 |
| rb+      | 以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。 |
| wb+      | 以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件 |
| ab+      | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写 |

这些文件打开模式，你不必死记硬背，最常用的有r, w, a+ ，记住这几个就可以了。

### 1.2 读取文件内容

### 1.2.1 readlines

readlines 以列表的形式返回文件里的所有数据，文件有多少行，列表里就有多少个字符串，没一行的换行符也会被读取

```python
f = open('学生信息.txt', 'r')
lines = f.readlines()
print(lines)
```

### 1.2.2 readline

readline一次只读取一行，如果文件特别大，readlines会一次性把数据读取到内存中，这样会非常耗费内存，而readline就不存在这样的问题，但由于一次只读取一行，所以，想要读取全部数据需要使用while循环

```python
f = open('学生信息.txt', 'r')
line = f.readline()
while line:
    print(line)
    line = f.readline()
```

### 1.2.3 使用迭代

open返回的文件句柄，是一个可迭代对象，因此，你可以像迭代列表一样去迭代一个文件

```python
f = open('学生信息.txt', 'r')
for line in f:
    print(line)
```

### 1.2.4 关闭文件

文件打开以后一定要关闭，否则就会出现内存泄漏，关闭文件使用close方法

```python
f = open('学生信息.txt', 'r')
for line in f:
    print(line)

f.close()
```

为了避免忘记close文件，还可以使用with语法，该语法可以保证在with语句块退出时可以自动关闭文件

```python
with open('学生信息.txt', 'r') as f:
    for line in f:
        print(line)
```

## 2. 练习题

### 2.1 题目要求

文件 学生信息.txt 中只有三行数据

```text
小明  15  170
小刚  14  168
小红  15  165
```

请使用上面所学的任意一种读取文件的方法，读取txt文件里的信息，第一列是人名，第二列是年龄，第三列是身高，请设计一种合理的存储方法存储这些信息。

### 2.2 思路分析

现在，数据存储在文件中，题目要求你读取文件数据，本篇教程一共展示了3种读取文件的方法，他们所读取到的数据都是字符串，但文件里的数据每一列都是特定的内容，如果就这样一行一行的存储在字符串里显然不利于操作。最好的办法是将一行数据里的一个人的信息存储到字典里，然后将这些字典存储到列表里。

### 2.3 示例代码

```python
lst = []
with open('学生信息.txt', 'r') as f:
    for line in f:
        arrs = line.strip().split()
        info = {
            'name': arrs[0],
            'age': int(arrs[1]),
            'height': int(arrs[2])
        }
        lst.append(info)


print(lst)
```

### 2.4 难度升级

请实现函数avg，读取文件里的数据计算学生的年龄和身高的平均值

```python
def avg(filename):
    """
    计算文件filename里学生的平均年龄和身高
    :param filename:
    :return:
    """
    pass

avg('学生信息.txt')
```

#### 2.4.1 思路分析

将问题的解决分为两个步骤，第一步，从文件里读取数据，就像2.3中的示例代码那样，将每个学生的数据存储为字典，将所有学生的数据存储到列表中。第二步，遍历列表，计算身高与年龄的平均值。为什么要分为这两个步骤呢，如果一步就全部完成不可以么？

之所以把数据读取和数据分析的代码分开，是因为这是两个完全不同的事情，读取文件专门写成一个函数，文件地址作为参数传入，这样就可以读取其他相同存储方式的文件。将数据分析单独封装成一个函数，与数据读取分离，这样当有新的数据分析的需求时，比如计算学生身高的中位数，就可以再封装一个函数，当你想获取学生身高均值时，就可以将数据读取函数与计算身高均值函数结合在一起使用，当你想获取学生身高中位数时，就可以将数据读取函数与计算身高中位数结合在一起。

#### 2.4.2 示例代码

```python
def read_info(filename):
    """
    从文件中读取学生信息
    :param filename:
    :return:
    """
    lst = []
    with open(filename, 'r') as f:
        for line in f:
            arrs = line.strip().split()
            info = {
                'name': arrs[0],
                'age': int(arrs[1]),
                'height': int(arrs[2])
            }

            lst.append(info)
    return lst


def avg_age_avg_height(lst):
    """
    解析lst,计算平均年龄和身高
    :param lst:
    :return:
    """
    age_sum = 0
    height_sum = 0
    for stu in lst:
        age_sum += stu['age']
        height_sum += stu['height']

    avg_age = age_sum/(len(lst))
    avg_height = height_sum/(len(lst))

    return avg_age, avg_height


def avg(filename):
    """
    计算文件filename里学生的平均年龄和身高
    :param filename:
    :return:
    """
    stu_lst = read_info(filename)
    avg_age, avg_height = avg_age_avg_height(stu_lst)
    print(avg_age, avg_height)

avg('学生信息.txt')
```

## 3. 写文件

写文件，要使用w模式， 写入数据，使用write方法，需要注意的是，write并不自动写入换行，因此需要你自己添加换行

```python
with open('test', 'w') as f:
    for i in range(10):
        f.write(str(i) + "\n")
```

## 4.练习题

### 4.1 找出文件中的相同IP

#### 4.1.1 题目要求

data1文件中有一些IP

```text
192.168.13.112
192.168.13.113
192.168.13.114
192.168.13.115
192.168.13.116
```

data2文件中也有一些IP

```text
192.168.14.112
192.168.14.113
192.168.14.114
192.168.14.115
192.168.14.116
192.168.13.114
192.168.13.115
192.168.13.116
```

请写程序，找出在两个文件中都存在的IP

#### 4.1.2 思路分析

还是将数据读取与数据分析的代码分开来写，数据读取专门写一个函数，接下来要考虑读取文件数据时将数据以何种方式保存。题目要求你找出两个文件中都存在的IP，我建议你将文件里的IP存储到集合中，两个集合求交集，就可以得到共同出现的IP了。

#### 4.1.3 示例代码

```python
def read_ip(filename):
    """
    从文件filename当中读取ip,保存到集合中,并返回
    :param filename:
    :return:
    """
    f = open(filename, 'r')
    lines = f.readlines()
    ip_set = set()

    for line in lines:
        ip_set.add(line.strip())

    return ip_set


def same_ip():
    ip_set_1 = read_ip('data1')
    ip_set_2 = read_ip('data2')

    same_ip_set = ip_set_1.intersection(ip_set_2)
    print(same_ip_set)


if __name__ == '__main__':
    same_ip()
```

### 4.2 统计你的代码行数

#### 4.2.1 题目要求

实现函数get_py_program_count

```python
def get_py_program_count(py_path):
```

py_path是你的python代码保存的地址，这个函数返回py_path目录下所有py脚本的代码行数

#### 4.2.2 思路分析

py_path目录下可能还会有文件夹，文件夹的嵌套层次事先是不知道的，因此你现在需要一个函数，这个函数可以获取一个文件夹下的所有指定文件，不管这个文件夹下有多少层文件夹。

对于一个python脚本而言，空行不算代码，一行内容去掉空格后是以# 开头的也不算代码。你需要一个函数，这个函数可以获得一个python脚本的代码函数。

有了前面这两个函数的帮助，就可以获取py_path目下所有python脚本的代码行数了

#### 4.2.3 示例代码

```python
import os


def get_file_lst(py_path, suffix):
    """
    遍历指定文件目录下的所有指定类型文件
    :param py_path:
    :return:
    """
    file_lst = []
    for root, dirs, files in os.walk(py_path):
        for file in files:
            file_name = os.path.join(root, file)
            if file_name.endswith(suffix):
                file_lst.append(file_name)

    return file_lst


def get_program_line_count(file_name):
    """
    返回单个py脚本的代码行数
    :param filename:
    :return:
    """
    # 存储代码行数
    count = 0
    with open(file_name, 'r', encoding='utf-8')as f:
        for line in f:
            line = line.strip()
            if not line:
                continue

            if line.startswith("#"):
                continue
            count += 1
    return count



def get_py_program_count(py_path):
    # 获取指定目录下所有的python脚本
    file_lst = get_file_lst(py_path, '.py')
    count = 0
    # 遍历列表,获取单个python脚本的代码行数
    for file_name in file_lst:
        count += get_program_line_count(file_name)  # 累加代码函数

    return count   # 返回代码总函数


if __name__ == '__main__':
    py_path = '/Users/kwsy/PycharmProjects/pythonclass'
    print(get_py_program_count(py_path))
```
