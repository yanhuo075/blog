---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.8 生成器
order: 7
---

# python 生成器

首先要明确一点，生成器是一种迭代器，生成器拥有next方法并且行为和迭代器相同，都可以用于for循环。
生成器对延迟操作提供了支持，这使得python可以在需要的时候才生成结果，而不是立即产生结果。生成器更加高效的利用内存, 当需要产生大量数据时, 使用生成器返回一个惰性的迭代器, 而不是在内存中保存他们。

## 1. 提供生成器的两种方式

python有两种方式提供生成器

- 生成器函数
- 生成器表达式

### 1.1 生成器函数

如果一个函数内部出现了yield这个关键字，那么该函数就是一个生成器函数，调用生成器函数将得到一个生成器，下面的示例将演示一个生成器函数的定义和使用：

```python
# coding=utf-8

# 定义一个生成器函数
def my_generator(n):
    index = 0
    while index < n:
        yield index
        index += 1

generate = my_generator(5)
print(type(generate))
for i in generate:
    print(i)
```

程序输出结果为：

```text
<type 'generator'>
0
1
2
3
4
```

理解上面的代码，要注意以下几点：

- 当执行generate = my_generator(5) 这行代码时，my_generator函数里的代码没有被执行，因为函数里有yield关键，函数已经变成了一个生成器函数，生成器函数在被调用时会返回一个生成器，此时，函数里的代码不会被执行。
- for循环的过程就是执行next方法的过程，当生成器的next方法被调用时，函数内部的代码才会执行，执行过程中遇到yield关键字，就会暂停（挂起），并将yield的参数做此次next方法的返回值。
- 随着next方法的执行，函数内的while循环终究会有停止的时候，这个时候函数结束了，抛出StopIteration异常。
- 生成器每次遇到yield暂停执行时，会把函数内的所有变量封存在生成器中，当下一次next方法被执行时，恢复这些变量。
- 生成器函数内部，不允许使用return语句返回任何值，因为生成器函数已经默认返回一个生成器了，但是你可以只写一个return，后面不带任何值。

### 1.2 生成器表达式

下面展示一个生成器表达式

```python
squares = (x**2 for x in range(5))
print type(squares)
```

程序输出结果为:

```text
<type 'generator'>
```

## 2. 生成器有哪些优点

本章开头处便讲到，生成器为延迟计算提供了支持，只在需要的时候再进行计算，这样一来，就能够减少对内存的使用。接下来将用一个例子来讲解生成器的好处。
已知有两个字典，内容如下：

```python
dict_1 = {
    'key1': [1, 2, 3],
    'key2': [2, 3, 4],
    'key3': [3, 4, 5]
}

dict_2 = {
    'key1': ['a', 'b', 'c'],
    'key2': ['b', 'c', 'd'],
    'key3': ['c', 'd', 'e']
}
```

可以看到两个字典有相同的key，且value都是列表，由于篇幅有限，我让每个字典都只有3个key，但其实它可以拥有更多的key，value即列表里也可以拥有更多的元素，这样的假设是希望你能明白，更多的元素要占用更多的内存。

现在，我想知道两个字典合并后每个key对应的value是什么内容，并且对它进行输出，我先来实现一个耗费内存的版本，示例1 代码如下:

```python
dict_1 = {
    'key1': [1, 2, 3],
    'key2': [2, 3, 4],
    'key3': [3, 4, 5]
}

dict_2 = {
    'key1': ['a', 'b', 'c'],
    'key2': ['b', 'c', 'd'],
    'key3': ['c', 'd', 'e']
}

def merge_dict(dict_1, dict_2):
    merge_dit = {}
    merge_dit.update(dict_1)
    for k, v in merge_dit.items():
        merge_dit[k].extend(dict_2[k])

    return merge_dit

merge_dit = merge_dict(dict_1, dict_2)
for k, v in merge_dit.items():
    print k, v
```

程序输出结果为:

```text
key3 [3, 4, 5, 'c', 'd', 'e']
key2 [2, 3, 4, 'b', 'c', 'd']
key1 [1, 2, 3, 'a', 'b', 'c']
```

