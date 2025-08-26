---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 3.9 字典
order: 8
---

# 字典

## 1. 什么是字典

字典并不是什么全新的概念，早在上小学时，你就已经接触过字典，没错，就是《新华字典》，新华字典的结构和python语言中的字典，在结构上是一样的。

咱们以读音来查找一个汉字，比如"张"这个字，读音是zhang,一声，你一定可以在字典里找到与它对应的页数，假设是第100页，那么zhang 和 100之间就有一个映射关系，你知道了zhang，也就知道了100。

另一个较为常见的例子是手机通讯录，你想找一个人的电话时，你应该在通讯录里找到这个人的名字，然后点进去查看它的电话号，姓名和电话号之间存在着映射关系，你知道姓名，就知道电话号，下面这张图展示了字典的数据结构

![python字典(dict)的结构](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822155925791.jpeg)
左侧是key,右侧是value。



# python字典(dict)详解

## 1. 字典的定义

python的字典(dict)属于映射类型，是数据的无序集合，字典内的元素都是key:value形式的键值对，所有键值对存储在一对大花括号{}中，每一个键值对之间使用逗号分隔。

```text
{'name': '小明', 'age': 14}
```

字典的key不允许重复，如果向字典中插入重复的key，新的value会替换旧的value。

当数值类型做为字典的key时遵循数字比较的一般规则，如果数值相等则视为同一个key，例如1 和 1.0 ，他们使用比较运算符 == 进行比较时是相等的，但考虑到计算机对浮点型数据存储的是近似值，因此用float类型数据做字典的key是不明智的。

此外，尽量避免使用bool类型对象做字典的key，bool类型是int类型的子类，True与1在python中是相等的，因此下面的字典看似定义了两个key:value对，实际只有一个

```python
int_dict = {
    1: '1做key',
    True: 'True做key'
}

print(int_dict)         # {1: 'True做key'}
```

后定义的True: 'True做key' 覆盖了之前定义的1: '1做key'。

任意类型的对象都可以做python字典的value，但只有可hash的对象才能做字典的key，像列表，字典，集合都不是可hash的对象，因此他们不能做字典的key。判断是一个对象是否可hash，可以使用内置函数hash，如果对象是可hash的，hash函数就能够返回对象的hash值，若不可hash则会抛出TypeError异常。

## 2. 创建python字典

### 2.1 使用{}创建字典

使用一对大括号{} ，可以创建一个空字典，字典里没有任何元素。

```python
empty_dict = {}
```

在{}里添加key:value对，可以创建非空的字典

```python
contacts_dict = {
    "小王": '13892339876',
    "小张": '13898320987',
    "小李": '13890348745'
}
```

字典中元素的个数是3，一个key:value对算是一项元素，这个字典里的key和value都是字符串类型，想要获得字典里元素的个数，需要使用内置函数len

```python
print(len(contacts_dict))   # 3
```

### 2.2 使用内置函数dict创建字典

dict函数的语法如下

```python
class dict(**kwarg)
class dict(mapping, **kwarg)
class dict(iterable, **kwarg)
```

**参数说明：**

- kwarg 关键字参数
- mapping 关联式的容器类型
- iterable 可迭代对象

dict函数返回一个新的字典，下面是使用示例

**传入一个字典**

```python
>>> new_dict = dict({'name': '小明', 'age': 14})
>>> new_dict
{'name': '小明', 'age': 14}
```

**传入关键字参数**

```python
>>> new_dict = dict(name='小明', age=14)
>>> new_dict
{'name': '小明', 'age': 14}
```

**传入可迭代对象**

```python
>>> new_dict = dict([('name', '小明'), ('age', 14)])
>>> new_dict
{'name': '小明', 'age': 14}
```

## 3. 字典新增键值对

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

score_dict['小丽'] = 100
```

通过对字典的一个key进行赋值可以向字典中插入新的key:value对，字典在使用key时，必须要用中括号[]紧跟在字典后面，在中括号里填写key的值，这种方式被称为索引方式。

## 4. 修改字典

修改字典里某个key所对应的value值，写法与向字典里新增key:value对是相同的，都需要通过使用[]的索引方式进行。

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}
score_dict['小明'] = 100
print(score_dict)
```

