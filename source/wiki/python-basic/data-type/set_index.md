---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 3.10 集合
order: 9
---

# 集合

编程语言中的集合概念，与初中时数学课上所学的集合概念相同。集合里不会存在重复数据，因此可以用来对数据进行去重处理，两个集合之间可以进行交集运算，并集运算，差集运算。

集合中的元素有3个特性：

1. 确定性：给定一个集合，任何对象是不是这个集合的元素是确定的了。
2. 互异性：集合中的元素一定是不同的。
3. 无序性：集合中的元素没有固定的顺序。



# python 集合(set) 详解

## 1. python集合定义

python的集合是一个无序且没有重复元素的序列，集合中的元素必须是可hash对象。集合不记录元素位置和插入顺序，因此，也不支持索引，切片等其他序列类的操作。

定义集合可以使用{} 或者 set()，但定义空集合不能使用{} ，因为{ } 是创建空字典的方法，定义空集合可以使用set()。

下面分别用{} 和 set() 演示如何创建集合

```python
language_set = {'java', 'c', 'python'}      # 定义集合
name_set = set()            # 定义空集合
fruits_set = set(['apple', 'orange', 'banana', 'apple'])  # 定义时有重复元素

print(language_set)     # {'c', 'python', 'java'}
print(name_set)         # set()
print(fruits_set)       # {'banana', 'orange', 'apple'}
```

在定义fruits_set时，字符串apple存在重复的情况，但python集合会自动去重，因此在输出时，apple并没有重复。**去除重复元素是集合的一项重要功能，此外还有成员检测，数学上集合类的计算，例如交集，并集，差集等操作。**

## 2. python集合常用基础操作

### 2.1 add，update方法添加新元素

add方法向集合中添加新的元素，语法如下：

```python
s.add(element)
```

add方法将element添加到集合s中，如果集合s已经包含了element，则不进行任何操作，集合中的元素具有唯一性，绝不会出现重复的情况。

```python
language_set = {'java', 'c', 'python'}      # 定义集合
language_set.add('c++')
print(language_set)         # {'python', 'java', 'c++', 'c'}
```

update方法同样可以向集合中添加新的元素，且可以一次性添加多个，update方法的本质是先计算两个集合的并集，在用并集更新原集合。

update语法如下：

```python
s.update(seq)
```

seq参数可以是列表，元素，集合

```python
language_set = {'java', 'c', 'python'}      # 定义集合
language_set.update(['node.js', 'php'])     # 传入列表
print(language_set)     # {'python', 'php', 'java', 'c', 'node.js'}

language_set.update(('javascript', 'c#'))   # 传入元组
print(language_set)     # {'python', 'php', 'javascript', 'java', 'c', 'c#', 'node.js'}

language_set.update({'vb', 'go'})
print(language_set)     # {'python', 'java', 'c', 'javascript', 'c#', 'go', 'vb', 'php', 'node.js'}
```

### 2.2 remove，discard， pop方法删除集合元素

python的集合提供了3种删除元素的方法，分别是remove,discard, pop。remove方法语法是

```python
s.remove(element)
```

remove方法将element从集合中删除，需要注意的是如果element不在集合中，remove方法将会发生错误

```python
>>> language_set = {'java', 'c', 'python'}
>>> language_set.remove('c')
>>> language_set.remove('c++')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'c++'
```

remove是一种不安全的删除集合元素的方法，想要安全的删除集合元素，可以使用discard方法，当被删除的元素不存在时discard不会发生错误，discard语法如下：

```python
s.discard(element)
```

示例代码

```python
>>> language_set = {'java', 'c', 'python'}
>>> language_set.discard('c')
>>> language_set.discard('c++')
>>> language_set
{'java', 'python'}
```

remove和discard方法都没有返回值，它们原地修改集合。

pop方法比较特殊，它的语法定义如下

```text
s.pop()
```

pop方法没有参数，它随机的删除一个元素并返回该元素，当集合为空时，pop方法会引发KeyError异常。

```python
>>> language_set = {'java', 'c', 'python'}
>>> language_set.pop()
'java'
>>> language_set.pop()
'c'
>>> language_set.pop()
'python'
>>> language_set.pop()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'pop from an empty set'
```

### 2.3 获取集合元素个数

获取集合中元素的个数，使用len内置函数

```python
>>> language_set = {'java', 'c', 'python'}
>>> len(language_set)
3
```

### 2.4 获取集合中的元素

由于集合不支持索引，因此无法像列表那样通过索引操作来获取元素，只能通过for循环遍历集合

```python
language_set = {'java', 'c', 'python'}      # 定义集合

for language in language_set:
    print(language)
```