之所以说这个版本的实现是耗费内存的，是因为merge_dict函数创建了一个新的字典用来保存合并后的结果，理论上内存的使用增加了一倍，接下来看不节省内存的版本，示例2代码如下：

```python
dict_1 = {
    'key1': [1, 2, 3],
    'key2': [2, 3, 4],
    'key3': [3, 4, 5]
}

dict_2 = {
    'key1': ['a', 'b', 'c'],
    'key2': ['b', 'c', 'd'],
    'key3': ['c', 'd', 'e']
}

for key, value in dict_1.items():
    value.extend(dict_2[key])
    print(key, value)
```

这个版本的实现的确在内存使用上比示例1好很多，但是却有一些不易被发现的缺陷

1. 我现在的要求是输出合并后的key和value，但是如果出现了新的要求，计算value中的数字之和，那么就需要写一段新的代码，例如下面这样

   ```python
   for key, value in dict_1.items():
   value.extend(dict_2[key])
   sum = 0
   for item in value:
       if isinstance(item, int):
           sum += item
   print(key, sum)
   ```

2. 对合并后结果的使用逻辑必须和合并的逻辑放在一起，就是说在for循环内部，既要完成字典合并又要完成结果的输出或者数字之和的计算

3. 其他函数或者模块无法直接使用合并后的结果

对于第3条，如果采用示例1中的办法，其他函数可以直接使用新创建的字典，但这样耗费内存，采用示例2中的办法，虽然不耗费内存，但是必须直接对两个字典进行操作，而且还要关心如何去合并，那么有没有什么好的办法，既能不耗费内存，又能让代码工整简洁，利于维护呢，请看示例3

```python
# coding=utf-8

dict_1 = {
    'key1': [1, 2, 3],
    'key2': [2, 3, 4],
    'key3': [3, 4, 5]
}

dict_2 = {
    'key1': ['a', 'b', 'c'],
    'key2': ['b', 'c', 'd'],
    'key3': ['c', 'd', 'e']
}


def gen_dict(dict_1, dict_2):
    """
    gen_dict 封装了合并两个字典的细节,而且不耗费内存
    :param dict_1:
    :param dict_2:
    :return:
    """
    for key, value in dict_1.items():
        value.extend(dict_2[key])
        yield key, value


# generate 是一个生成器,你在使用时根本不需要关心两个字典是如何合并的
generate_1 = gen_dict(dict_1, dict_2)
for key, value in generate_1:
    print(key, value)

# 计算合并后每个key所对应的value内部元素之和
generate_2 = gen_dict(dict_1, dict_2)
for key, value in generate_2:
    sum = 0
    for item in value:
        if isinstance(item, int):
            sum += item
    print(key, sum)
```

程序输出结果为:

```python
('key3', [3, 4, 5, 'c', 'd', 'e'])
('key2', [2, 3, 4, 'b', 'c', 'd'])
('key1', [1, 2, 3, 'a', 'b', 'c'])
('key3', 12)
('key2', 9)
('key1', 6)
```

看到示例3的代码，是不是感到清爽许多，函数gen_dict返回一个生成器，该函数实现了对两个字典的合并而且不耗费内存。

在使用生成器generate_2时，你根本不需要关心两个字典是如何合并的，你只需要关心如何计算数字之和，合并字典的逻辑与合并后结果的处理逻辑是完全可以分开的，不必像示例2那样纠缠在一起。

## 3. 使用生成器需要避免的坑

生成器虽然好用，但如果使用不当就会引发问题。

```python
def generate_num(n):
    for i in range(n):
        yield n

iter_num = generate_num(10)
for num in iter_num:
    print(num)

for num in iter_num:        # 第二次执行for循环,没有任何效果
    print(num)
```

for循环的过程就是执行next方法的过程，经历过一次for循环以后，迭代器已经到了末尾，在for循环的内部，已经抛出StopIteration异常，for循环在捕捉到这个异常后停止遍历，因此第二次for循环时不会产生任何效果。