在定义字典score_dict时，已经有一个key的值是'小明'， score_dict['小明'] = 100 会修改'小明' 所对应的value，print语句最终输出的结果

```python
{
    '小明': 100,
    '小刚': 98,
    '小红': 94
}
```

稍作总结，当使用如下的语法时

```text
dict[key] = value
```

如果key已经存在，那么这行代码的作用就是在修改key所对应的value，如果key不存在，这行代码的作用就是向字典里插入一个新的key:value对。

## 5. 删除字典里的元素

删除字典里的某个key:value,可以使用del关键字。

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

del score_dict['小明']
print(score_dict)           # {'小刚': 98, '小红': 94}
```

此外，也可以使用字典的pop方法

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

score_dict.pop('小明')
print(score_dict)           # {'小刚': 98, '小红': 94}
```

不论是del还是pop方法，都需要指明要删除哪个key，如果想要删除全部的元素，则可以使用clear方法，clear会清空字典里所有的元素。

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

score_dict.clear()
print(score_dict)           # {}
```

执行clear方法后，score_dict变为一个空字典。

## 6. 获取字典里的值

![python 字典](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250824013152769.jpeg)

上小学时我们都有过查字典的经历，我们想要知道一个汉字在字典里的哪一页，可以通过拼音音节索引来查找，拼音音节就相当于python字典里的key，音节所对应的页数就相当于python字典里的value。

任何时候，我们对字典里的元素的访问，修改，删除，都必须通过key来进行。

想要获取某个特定的key所对应的value值，可以通过dict[key] 这种语法来实现

```python
score_dict = {
    '小明': 99,
    '小刚': 98,
    '小红': 94
}

print(score_dict['小红'])       # 94
```

需要特别指出的是，通过key获取value时，如果key不存在，会引发KeyError异常

```python
print(score_dict['小丽'])
```

字典中并没有'小丽'这个key，因此这样代码会引发异常

```text
Traceback (most recent call last):
  File "/Users/kwsy/PycharmProjects/pythonclass/mytest/demo.py", line 7, in <module>
    print(score_dict['小丽'])
KeyError: '小丽'
```

KeyError表示不存在'小丽'这个key

## 7 遍历字典

遍历字典的key与value是常见的操作，写法很多，下面两种写法都能够实现对字典的遍历。

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

for key in score_dict:
    print(key, score_dict[key])
```

这种for循环写法会遍历字典的key，通过key获得所对应的value，另外一种写法是通过字典的items方法来同时遍历字典的key与value

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

for key, value in score_dict.items():
    print(key, value)