程序输出结果

```text
java
python
c
```

### 2.5 clear方法清空集合

语法定义

```text
s.clear()
```

clear方法会将集合中所有元素都删除

```python
>>> language_set = {'java', 'c', 'python'}
>>> language_set.clear()
>>> language_set
set()
```

### 2.6 判断元素是否在集合中

判断一个元素是否在集合中需要使用成员运算符 in

```python
>>> language_set = {'java', 'c', 'python'}
>>> 'java' in language_set
True
>>> 'c++' in language_set
False
```

## 3. 集合交集，并集，差集运算

python集合最常用的3个运算是求集合的交集，并集，差集，它们在编程中的作用非常重要。

### 3.1 集合交集

交集运算返回一个新的集合，新集合里的元素存在与所有参与计算的集合中，语法定义如下

```python
s.intersection(set1, set2 ... etc)
```

intersection方法可以传入多个集合，最少传入一个集合，因此set1是必须要传入的，返回的新集合中的元素既在s中，也在set1,set2 ... 中。

```python
>>> language_set_1 = {'java', 'c', 'python'}  
>>> language_set_2 = {'java', 'php'}
>>> intersection_set = language_set_1.intersection(language_set_2)
>>> intersection_set
{'java'}
>>> intersection_set = language_set_2.intersection(language_set_1)
>>> intersection_set
{'java'}
```

字符串java既在language_set_1中，也在language_set_2中，language_set_1调用intersection 和 language_set_2 调用intersection方法，得到的交集是相同的。

### 3.2 集合并集

并集运算返回一个新的集合，新集合中的元素包含了所有参与运算的集合的元素，你可以理解为将所有集合的元素放在一起组成了一个新的集合。

语法定义

```text
s.union(set1, set2...)
```

union方法允许传入多个集合，set1必传，因为至少需要传入一个集合参与计算。

```python
>>> language_set_1 = {'java', 'c', 'python'}
>>> language_set_2 = {'java', 'php'}
>>> union_set = language_set_1.union(language_set_2)
>>> union_set
{'java', 'c', 'php', 'python'}
```

### 3.3 集合差集

差集运算返回一个新集合，差集运算的结果与运算顺序有关，比如两个集合s1 和 s2，s1对s2求差集的结果与s2对s1求差集的结果是不相同的，除非s1与s2完全相同，他们的差集是一个空集合。

差集语法定义如下

```text
s1.difference(s2)
```

实例代码

```python
>>> language_set_1 = {'java', 'c', 'python'}
>>> language_set_2 = {'java', 'php'}
>>> diff_set = language_set_1.difference(language_set_2)
>>> diff_set
{'c', 'python'}
>>> diff_set = language_set_2.difference(language_set_1)
>>> diff_set
{'php'}
```

