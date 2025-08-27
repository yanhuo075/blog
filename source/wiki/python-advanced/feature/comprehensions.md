---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.7 生成式
order: 6
---

# 列表生成式, 字典生成式, 集合生成式

## 1. 列表生成式

列表生成式是python内置的一种创建列表的方法，通过在[]内部执行一行for循环语句，将for循环所遍历到的元素添加到列表中。由于编译后的字节码更少， 因此比普通的采用append方法生成列表要快很多，不仅如此，使用列表生成式编写的代码更加简洁，通过添加if else 语句，列表生成式也能对列表里的元素进行限制。

### 1.1 普通方法生成列表

要求你生成一个列表，列表里有10个元素，索引为奇数的元素值为1，索引为偶数的位置值为0。

```python
lst = []
for i in range(1, 11):
    if i % 2 == 0:
        lst.append(0)
    else:
        lst.append(1)

print(lst)
```

程序输出结果为

```text
[1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
```

### 1.2 列表生成式生成列表

```python
lst = [1 if item % 2 == 1 else 0 for item in range(1, 11)]
print(lst)
```

程序输出结果为

```text
[1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
```

通过对比可以发现，生成同样内容的列表，列表生成式的方法所使用的代码更少，更加简洁，让代码看起来更加的清爽，而这还不是列表生成式的唯一优点。

### 1.3 列表生成式更快

列表生成式比普通的生成列表的方法更快，大部分人知道这个事实，但并不清楚为何会这样，下面我将定义两个函数，分别采用普通方法和列表生成式方法创建一个内容相同的列表,每个函数各调用10000次，并统计耗时

```python
import time

def func1():
    lst = []
    for j in range(1000):
        lst.append(j)


def func2():
    lst = [j for j in range(1000)]

t1 = time.time()
for i in range(10000):
    func1()
t2 = time.time()
print('10000次调用func1耗时{cost}'.format(cost=t2-t1))


t1 = time.time()
for i in range(10000):
    func2()
t2 = time.time()
print('10000次调用func2耗时{cost}'.format(cost=t2-t1))
```

程序执行结果为

```text
10000次调用func1耗时1.4992802143096924
10000次调用func2耗时0.6083369255065918
```

生成同样的内容，调用相同的次数，func1的耗时竟然是func2的近乎三倍，到底是什么原因，让列表生成式更快呢，你需要深入到字节码来一探究竟。

### 1.4 字节码

python不是编译型语言，但还是要被解析成二进制的字节码才能被执行，执行这些字节码的正是python解释器。python中的dis模块，可以查看被编译后的字节码，通过字节码，可以看到普通生成列表的方法和列表生成是生成列表方法之间的根本不同。

```python
import dis

def func1():
    lst = []
    for j in range(1000):
        lst.append(j)


def func2():
    lst = [j for j in range(1000)]

dis.dis(func1)
print('*'*20)
dis.dis(func2)
```

程序执行结果为

```text
  4           0 BUILD_LIST               0
              2 STORE_FAST               0 (lst)

  5           4 SETUP_LOOP              26 (to 32)
              6 LOAD_GLOBAL              0 (range)
              8 LOAD_CONST               1 (1000)
             10 CALL_FUNCTION            1
             12 GET_ITER
        >>   14 FOR_ITER                14 (to 30)
             16 STORE_FAST               1 (j)

  6          18 LOAD_FAST                0 (lst)
             20 LOAD_ATTR                1 (append)
             22 LOAD_FAST                0 (lst)
             24 CALL_FUNCTION            1
             26 POP_TOP
             28 JUMP_ABSOLUTE           14
        >>   30 POP_BLOCK
        >>   32 LOAD_CONST               0 (None)
             34 RETURN_VALUE
********************
 10           0 LOAD_CONST               1 (<code object <listcomp> at 0x1086dadb0, file "/Users/kwsy/kwsy/coolpython/demo.py", line 10>)
              2 LOAD_CONST               2 ('func2.<locals>.<listcomp>')
              4 MAKE_FUNCTION            0
              6 LOAD_GLOBAL              0 (range)
              8 LOAD_CONST               3 (1000)
             10 CALL_FUNCTION            1
             12 GET_ITER
             14 CALL_FUNCTION            1
             16 STORE_FAST               0 (lst)
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE      
```

不要被这些字节码吓到，其实理解他们并不难，这些命令的解释，可以在官方文档中查看,[字节码解释](https://docs.python.org/3.6/library/dis.html)

对比两段字节码，可以发现，根本的不同在于添加元素的操作。

func1 中需要执行（20 LOAD_ATTR ）操作，将append方法加载进来，然后执行（CALL_FUNCTION），执行append方法。func2中添加一个元素时执行的字节码更少，因此速度更快。

## 2. 字典生成式

python的字典生成式通过在大括号{} 内执行一行for循环语句创建一个新的字典，大括号内的语句需要指定键值对的key与value，这两项值都是for循环语句中变量。

```python
class_dict = {
    'c++': 90,
    'python': 93,
    'java': 95,
    'javascript': 96,
    'node.js': 94
}

new_class_dict = {k: v for k, v in class_dict.items() if v >= 95}
print(new_class_dict)
```

根据class_dict生成一个新的字典，筛选key-value对的条件要放到后面，这是语法上的要求。

## 3. 集合生成式

python集合生成式与列表生成式几乎是一模一样的，只需要将[]替换为{} 即可，在{}内执行一个for循环语句，for循环所遍历的元素自动添加到集合之中。

```python
lst = [
    {'name': 'python', 'price': 100},
    {'name': 'c++', 'price': 100},
    {'name': 'java', 'price': 100},
    {'name': 'java', 'price': 100},
    {'name': 'python', 'price': 100},
    {'name': 'node', 'price': 100},
]

name_set = {item['name'] for item in lst}
print(name_set)
```
