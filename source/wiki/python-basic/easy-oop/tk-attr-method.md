---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.14 填坑，关于属性和方法（1）
order: 13
---

# 轻松学会python面向对象第13篇---填坑，关于属性和方法（1）

![python面向对象](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823235756244.jpeg)

## 1. 为何要填坑

本系列文章遵守一个简单的原则---不容易解释的概念，暂时不解释，先从代码形式入手，等大家熟悉这些概念和名词后，再回来填坑，以确保做到先易后难，循序渐进。

因此呢，前面的一些概念，讲解的并不准确。只是在那个时候，不准确的概念更有利于你理解，但随着学习的深入，又必须澄清这些概念，否则，就走错了路。

从这一篇开始，我后续会逐步将前面挖的坑填上。

## 2. 属性和方法的坑

本系列的第2篇文章，主题便是属性和方法，文章里对于属性和方法的介绍非常的粗糙，甚至是不准确的。

对于属性，有**类属性**和**实例属性**之分。

对于方法，有**类方法**，**实例方法**，**静态方法**之分

本篇重点讲解类属性和实例属性

## 3. 类属性和实例属性

### 3.1 实例属性

```python
class Book():
    def __init__(self, _name, _author, _price):
        self.name = _name               # 书名
        self.author = _author           # 作者
        self.price = _price             # 价格
```

name，author, price 都是实例属性，在本系列第10篇文章里已经阐明，属性属于对象，对象和实例在本系列文章中表示同一个事物。实例属性，在对象，实例被创建出来以后才存在的。上面的代码，只是定义了一个类，没有创建任何实例对象，这些属性压根就不存在，实例属性，依附于实例而存在。

### 3.2 类属性

类属性又是什么东西呢？我定义一个新的类

```python
class NovelBook():
    book_type = "小说"

    def __init__(self, _name, _author, _price):
        self.name = _name  # 书名
        self.author = _author  # 作者
        self.price = _price  # 价格
```

book_type就是类属性，它是NovelBook的类属性，你可以这样来使用它

```python
print(NovelBook.book_type)      # "小说

novel = NovelBook("西游记", '吴承恩', '100')
print(novel.book_type)          # 小说

novel.book_type = '言情小说'
print(novel.book_type)          # 言情小说

print(NovelBook.book_type)      # 小说
NovelBook.book_type = "魔幻小说"
print(NovelBook.book_type)      # 魔幻小说
```

你可以通过类直接访问类的属性，也可以通过对象实例访问类的属性，但是你不能通过对象实例修改类的属性，因为类的属性是属于类的，这一点从名称上就可以看得出来，也因为如此，你可以通过类来修改类属性。至于为什么不能通过实例对象修改类属性，我后面来填这个坑。

### 3.3 实例属性只能在类里定义么？

3.1 小节中的内容，可能会让你产生一个误解，以为只有定义在__init__方法里的self.XXX才是实例属性，实际情况不是这样，我下面举两个例子

```python
class NovelBook():
    book_type = "小说"

    def __init__(self, _name, _author, _price):
        self.name = _name  # 书名
        self.author = _author  # 作者
        self.price = _price  # 价格

    def set_page_count(self, count):
        self.page_count = count             # page_count也是实例属性

novel = NovelBook("西游记", '吴承恩', '100')
novel.set_page_count(290)
print(novel.page_count)                     # 290

setattr(novel, 'score', 100)
print(novel.score)                          # 100
```

page_count不是在__init__方法里定义的，但也是实例属性，区别在于，实例对象被创建好了以后，就会自动调用__init__方法进行初始化，你可以理解为实例被创建出来以后，name, author， price就已经存在了，而此时page_count 还不存在，要等到novel对象调用set_page_count方法时，page_count才会被创建。

我用setattr函数，强行为示例对象novel增加了一个socre属性，所有才能够用novel.score这种写法获取到score属性。

实例属性有这样的表现，你应当已经猜到，类属性也可以这样来操作。

### 3.4 类属性有什么作用呢？

实例属性的作用比较容易理解，类属性则困难一些，你有这样的疑惑倒也正常。这里又要给你挖一个坑，python语言里，一切皆对象，你用class 定义出来的类，比如NovelBook，其实，也是一个对象，它是type这个类创建出来的对象，NovelBook也有一些需要被描述的属性，比如book_type，描述书的类型。

关于类属性，对它的思考，不必过于执着，暂且从概念上理解到这里。它存在的合理性和必要性，要等到你对编程有更深入的理解和实践之后才能够有所体会，眼下，还是从形式上记住它。

如果你总是尝试用理解代替记忆，那么你就会执着于寻找具体的应用实例来解答心中的疑惑，通过具体应用场景来理解它。可是理解具体应用实例需要你自身有坚实的基础和丰富的知识，这本身又是你目前所欠缺的，这很矛盾。

越是学习高阶的编程知识，越是无法从现实生活中寻找到可以类比的事物和概念，即便勉强找到了，也是词不达意。进入到高阶知识学习阶段，你必须融入到编程的世界里来，用计算机的语言来描述你的问题，用编程语言的语法和概念，逐步的构建出一个虚拟的世界。

这个世界里，天马行空，一切超乎寻常的想象，只要符合逻辑，都可以合理的存在。