## 4. 集合常用方法

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [add()](http://www.coolpython.net/method_topic/set/set-add.html) | 为集合添加新元素                                             |
| [clear()](http://www.coolpython.net/method_topic/set/set-clear.html) | 删除集合中的所有元素                                         |
| [copy()](http://www.coolpython.net/method_topic/set/set-copy.html) | 拷贝一个集合                                                 |
| [difference()](http://www.coolpython.net/method_topic/set/set-difference.html) | 返回多个集合的差集                                           |
| [difference_update()](http://www.coolpython.net/method_topic/set/difference_update.html) | 从一个集合中删除另一个集合的所有元素，本质上就是删除两个集合的交集部分 |
| [discard()](http://www.coolpython.net/method_topic/set/discard.html) | 删除集合中指定的元素，元素不存在时不会引发异常               |
| [intersection()](http://www.coolpython.net/method_topic/set/intersection.html) | 返回集合的交集                                               |
| [intersection_update()](http://www.coolpython.net/method_topic/set/intersection_update.html) | 计算多个集合的交集然后用交集更新替换原集合                   |
| [isdisjoint()](http://www.coolpython.net/method_topic/set/isdisjoint.html) | 判断交集是否为空                                             |
| [issubset()](http://www.coolpython.net/method_topic/set/issubset.html) | 判断集合是否是另一集合的子集                                 |
| [issuperset()](http://www.coolpython.net/method_topic/set/issuperset.html) | 判断集合是否包含另外一个集合                                 |
| [pop()](http://www.coolpython.net/method_topic/set/pop.html) | 随机删除集合中的元素                                         |
| [remove()](http://www.coolpython.net/method_topic/set/remove.html) | 移除集合中指定元素，元素不存在时会引发异常                   |
| [symmetric_difference()](http://www.coolpython.net/method_topic/set/symmetric_difference.html) | 返回两个集合中不重复的元素集合。                             |
| [symmetric_difference_update()](http://www.coolpython.net/method_topic/set/symmetric_difference_update.html) | python集合的symmetric_difference_update方法与symmetric_difference类似，都是计算两个集合不重复的部分，区别是symmetric_difference方法返回不重复元素组成的集合，而symmetric_difference_update用这个新集合更新原集合 |
| [union()](http://www.coolpython.net/method_topic/set/union.html) | 返回两个集合的并集                                           |
| [update()](http://www.coolpython.net/method_topic/set/update.html) | 计算原集合与另一个集合的并集，并用计算得出的并集更新原集合   |



# python集合方法讲解

你必须熟练python集合的各种方法，只有这样才能在具体的实践应用中灵活的运用集合实现特定的功能。编程入门这件事情，最简单的是理解语法，最难的是记住每一种数据类型的方法并灵活使用，前者侧重理解，每个人都有各自的理解，都可以说 自己理解了，但熟练掌握这些方法并能在特定场景下使用却只有一个标准，那就是写出来，并且用得对。

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [add()](http://www.coolpython.net/method_topic/set/set-add.html) | 为集合添加新元素                                             |
| [clear()](http://www.coolpython.net/method_topic/set/set-clear.html) | 删除集合中的所有元素                                         |
| [copy()](http://www.coolpython.net/method_topic/set/set-copy.html) | 拷贝一个集合                                                 |
| [difference()](http://www.coolpython.net/method_topic/set/set-difference.html) | 返回多个集合的差集                                           |
| [difference_update()](http://www.coolpython.net/method_topic/set/difference_update.html) | 从一个集合中删除另一个集合的所有元素，本质上就是删除两个集合的交集部分 |
| [discard()](http://www.coolpython.net/method_topic/set/discard.html) | 删除集合中指定的元素，元素不存在时不会引发异常               |
| [intersection()](http://www.coolpython.net/method_topic/set/intersection.html) | 返回集合的交集                                               |
| [intersection_update()](http://www.coolpython.net/method_topic/set/intersection_update.html) | 计算多个集合的交集然后用交集更新替换原集合                   |
| [isdisjoint()](http://www.coolpython.net/method_topic/set/isdisjoint.html) | 判断交集是否为空                                             |
| [issubset()](http://www.coolpython.net/method_topic/set/issubset.html) | 判断集合是否是另一集合的子集                                 |
| [issuperset()](http://www.coolpython.net/method_topic/set/issuperset.html) | 判断集合是否包含另外一个集合                                 |
| [pop()](http://www.coolpython.net/method_topic/set/pop.html) | 随机删除集合中的元素                                         |
| [remove()](http://www.coolpython.net/method_topic/set/remove.html) | 移除集合中指定元素，元素不存在时会引发异常                   |
| [symmetric_difference()](http://www.coolpython.net/method_topic/set/symmetric_difference.html) | 返回两个集合中不重复的元素集合。                             |
| [symmetric_difference_update()](http://www.coolpython.net/method_topic/set/symmetric_difference_update.html) | python集合的symmetric_difference_update方法与symmetric_difference类似，都是计算两个集合不重复的部分，区别是symmetric_difference方法返回不重复元素组成的集合，而symmetric_difference_update用这个新集合更新原集合 |
| [union()](http://www.coolpython.net/method_topic/set/union.html) | 返回两个集合的并集                                           |
| [update()](http://www.coolpython.net/method_topic/set/update.html) | 计算原集合与另一个集合的并集，并用计算得出的并集更新原集合   |



# python集合 (set) 练习题

集合 (set) 是python 的基础数据类型，本练习题主要考察你对集合的交集，并集，差集的理解和运用，这三个操作，是集合最常见也是最为重要的操作

## 集合基本操作

集合间的运算

```python
lst1 = [1, 2, 3, 5, 6, 3, 2]
lst2 = [2, 5, 7, 9]
```

- 哪些整数既在lst1中，也在lst2中
- 哪些整数在lst1中，不在lst2中
- 两个列表一共有哪些整数

虽然题目一直在问两个列表，但用列表解答这3个题目效率很低，你应该用集合

```python
lst1 = [1, 2, 3, 5, 6, 3, 2]
lst2 = [2, 5, 7, 9]

set1 = set(lst1)
set2 = set(lst2)

# 哪些整数既在lst1中，也在lst2中
print(set1.intersection(set2))

# 哪些整数在lst1中，不在lst2中
print(set1.difference(set2))

# 两个列表一共有哪些整数
print(set1.union(set2))
```
