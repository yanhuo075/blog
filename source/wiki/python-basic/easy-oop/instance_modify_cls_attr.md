---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.16 为什么实例不能修改类属性
order: 15
---

# 轻松学会python面向对象第15篇---为什么实例不能修改类属性

## 1. 实例不能修改类属性

在本系列第13篇文章中有一个结论，**“你可以通过类直接访问类的属性，也可以通过对象实例访问类的属性，但是你不能通过对象实例修改类的属性”**，下面的代码，向你证明这一点

```python
class NovelBook():
    book_type = "小说"

    def __init__(self, _name, _author, _price):
        self.name = _name  # 书名
        self.author = _author  # 作者
        self.price = _price  # 价格

book = NovelBook("西游记", "吴承恩", 99)
print(book.book_type)           # 小说
print(NovelBook.book_type)      # 小说

book.book_type = "魔幻小说"
print(book.book_type)           # 魔幻小说
print(NovelBook.book_type)      # 小说
```

输出结果为

```text
小说
小说
魔幻小说
小说
```

这真是一个让初学者难以理解的现象，但只要研究清楚python的对象如何存储属性，问题就迎刃而解。

## 2. 实例的 __dict__

python里一切皆对象，每个对象都有一个__dict__属性，它是一个字典，对象的属性都存储在这里

```python
book = NovelBook("西游记", "吴承恩", 99)
print(book.__dict__)
```

输出结果

```text
{'name': '西游记', 'author': '吴承恩', 'price': 99}
```

__dict__里存储的，刚好是实例对象的3个属性，使用点号运算时，比如book.name，根据python寻找属性的机制，会先到对象的__dict__里查看是否有name属性，这里显然可以找到的，因此book.name 的值是“西游记”。

注意，book.__dict__ 并没有book_type 属性，但是print(book.book_type) 仍然输出了“小说”，这是因为当在实例对象里的__dict__找不到时，就会去实例所属的类中寻找，在NovelBook.__dict__ 中，恰好可以找到book_type属性，如果NovelBook还是找不到，会根据类的继承关系继续向上寻找。

在执行print(book.book_type) 这行代码时，book.book_type是从NovelBook.__dict__取到的值，最终输出为“小说”

当下面的代码执行时

```python
book.book_type = "魔幻小说"
```

并没有对NovelBook.__dict__ 进行任何修改，而是修改了book.__dict__， 在book.__dict__ 中新增了一个key-value对，key是book_type， value是魔幻小说，这也就解释了最后两次print输出内容为何会不一样。

下面的代码，可以证实上面所讲述内容

```python
book.book_type = "魔幻小说"
print(book.__dict__)
```

输出内容为

```text
{'name': '西游记', 'author': '吴承恩', 'price': 99, 'book_type': '魔幻小说'}
```

book.__dict__ 中可以找到book_type属性，就不会去NovelBook.__dict__寻找，对实例属性进行赋值，只能修改实例的__dict__ 。

## 3. 类的 __dict__

写一个有方法的类，输出类的__dict__

```python
class NovelBook():
    book_type = "小说"

    def __init__(self, _name, _author, _price):
        self.name = _name  # 书名
        self.author = _author  # 作者
        self.price = _price  # 价格

    def discount(self, ratio):
        """
        打折
        :param ratio:
        :return:
        """
        self.price *= ratio

print(NovelBook.__dict__)
```

输出内容

```text
{'__module__': '__main__', 'book_type': '小说', 
'__init__': <function NovelBook.__init__ at 0x000001C307207BF8>, 
'discount': <function NovelBook.discount at 0x000001C307207C80>, 
'__dict__': <attribute '__dict__' of 'NovelBook' objects>, 
'__weakref__': <attribute '__weakref__' of 'NovelBook' objects>, '__doc__': None}
```

类的__dict__内容要多一些，你可以找到book_type， 也可以找到discount方法，还记得本系列第10篇的文章么，属性属于对象，方法属于类，更准确严谨的表述是：实例属性属于实例对象，方法属于类
