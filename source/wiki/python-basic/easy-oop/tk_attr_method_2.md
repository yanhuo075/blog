---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.15 填坑，关于属性和方法（2）
order: 14
---

# 轻松学会python面向对象第14篇---填坑，关于属性和方法（2）


上一篇文章，我填了属性的坑，这一篇继续来填方法的坑。此前在介绍类时，我一直强调类是一个约定，属性是对数据的约定，方法是对操作数据行为的约定。python的面向对象有3种方法，分别是实例方法，类方法，静态方法，接下来，我将从多个角度来介绍这三种方法

## 1. 定义方法不同

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

    @classmethod
    def out_type(cls):
        """
        输出类型
        :return:
        """
        print(cls.book_type)

    @staticmethod
    def off_the_shelf():
        """
        下架
        :return:
        """
        print("商品下架")
```

从定义形式上对他们进行区分

1. 没有被classmethod和staticmethod装饰的方法就是实例方法，例如discount
2. 被classmethod装饰器修饰的方法就是类方法，例如out_type
3. 被staticmethod装饰的方法就是静态方法，例如off_the_shelf

实例方法第一个参数通常被定义为self，类方法第一个参数通常被定义为cls，但这不是必须，仅仅是一个约定俗成的规则，因此不能作为区分它们的特征。self表示实例对象，谁调用，self就是谁；cls表示类，也遵守谁调用cls就是谁的原则，但略有不同，本文3.1小节会解释这种不同。

__init__方法的第一个参数也是self，且也没有被classmethod和staticmethod装饰，但在类型划分上，并没有将其归为实例方法，而是为这一类方法起了一个特定的名称---魔法方法，关于魔法方法，后面会有专题文章进行介绍，好吧，又给自己挖了一个坑。

## 2. 划分三种方法的意义是什么？

为什么要弄出来三种不同类型的方法呢，怎样理解他们存在的必要性呢？想要回答这个问题，先要确立一个宗旨和原则，我们所采用的种种技术，一个最核心的动机便是保护数据。
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823235839260.jpeg)

生活中，我们保护文物，保护环境，保护小动物，在编程的世界里，我们要保护数据，保护数据不被随意修改，或者是不小心的修改。

在实例方法里，你不能修改类的属性，在类方法里，你不能修改实例的属性，在静态方法里，你既不能修改实例的属性，也不能修改类的属性。通过这种限制，最大程度上保护了数据。

## 3. 如何使用他们

三种不同的方法，如何调用呢？调用的主体有两个，类，或者实例对象，下面分别介绍他们做为调用主体时的调用方法

### 3.1 实例作为调用主体

当实例作为调用主体时，三种方法都可以调用

```python
book = NovelBook('西游记', '吴承恩', 99)
book.discount(0.9)      # 调用实例方法
book.out_type()         # 调用类方法
book.off_the_shelf()    # 调用静态方法
```

实例对象调用实例方法，乃是天经地义之事，无需赘言，调用静态方法，也是畅通无阻，唯独调用类方法时，你可能会感到一丝丝的疑惑，当实例对象调用类方法时，参数cls究竟是类还是实例对象，下面我们通过实验来揭开真相。

我为NovelBook类增加一个可以修改类属性的方法

```python
    @classmethod
    def set_type(cls, _type):
        print(cls)
        cls.book_type = _type
```

我在set_type方法里输出cls的值，以此来确定当实例对象调用它时，参数cls到底是谁，执行下面的代码

```python
book = NovelBook('西游记', '吴承恩', 99)
book.set_type('魔幻小说')
print(NovelBook.book_type)      # 魔幻小说
```

输出结果是

```text
<class '__main__.NovelBook'>
魔幻小说
```

结合代码和输出结果进行分析，我们可以得出一个结论：**尽管是实例对象调用了类方法，但方法里的cls仍然是类，并没有严格遵守谁调用cls就是谁的原则，当实例对象调用时，cls是这个实例对象的类型，谁创建了实例，cls就是谁。**

### 3.2 类作为调用主体

当类作为调用主体时，三种方法都可以调用

```python
book = NovelBook('西游记', '吴承恩', 99)

NovelBook.set_type('魔幻小说')       # 调用类方法
print(NovelBook.book_type)
NovelBook.discount(book, 0.9)       # 调用实例方法
NovelBook.off_the_shelf()           # 调用静态方法
```

类，调用类方法，乃是天经地义之事，调用静态方法，也无阻碍，只是在调用实例方法时，要做特殊处理。当实例对象调用实例方法时，遵循谁调用，self就是谁的原则，self就是那个调用实例方法的实例对象，当调用主体是类时，由于没有实例对象的存在，因此必须为self指定实例对象。

可不可以不指定实例对象呢？答案是可以不指定，python是解释型语言，没有预编译这个过程，因此你可以将book替换成None，改成

```text
NovelBook.discount(None, 0.9)
```

这样写，还是可以调用实例方法discount，但是，程序最终会报错，原因在于在方法discount内，需要修改price属性，可是self此刻并非是实例对象，而是None，可None没有price属性。进一步思考，如果在实例方法里不访问任何实例属性，那么当类作为调用主体时，self参数就可以随意传参了呢？有想法，用代码验证，我增加一个新的实例方法

```python
    def test_none(self, ratio):
        print(ratio)
```

在实例方法test_none里，没有访问任何实例属性，接下来，看我如何用类来调用这个实例方法

```python
NovelBook.test_none(None, 0.9)              # 0.9
NovelBook.test_none(1, 0.9)                 # 0.9
NovelBook.test_none('随意传参', 0.9)        # 0.9
```

这样做很有趣，但没有人会这样做，如果实例方法里不访问任何实例属性，那么就应当将其定义为静态方法。静态方法里既不访问实例属性，也不会访问类属性。你没有必要写一个不访问实例属性的实例方法，也没有必要写一个不访问类属性的类方法，当然，如果你不怕被同行耻笑，你可以这样做，技术上没有任何限制。