```

两种写法最终输出的结果一致

```text
小明 96
小刚 98
小红 94
```

## 8. 字典常用方法

| 方法                                                         | 功能作用                  |
| ------------------------------------------------------------ | ------------------------- |
| [clear()](http://www.coolpython.net/method_topic/dict/dict-clear.html) | 删除字典内所有的元素      |
| [copy()](http://www.coolpython.net/method_topic/dict/dict-copy.html) | 返回字典的浅复制          |
| [fromkeys()](http://www.coolpython.net/method_topic/dict/dict-fromkeys.html) | 以指定key创建一个新的字典 |
| [get()](http://www.coolpython.net/method_topic/dict/dict-get.html) | 返回指定key的值           |
| [items()](http://www.coolpython.net/method_topic/dict/dict-items.html) | 成对返回所有key和value    |
| [keys()](http://www.coolpython.net/method_topic/dict/dict-keys.html) | 返回字典所有的key         |
| [values](http://www.coolpython.net/method_topic/dict/dict-values.html) | 返回字典所有value         |
| [setdefault()](http://www.coolpython.net/method_topic/dict/dict-setdefault.html) | 为key设置对应的默认值     |
| [update()](http://www.coolpython.net/method_topic/dict/dict-update.html) | 更新字典                  |
| [pop()](http://www.coolpython.net/method_topic/dict/dict-pop.html) | 删除键值对                |



# python嵌套字典

同嵌套列表一样，python的字典作为一个容器，可以在字典里存储字典，类似下面的形式

```python
stu_dict = {
    'name': '小明',
    'age': 12,
    'score': {
        '语文': 90,
        '数学': 98
    }
}
```

在嵌套列表，嵌套字典中，对于嵌套的层次，没有任何要求，只要你自己能理清嵌套逻辑，你想嵌套多少层都可以。

如果你想获取小明的语文分数，那么就需要逐层的来获取value

```python
print(stu_dict['score']['语文'])
```

stu_dict['score']的值是

```python
{
    '语文': 90,
    '数学': 98
}
```

仍然是一个字典，想要获取语文分数，就继续使用[]操作符来获取key所对应的value



# python字典方法介绍

| 方法                                                         | 功能作用                  |
| ------------------------------------------------------------ | ------------------------- |
| [clear()](http://www.coolpython.net/method_topic/dict/dict-clear.html) | 删除字典内所有的元素      |
| [copy()](http://www.coolpython.net/method_topic/dict/dict-copy.html) | 返回字典的浅复制          |
| [fromkeys()](http://www.coolpython.net/method_topic/dict/dict-fromkeys.html) | 以指定key创建一个新的字典 |
| [get()](http://www.coolpython.net/method_topic/dict/dict-get.html) | 返回指定key的值           |
| [items()](http://www.coolpython.net/method_topic/dict/dict-items.html) | 成对返回所有key和value    |
| [keys()](http://www.coolpython.net/method_topic/dict/dict-keys.html) | 返回字典所有的key         |
| [values](http://www.coolpython.net/method_topic/dict/dict-values.html) | 返回字典所有value         |
| [setdefault()](http://www.coolpython.net/method_topic/dict/dict-setdefault.html) | 为key设置对应的默认值     |
| [update()](http://www.coolpython.net/method_topic/dict/dict-update.html) | 更新字典                  |
| [pop()](http://www.coolpython.net/method_topic/dict/dict-pop.html) | 删除键值对                |



# python字典最佳实践

本篇教程部分内容超出现在所学知识范围，但并没有超出你的能力范围，对于字典的遍历，我将放到for循环章节中进行讲解，对于新知识，你应当抱有学习的热情，主动搜索资料学习他们，积极探索，是掌握一门编程语言的最佳路径。

## 1. 直接取值有风险，建议使用get方法

直接从字典里取值，面临一定的风险，比如下面的代码

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

print(score_dict['小丽'])
```

程序会报错

```text
Traceback (most recent call last):
  File "/Users/zhangdongsheng/experiment/test/test.py", line 7, in <module>
    print(score_dict['小丽'])
KeyError: '小丽'
```

使用一个在字典中不存在的key，会引发异常，为了避免这种异常，通常有两种方法，先来说第一种方法

### 判断key是否存在

在使用key之前，先判断这个key是否存在，存在了取值，不存在则根据业务要求采取对应的措施

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

if '小丽' in score_dict:
    print(score_dict['小丽'])
else:
    print("字典里没有小丽的分数")
```

### 使用get方法

字典的get方法是非常实用的方法,该方法可以指定两个参数,一个是key，另一个是key不存在时返回的数据，默认范围None

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

print(score_dict.get('小丽'))
```

上面的代码中，要获取'小丽'这个key所对应的value，但是'小丽'不存在于字典中，因此get方法默认返回None,如果你希望get方法在key不存在时返回0，你可以这样写

```python
print(score_dict.get('小丽', 0))
```

## 2. 不要用bool类型数据做key

bool类型是int类型的子类，1和 Ture是相等，0和False是相等的，如果使用字典时，1已经做了key，那么你再用True去做key，就会修改1所对应的value，下面是一个简单示例

```python
int_dict = {
    1: '1做key',
    True: 'True做key'
}

print(int_dict)
```

执行代码，输出结果为

```text
{1: 'True做key'}
```

## 3. 使用dict.setdefault方法

一个列表里存放了若干个单词

```python
lst = ['']
```

现在要求你统计每个单词出现的次数，并将单词与单词出现的次数存储到字典中

你可以这样实现代码

