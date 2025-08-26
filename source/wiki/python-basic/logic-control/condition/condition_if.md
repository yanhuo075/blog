---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.3 if语句
order: 2
---

# python if条件语句详解

## if条件语句定义

python if条件语句是流程控制语句，if 语句包含零个或多个 elif 子句及可选的 else 子句，关键字elif 是else if的缩写，可以避免过多的缩进，python里没有else if 这种语法。if语句的核心作用是控制程序走向哪一个逻辑分支。

**下面是if语句的伪代码表示形式**。

```python
if test :
    suite 
(
elif test :
    suite
)* 
[
else :
    suite
]
```

- if关键字后面必须紧跟一个表达式，这里用test表示
- elif关键字后面也必须紧跟一个表达式
- if , elif , else 各自产生一个逻辑分支，也就是suite部分
- 在伪代码里，elif 子句我用小括号括起来并使用*来表示这部分可以有0或者多个
- else语句用中括号括起来，这表示可选，可以有0或者一个else

**if语句的执行过程如**下：

1. 从上到下，依次判断test表达式是否成立，如果成立，则进入对应的逻辑分支执行代码
2. 表达式的计算结果如果是True，那么表达式就是成立的，反之，表达式不成立。
3. if,elif, else 各自形成逻辑分支，程序在执行时，只能进入其中的某一个逻辑分支，无法进入两个或两个以上。

## if语句的三种形式

从形式上看，if语句可以分为3中形式：

1. if statement
2. if-else
3. if-elif-else

**if statement**，这种形式的if语句只有一个if关键字，当满足某个条件时，程序执行对应的逻辑分支。

```python
value = input("请输入一个正整数:")
i_value = int(value)
if i_value % 2 == 0:
    print("你输入的整数是:{value}, 它是偶数".format(value=value))
```

当用户输入的正整数是偶数时，表达式i_value % 2 的结果是True，条件成立，进入if语句的逻辑分支输出它是一个偶数。如果用户输入的是一个奇数，表达式i_value % 2 的结果是False，条件不成立，程序不做任何操作。

如果想对输入的奇数做响应，那么应当使用**if-else** 这种if语句形式。

```python
value = input("请输入一个正整数:")
i_value = int(value)
if i_value % 2 == 0:
    print("你输入的整数是:{value}, 它是偶数".format(value=value))
else:
    print("你输入的整数是:{value}, 它是奇数".format(value=value))
```

用户输入的整数是偶数时，表达式i_value % 2 成立，进入if关键所对应的逻辑分支，输出它是偶数；用户输入的整数是奇数时，表达式i_value % 2 不成立，则进入到else所对应逻辑分支，输出它是奇数。

当对应多个条件有多个逻辑分支时，则应使用**if-elif-else** 形式的if语句来处理，比如下面的代码

```text
temperature = input("请输入今天的温度:")
temperature = int(temperature)
if temperature < 0:
    print("寒冷")
elif temperature < 15:
    print("微冷")
elif temperature < 25:
    print("温暖")
else:
    print("炎热")
```

我将温度划分为4个区间

1. 小于0， 寒冷
2. 大于等于0小于15度，微冷
3. 大于等于15小于25，温暖
4. 大于等于25，炎热

程序将根据用户输入的温度判断并输出温度输出哪一个区间。if语句在执行时，从上至下依次判断下面3个表达式是否成立

1. temperature < 0
2. temperature < 15
3. temperature < 25

如果表达式1成立，就进入if关键字所对应的逻辑分支输出寒冷，注意，此刻已经有一个条件表达式成立了，那么其他的条件表达式就不会再判断了。

如果表达式1不成立，继续判断表达式2是否成立，此时已经可以断定temperature是大于等于0的，因此第二个表达式里我没有特别的注明大于等于0，其实也可以写成 0 <= temperature < 15，只是从逻辑上看是毫无必要的，因此只有当temperature大于等0时才有机会判断第二个表达式是否成立。

如果表达式2也不成立，则计算判断表达式3，如果表达式3也不成立，则直接进入到else所对应的逻辑分支，temperature 必然是大于等于25。

## if语句如何判断条件表达式是否成立

python的if语句在判断条件表达式是否成立时有自己特殊的规则，我们先来看常规的情况

```python
if 5 > 3:
    pass
```

表达式 5 > 3 的计算结果是True，属于bool类型，此时表达式成立，这与其他编程语言并无区别，python的特殊之处在于它并不要求if语句中的表达式计算结果必须是bool类型，比如下面的if语句

```python
lst = []
if lst:
    print("lst不是空列表")
```

表达式lst的计算结果就是lst本身，属于列表类型，这在其他编程语言里可能是个语法错误，但python却支持这种写法，那么它是如何处理的呢，方法其实很简单，使用bool内置函数对表达式的结果进行换换，如果转换结果是True，条件表达式成立，反之条件表达式不成立。

bool([]) 的结果是False，bool函数传入的参数如果是空列表，空集合，空字符串，空字典，None，0， False，函数的返回值就是False，除此以外都是True。

## 使用程序流程图来理解if语句执行过程

通过绘制程序的流程图，你能更加清晰的理解程序的执行过程

```python
value = input("请输入一个正整数:")
i_value = int(value)
if i_value % 2 == 0:
    print("你输入的整数是:{value}, 它是偶数".format(value=value))
else:
    print("你输入的整数是:{value}, 它是奇数".format(value=value))
```

我们来画出这段程序的流程图
![python if语句](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822172820324.jpeg)

这段程序比较简单，我们来看复杂一点的，程序要求如下：
使用input函数接收用户的输入，用户输入一种颜色，如果是红色，程序输出“红灯停”,如果是绿色，则输出“绿灯行”,如果是黄色，则输出“等一等”，如果是其他颜色，则输出“无法识别”。

示例代码

```python
value = input("输入一种颜色:")
if value == '红色':
    print("红灯停")
elif value == '绿色':
    print("绿灯行")
elif value == '黄色':
    print("等一等")
else:
    print("无法识别")
```

程序流程图如下
![python if语句](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822172806069.jpeg)
