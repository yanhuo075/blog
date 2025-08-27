---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.4 一切皆对象
order: 3
---

# python一切皆对象

**一切皆对象**是pyhton这门血统纯正的编程语言的重要概念，对象在其他编程语言里往往代指类创建出来的实例，而python不同，python中的一切都是对象。 你所熟知的函数是对象，int是对象，列表是对象，甚至你自定义的类本身也是对象，自定义类是type创建出来的对象，而type则是type自身创建出来的对象。

## 1. 关于python的类

### 1.1 先了解类

与对象相伴的一个概念叫类，类就好比模板，对象都是根据这个模板刻画衍生出来的。

比如下面这行简单的代码

```python
a = int('3')
print(a, type(a))
```

程序最终输出结果是

```text
3 <class 'int'>
```

虽然各类教程，包括python官方文档里将int()称之为内置函数，但你查看源码可以发现，它其实是类

```python
class int(object):
    """
    int(x=0) -> integer
    int(x, base=10) -> integer
    """
```

你平时在程序里的各种整型数据，都是这个类生成的，这个类是python提供给我们的，而且我们不用关心对象的创建过程。

### 1.2 自己定义的类

我们可以自己创建类，然后用这个类去创建对象

```python
class MyClass:
    def print(self):
        print("自己创建的类")


mc = MyClass()
print(mc, type(mc))
```

程序输出结果

```text
<__main__.MyClass object at 0x101f59828> <class '__main__.MyClass'>
```

在面向对象的概念里，mc就是一个对象，MyClass是类，一个类可以生成很多个对象。

### 1.3 类也是对象

前面介绍了python提供的类和我们自己定义的类，你已经知道类可以生成对象，现在突然告诉你，类也是对象，你肯定感觉有丈二和尚摸不着头脑。

咱们打个比方，mc是儿子，MyClass就是爸爸，现在告诉你，MyClass也是儿子，你还糊涂么?MyClass也是儿子，那它的爸爸是谁呢？

咱们可以用type函数寻找它的爸爸

```python
class MyClass:
    def print(self):
        print("自己创建的类")


mc = MyClass()

father = type(mc)
print(father)

grandfather = type(father)
print(grandfather)

print(type(grandfather))
```

程序输出结果

```text
<class '__main__.MyClass'>
<class 'type'>
<class 'type'>
```

从输出结果，我们可以推导出如下结论

1. mc的爸爸是MyClass
2. MyClass的爸爸是type
3. type的爸爸是它自己

惊不惊喜，意不意外，type也是一个类

```python
class type(object):
    """
    type(object_or_name, bases, dict)
    type(object) -> the object's type
    type(name, bases, dict) -> a new type
    """
```

在python中，不管是什么东西，只要你用type去追踪寻找它的爸爸，最终的线索都指向type,一个type不够用，就再加一个type，不行就再加

```python
print(type(3))
print(type(type(3)))

def test():
    pass

print(type(test))
print(type(type(test)))

class T:
    pass

print(type(T))
```

程序输出结果

```text
<class 'int'>
<class 'type'>
<class 'function'>
<class 'type'>
<class 'type'>
```

int类型是对象，函数是对象，我们自己定义的类也是对象，他们的类型都是type，type是个万能的类啊，就连type自己都是对象,所以才有python一切皆对象。

### 1.4 梳理一下思路

普通对象，例如我们自己创建的mc,mc的类型是类，而类的类型就叫做元类（metaclass）， 普通的类，可以生成对象，对象也可以称之为实例，元类是类，当然也可以生成对象，元类生成的对象就是我们创建的普通类。

## 2. 类的创建

### 2.1 直接定义一个类

想要创建一个类，你可以直接定义它

```python
class MyTest:
    def __init__(self, name):
        self.name = name


mt = MyTest('mytest')
print(mt.name)
```

### 2.2 使用type创建

既然，MyTest的类型是type, MyTest是type的一个实例，那么我们就可以通过type这个类来生成它，用type生成一个类并不难，其构造函数如下

```text
type(name, bases, dict):
```

1. name 是类的名字
2. bases 是继承的类
3. dict 存放类的属性和方法
   现在，用type来创建一个类，和上面定义的MyTest一模一样的类

```python
def init(self, name):
    self.name = name


mytest_class = type('MyTest', (), {'__init__': init})
mt = mytest_class('mytest')
print(mt.name)
```

程序输出结果

```text
mytest
```

看明白了吧，type是类MyTest 的元类

### 2.3 类的创建过程

2.2 小节中，我直接用type去创建类，而在2.1小节中，我只是直接定义了一个类，似乎和type没有关系，其实不然。

当程序启动后，python解释器会解释执行这些代码，类的创建包含下列过程

1. 遇到class 关键字时，要解析对类的定义，如果没有指定metaclass，那么metaclass 就默认使用type
2. 准备namespace，如果元类实现了 __prepare__ 函数，则会调用它来得到默认的 namespace, 如果没有实现，创建一个空的有序字典作为namespace
3. 调用exec 来执行类的body,类的属性和方法最终都会保存到namespace中
4. 调用类的构造函数来创建类

知道了类的创建过程，知道了元类控制着类的创建，也知道了默认情况下元类是type且这个元类是可以设置的，那么如果你想对类的创建过程进行控制，进行有目的的定制操作，就可以通过元类来进行。