```python
lst = ['green', 'white', 'black', 'white', 'green', 'green']

info = {}

for word in lst:
    if word not in info:
        info[word] = 0

    info[word] += 1

print(info)
```

上面的代码虽然完成了要求，但是不够优雅，不符合python一贯的简洁作风，你应该使用setdefault方法让代码看起来更加简洁

```python
lst = ['green', 'white', 'black', 'white', 'green', 'green']

info = {}

for word in lst:
    info.setdefault(word, 0)
    info[word] += 1

print(info)
```

setdefault方法尝试为key设置默认值，如果这个key已经存在，那么setdefault什么都不做，如果key不存在，则向字典里新增一个key-value键值对，value就是方法里的第二个参数。

上面的代码里，如果单词不存在于字典中，则设置这个单词所对应的value为0，info[word] += 1 等价于 info[word] = info[word] + 1 ，实现了单词出现次数加1的目的

## 4. 使用items()方法遍历字典

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}
```

现在要求你输出字典里每一个学生的姓名和分数，你可以使用for循环来完成这个题目

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

for key in score_dict:
    print(key, score_dict[key])
```

上面的代码，不够优雅，建议你使用items()方法来实现遍历

```python
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

for key, value in score_dict.items():
    print(key, value)
```

代码看起来是不是简明一些呢，省去了使用key获取value的过程，items()方法返回一个可迭代对象，使用for循环遍历这个可迭代对象时，得到的是一个元组，元组内包含key和 value

下面的代码向你揭示items()方法返回的对象的本质面目

```python
from collections import Iterable
score_dict = {
    '小明': 96,
    '小刚': 98,
    '小红': 94
}

iter_obj = score_dict.items()
print(isinstance(iter_obj, Iterable))

for item in iter_obj:
    print(item)
```

程序输出结果为

```python
True
('小明', 96)
('小刚', 98)
('小红', 94)
```



# python字典练习题

通过几道python字典(dict)练习题来巩固对字典的掌握, 考察你对python字典常用方法的理解和使用,比如keys(), values(),如何判断一个key是否在字典中, 如何用字典来存储并表示数据

### 1. 字典基本操作

字典内容如下

```python
dic = {
    'python': 95,
    'java': 99,
    'c': 100
}
```

用程序解答下面的题目

1. 字典的长度是多少
2. 请修改'java' 这个key对应的value值为98
3. 删除 c 这个key
4. 增加一个key-value对，key值为 php, value是90
5. 获取所有的key值，存储在列表里
6. 获取所有的value值，存储在列表里
7. 判断 javascript 是否在字典中
8. 获得字典里所有value 的和
9. 获取字典里最大的value
10. 获取字典里最小的value
11. 字典 dic1 = {'php': 97}， 将dic1的数据更新到dic中

第1题，len(dic),结果为3
第2题，dic['java'] = 98,对字典里value的修改，必须通过key才可以
第3题，del dic['c']
第4题，dic['php'] = 90
第5题，lst = list(dic.keys())
第6题，lst = list(dic.values())
第7题，'javascript' in dic
第8题，sum(dic.values())
第9题，max(dic.values())
第10题，min(dic.values())
第11题，dic.update(dic1)

### 2. 字典应用（买水果）

小明去超市购买水果，账单如下

```text
苹果  32.8
香蕉  22
葡萄  15.5
```

请将上面的数据存储到字典里，可以根据水果名称查询购买这个水果的费用

很简单哦，用水果名称做key，金额做value，创建一个字典

```python
info = {
    '苹果':32.8,
    '香蕉': 22,
    '葡萄': 15.5
}
```

### 3. 字典应用（买水果2）

小明，小刚去超市里购买水果

小明购买了苹果，草莓，香蕉，一共花了89块钱，，小刚购买了葡萄，橘子，樱桃，一共花了87块钱

请从上面的描述中提取数据，存储到字典中，可以根据姓名获取这个人购买的水果种类和总费用。

以姓名做key，value仍然是字典

```python
info = {
    '小明': {
        'fruits': ['苹果', '草莓', '香蕉'],
        'money': 89
    },
    '小刚': {
        'fruits': ['葡萄', '橘子', '樱桃'],
        'money': 87
    }
}
```
