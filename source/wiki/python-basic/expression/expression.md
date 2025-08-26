---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 4.10 表达式
order: 9
---

# python表达式，运算符，原子，原语

python表达式有两个重要的概念，原子和原语。原子是原语的一种，他们可以单独构成表达式，但这不是最常见的形式，最常见的表达式形式是他们之间通过运算符进行的各式各样的计算。更通俗一点的定义，表达式是值，变量，运算符三者的组合。

## 1. 大部分人并不真的理解python表达式

表达式是一个乍看起来不那么重要的知识，很多人学习python很久，都未必理解什么是python表达式，若不是想要完善教程，我自己也不会凭着英语4级的水平去翻看那苦涩难懂的英文官方文档。

之所以要重写这部分，是因为带一个学生的过程中，她写出了一个错误的表达式

```text
lst = [1, 2, 3, 5]
for i in range(len(lst)1)
    print(i)
```

她并不理解为什么len(lst)1是一种错误的写法，对于已经熟练掌握python的人，这个事情或许有些可笑，可你未必能解释的清楚，为什么这种写法是错误的，至少，你讲不出令人信服的原理，len(lst)的值是4，那么len(lst)1可不可以理解为41呢？

我最早的主力语言是C++，python算是半路出家，一直以来，都是靠着C++的功底来理解和运用python，从未认真研究过python表达式的概念。我可以写出正确的表达式，但对于什么是python表达式，也并不清楚，于是乎，我花了半个小时的时间阅读了[官方文档](https://docs.python.org/3.6/reference/expressions.html)，结合我自身的理解，简化了官方文档的表述，以便读者更容易理解。

理解python表达式，有两个关键的概念要掌握，原子（Atoms）和原语（Primaries），这两个英文单词，我是通过谷歌翻译得到的。第一眼感觉到很陌生，因为接触python这么久了，从未见过这两个词语，阅读英文文档还是很有必要的，否则，永远只能学习二手资料。

## 2. 原子

原子是组成表达式的最基础的元素，单个原子就可以构成表达式。

```text
atom      ::=  identifier | literal | enclosure
enclosure ::=  parenth_form | list_display | dict_display | set_display
               | generator_expression | yield_atom
```

上面的这两行式子，叫EBNF，扩展的巴科斯范式，BNF（巴科斯范式）是一种形式化的语法表示方法，::=表示赋值，| 表示或， 我们这样来理解它： 原子（atom）可以是identifier，或者literal， 或者enclosure。 对于enclosure， 你可以参照atom来理解。下面，逐一来解释这些词语的含义，并给出具体的例子

### 2.1 identifier 标识符

你自己定义的变量，就属于标识符，此外函数名，类名，都是标识符的一种，他们都属于原子。

```python
a = 4
a
```

第二行的a 就构成一个表达式，尽管它及其简单，但的确算是一个表达式，表达式的值是4，这里一定要牢记一个概念，表达式是有结果的，这个结果我们称之为表达式的值，这一点对于你理解程序极为关键。

### 2.2 literal

```text
literal ::=  stringliteral | bytesliteral
             | integer | floatnumber | imagnumber
```

literal ，谷歌翻译成文字，百度翻译成字面意义的或者字面量，后者更贴切一点，python中的字符串，字节串，整型数据，浮点型数据，复数，随意拿出来一个具体的值，都属于literal

```python
"python"
b"python"
12
23.5
123-12j
```

你直接观察到的这些数值，都可以单独的构成表达式，他们都是原子，看下面这行代码

```python
print(23.5)
```

print函数里的23.5是表达式么？是的，放在哪里都是。

### 2.3 enclosure

这个单词，我没有找到准确的翻译，根据EBNF描述，可以理解为小括号，方括号或花括号中包含的形式。

#### 2.3.1 小括号

```text
()              # 空元组
(2, )           # 有一个元素的元组
(2, 4, 5)       # 有三个元素的元组
(6)             # 小阔号与6构成一个表达式，表达式的值是6
```

#### 2.3.2 列表，字典，集合的直接展示

```python
[2, 3, 5]               # 列表
{2, 5, 6}               # 集合
{"name": '小明'}        # 字典
```

这种形式和literal 就很像了，都是某个类型的数值，你可以直接观察它的值，这些都可以单独的构成表达式

#### 2.3.3 推导式

推导式，虽然不能直接观察其值，但可以通过运算获得其内容

```text
[i for i in range(10)]
{i for i in range(10)}
{i: i+1 for i in range(10)}
```

#### 2.3.4 生成器表达式

```text
(i for i in range(10))
```

推导式可以计算出列表，集合，字典，但上面的表达式的值不是元组，而是生成器

#### 2.3.5 yield 表达式

前面讲的四种原子，都和括号有关，yield 却是一个和括号无关的表达式，不知为何官方要划分到一起

```python
def gen():  
    yield 123       # yield 表达式
```

## 3. 原语

原语表示编程语言最紧密的绑定操作，原语稍稍比原子复杂一点，原子是原语5种形式中的一种

```text
primary ::=  atom | attributeref | subscription | slicing | call
```

### 3.1 attributeref 属性引用

```text
attributeref ::=  primary "." identifier
```

属性引用，是极为常见的操作

```python
class Dog():
    def __init__(self, name):
        self.name = name


dog = Dog('二哈')
dog.name
```

dog.name 就是一种attributeref形式，它是原语中的一种，可以单独构成表达式，表达式的值是“二哈”

### 3.2 subscription

```text
subscription ::=  primary "[" expression_list "]"
```

谷歌和百度都将subscription翻译成订阅，可我认为这样翻译词不达意，无奈自己也没有好的解释，还是通过代码来理解体会吧

```python
a = [1, 3, 4, 5]
a[3]        # 5
```

a[3] 就是一种subscription形式，可以单独构成一个表达式

### 3.3 Slicings

切片，这是比较熟悉的操作

```python
a = [1, 3, 4, 5]
a[1:2]      # [3]
```

### 3.4 call

任何可调用对象，函数，方法，类，实现了__call__的实例在被调用时，单独构成表达式

```python
def func():
    return 3

func()          # 表达式

class Dog():
    def __init__(self, name):
        self.name = name

Dog("二哈")      # 表达式
```

这个应该算是比较容易理解的，可能有人会疑惑，下面的代码怎么理解

```python
dog = Dog("二哈") 
```

这行代码是一条赋值语句，等号左侧是变量dog， 等号右侧是表达式。

## 4. await 表达式，条件表达式，lambda表达式

我把这3种表达式拿出来单独介绍，是因为他们3个不好归类。

先来看**await表达式**，想要写出这种表达式，你需要定义一个异步函数

```python
import asyncio

async def count():
    print("One")
    await asyncio.sleep(1)
    print("Two")

async def main():
    await asyncio.gather(count(), count(), count())

asyncio.run(main())
```

上面代码里以await开头的语句都是await表达式

**条件表达式**， 也就是三元表达式

```python
5 if 4 > 3 else 6       # 表达式的值是5
```

**lambda表达式**

```python
lst = [1, 2, 3, 4, 5, 6]

filter_lst = filter(lambda x: x % 2 == 0, lst)

for item in filter_lst:
    print(item)
```

lambda x: x % 2 == 0 就是lambda表达式

## 5. 原语之间进行计算

前面已经了解了表达式的两个最核心的概念，原子和原语，原子是原语的一种，他们可以单独构成表达式，但这不是最常见的形式，最常见的形式是他们之间进行格式各样的运算，这些计算包括：

1. 算数运算
2. 比较关系运算
3. 逻辑运算
4. 位运算
5. 成员运算
6. 身份运算

关于这些运算，本文不做详细介绍，你可以阅读本教程第4章[运算符与表达式](http://www.coolpython.net/python_primary/expression/index.html)

**原语之间的运算，构成了更大的表达式**

```python
3 + 5               # 表达式
int("4") - float("5.5")   # 表达式

def func1():
    return 7

def func2():
    return 5

func1() - func2()    # 表达式
```

## 6. 最大化原则

这是我自己总结定义的原则，在你理解代码里的表达式时，应当遵循最大化原则，比如下面的表达式

```python
4 + 5 + 6
5 if 4 > 3 and 5 > 4 else 8
```

单独一个原子或原语可以构成一个表达式，以4 + 5 + 6 为例，难道要理解成3个表达式么？显然不能，4 + 5 也构成一个表达式，5 + 6 也可以构成表达式，如果这样理解表达式，实在太混乱了。

这些原子，原语能够组成的最大的表达式，即为最终我们认定和理解的表达式，不对表达式进行拆分。

## 7. 理解len(lst)1

不可以将len(lst)1 理解为41，因为它不是一个正确的表达式，下面是推导结论的过程：

1. len是函数，属于可调用对象，len(lst) 是基本的原语，参见3.4
2. 1是原子中的literal形式，原子是原语的一种，参见2.2
3. 原语之间进行运算，可以构成表达式，参见第5小节
4. len(lst)1，两个原语仅仅相邻，这种形式不属于任意一种表达式，因此，它不是合法的表达式。

## 8. 表达式可计算，有结果

表达式可计算，有结果，这不是要求，而是既定事实，你不可能写出一个没有值的表达式，任何时候，将你所写的表达式放入print函数中输出，都会输出一个明确的结果

```python
print(表达式)       # 一定有内容输出
```

强调表达式可计算，有结果，这是理解代码的关键之处，代码在执行时，必须对表达式求值，那么你在理解代码时，就必须在表达式可求值的基础上进行代码阅读。

如果你不清楚表达式的值，也就无法理解代码的行为，所谓代码的行为不过是一系列逻辑的表达，而任何逻辑都是基于数据的，表达式的本质就是对数据的描述。

举一个简单的例子，判断年份是否为闰年

```python
year = int(input("输入一个年份: "))

if year % 4 == 0:
    if year % 100 == 0:
        if year % 400 == 0:
           print("{0} 是闰年".format(year))     # 整百年能被400整除的是闰年
        else:
            print("{0} 不是闰年".format(year))
    else:
        print("{0} 是闰年".format(year))        # 非整百年能被4整除的为闰年
else:
   print("{0} 不是闰年".format(year))
```

1. year % 4 == 0
2. year % 100 == 0
3. year % 400 == 0

这3个表达式的值，要么为True，要么为False， 程序将根据这3个表达式的值决定执行哪个print语句。不管程序看起来多么复杂，都遵循从上之下逐行执行的规则，这3个表达式的值，决定了进入哪个逻辑分支，执行哪个语句块，所以，理解程序的第一步是理解表达式。
