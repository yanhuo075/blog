---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.2.3 循环中的continue与break
order: 3
---

# python 循环中的continue 与 break

continue和break只能在for循环或者while循环中使用，continue的作用是跳过当前循环体内的剩余语句，进入下一次循环；break的作用是立即退出当前所在的循环，程序控制转移至循环的下一条语句。

## 1. continue

coninue的中文翻译是继续，在循环体里，continue的作用是跳过当前循环的剩余语句，结束本次循环，继续进行下一轮循环，下面用一个简单的例子来演示continue的功能

```python
lst = [4, 6, 1, 7, 2, 9, 3]

for item in lst:
    if item == 7:
        continue

    print(item)
```

程序输出结果为

```text
4
6
1
2
9
3
```

程序输出了列表里所有的数据，除了7以外，我们来看看，7为什么没有输出。循环过程中，item依次等于4, 6, 1, 7, 2, 9, 3，每进行一次循环，就按照顺序将一个新的值赋值给item，然后进入循环体里执行代码语句，当item等于7时，表达式 item == 7 成立，于是进入到if 下面的语句块中执行continue，continue的作用是跳过当前循环的剩余语句，结束本次循环，继续进行下一轮循环，当前循环剩余的语句只有一行，是print(item)，continue让程序跳过这行代码，因此没有输出7,下图是程序的流程图

![python中continue的作用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823231848933.jpeg)

## 2. continue 与 if

### 2.1 两种思路比较

如果你足够细心，或许已经发现一个可疑之处，实现同样的功能，是不是用if也可以啊，没错，continue的作用，可以用if来代替,使用if来重写代码

```python
lst = [4, 6, 1, 7, 2, 9, 3]

for item in lst:
    if item != 7:
        print(item)
```

既然如此，continue还有存在的必要么？很有必要，continue的核心作用是跳过当前循环剩余的代码，这对于我们思考问题极有帮助。循环过程中，满足某些条件时我们不希望代码继续执行，这个时候用continue就比用if要好很多，假设一个循环体里有两部分
![python-continue与break思路比较](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823231848847.jpeg)
代码块A执行结束后，需要进行一次条件判断以此来决定是否继续执行代码块B，当前的条件是condition, cotinue与if的思路哪个更好，你自己来体会和判断

1. continue的思路是，如果当前满足条件condition，代码块B就不执行了，进行下一轮循环，否则执行代码块B
2. if的思路是，如果当前条件满足非condition，则执行代码块B，否则不执行，进行下一轮循环

唯一的区别是，使用cotinue时，我们要找跳过代码块B的条件，而使用if 则要找不跳过代码块B的条件，我个人的体会是，找跳过代码块的条件更容易一些，就好比上学时，老师在放学前开玩笑的说：写不完作业，明天就别来上学了，几乎不会有老师说： 写完作业，明天才能来上学，前一个说法更符合我们思考方式。

### 2.2 continue让代码更容易理解

continue可以更好的控制代码缩进，同时由于其跳过剩余代码，使得代码也更容易理解和阅读，就上一篇教程《for循环》的最后一个练习题来说，寻找列表里的第2大的值，在没有学习continue之前，示例代码是

```python
lst = [4, 6, 1, 7, 2, 9, 3]

max_value = lst[0]
second_max = lst[0]

for item in lst:
    if item > second_max:
        if item >= max_value:
            second_max = max_value
            max_value = item
        else:
            second_max = item

print(second_max)
```

现在，有了continue，可以让代码看起来更加简洁

```python
lst = [4, 6, 1, 7, 2, 9, 3]

max_value = lst[0]
second_max = lst[0]

for item in lst:
    # 小于等于second_max,就没有必要继续比较了
    if item <= second_max:
        continue

    if item >= max_value:
        second_max = max_value
        max_value = item
    else:
        second_max = item

print(second_max)
```

当item <= second_max成立时，使用continue跳过剩余的代码，思路更清晰的同时，请注意代码的缩进层次最深为2层，而之前的代码最层为3层，就是这一层之差，就可以让代码更容易阅读，代码的可阅读性十分重要，等你一段代码要写100行时就深有体会。

## 3. break

满足某个条件时，我们希望能终止循环，这时，你需要使用break语句。不同于continue，break具有很强的破坏力，它的作用是直接停止当前所在的循环，程序控制转移至循环体的下一条语句。

请编写代码，从下面这个列表里找出一个大于10的数并输出

```python
lst = [1, 4, 5, 10, 2, 11, 15]
```

显然，你需要对这个列表进行遍历，并判断所遍历到的数据是否大于10，如果大于10，则输出，如果按照这个思路去做，你会发现，你输出的数据不只一个

```python
lst = [1, 4, 5, 10, 2, 11, 15]

for item in lst:
    if item > 10:
        print(item)
```

程序最终输出结果

```text
11
15
```

在输出11以后，循环并没有停止，因此后面的15也被输出了，可题目的要求是找出一个大于10的数并输出，而你却输出了两个，为了让循环终止，你需要使用break

```python
lst = [1, 4, 5, 10, 2, 11, 15]

for item in lst:
    if item > 10:
        print(item)
        break
```

程序输出结果

```text
11
```

当找到一个满足条件的数据时，先使用print输出，然后执行break，终止循环，这样就不会输出15了
