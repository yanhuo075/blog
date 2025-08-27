---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 10.6 metaclass元类
order: 5
---

# python中的元类 metaclass

在python中，类(class)本身也是一个实例对象, 它的类型则是元类, 如果没有指明, 则自定义类的类型是type. 换言之, 我们所定义的普通类都是type的实例对象, 如果一个类继承了type, 那么这个类就是元类.

### 1. 什么是元类

一个类继承了type，那么这个类就是元类

```python
class A(type):
    pass
```

A就是一个元类，元类能用来做什么呢，应该说，绝大多数时候都用不上元类，如果你想使用元类，请确保你非常理解它

### 2. 元类的__new__方法

在定义一个类时，指定metaclass，就意味着这个类将有所指定的metaclass来创建

```python
class MyMeta(type):
    def __new__(cls, *args, **kwargs):
        _class = super().__new__(cls, *args, **kwargs)
        print(_class.__name__)
        return _class


class Animal(metaclass=MyMeta):
    def __init__(self, name):
        self.name = name
```

类MyMeta是元类，在定义Animal这个类时，我指定了它的元类是MyMeta，因此，类Animal将由MyMeta的__new__方法来创建，换一个角度来描述，类Animal是类MyMeta的实例对象。在MyMeta的__new__方法中，我使用print语句输出了__class的__name__属性，理论分析告诉我们，这个值应该是Animal， 实际结果也确实是如此。

元类是用来创建普通类（自定义类）的，我们可以利用元类对普通类进行一些限制和要求，比如，我们可以要求所有继承Animal的类必须拥有run方法

```python
from inspect import isfunction

class MyMeta(type):
    def __new__(cls, *args, **kwargs):
        _class = super().__new__(cls, *args, **kwargs)
        if _class.__name__ != 'Animal':
            if not hasattr(_class, 'run') or not isfunction(getattr(_class, 'run')):
                raise Exception('类{name}没有实现run方法'.format(name=_class.__name__))
        return _class


class Animal(metaclass=MyMeta):
    def __init__(self, name):
        self.name = name


class Cat(Animal):
    def __init__(self, name):
        super().__init__(name)

cat = Cat('加菲猫')
```

类Cat继承了Animal，那么它的元类也是MyMeta，在MyMeta的__new__方法里将创建出类Cat，创建以后会检查类Cat是否有run属性且该属性是一个函数，如果不满足条件则抛出异常。如果类Cat实现了run方法，那么上述代码将正常执行

```python
class Cat(Animal):
    def __init__(self, name):
        super().__init__(name)

    def run(self):
        print('run')

cat = Cat('加菲猫')
cat.run()
```

我们务必想清楚一点，尽管我们在脚本里使用class定义了类Cat, 但并不是真正的创建了类Cat,我们所写的代码仅仅是一个定义，创建的过程使用元类MyMeta来完成的。

### 3. 元类的__init__方法

元类的__new__负责构建普通类，__init__负责对这个普通类进行初始化

```python
from inspect import isfunction

class MyMeta(type):
    def __new__(cls, *args, **kwargs):
        _class = super().__new__(cls, *args, **kwargs)
        if _class.__name__ != 'Animal':
            if not hasattr(_class, 'run') or not isfunction(getattr(_class, 'run')):
                raise Exception('类{name}没有实现run方法'.format(name=_class.__name__))
        return _class

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.home = 'earth'


class Animal(metaclass=MyMeta):
    def __init__(self, name):
        self.name = name


class Cat(Animal):
    def __init__(self, name):
        super().__init__(name)

    def run(self):
        print('run')

print(Animal.home)
print(Cat.home)
```

在元类的__init__方法里，self参数就是我们所创建的类，Animal和Cat， 我们为他们增加了类属性home， 重载__init__方法，可以更加优雅的实现单例模式

```python
class Singleton(type):
    def __init__(self, *args, **kwargs):
        self.__instance = None
        super().__init__(*args, **kwargs)

    def __call__(self, *args, **kwargs):
        if self.__instance is None:
            self.__instance = super().__call__(*args, **kwargs)
            return self.__instance
        else:
            return self.__instance

class FileLock(metaclass=Singleton):
    pass

file_lock_1 = FileLock()
file_lock_2 = FileLock()
print(file_lock_1 is file_lock_2)
```

### 4. 元类的__call__方法

```python
class MyMeta(type):
    def __call__(self, *args, **kwargs):
        raise TypeError('不能创建实例')


class FileTool(metaclass=MyMeta):
    @staticmethod
    def iter_folder(path):
        print('遍历文件夹')

ft = FileTool()
```

上面的代码执行会报错

```text
Traceback (most recent call last):
  File "/Users/kwsy/kwsy/coolpython/demo.py", line 13, in <module>
    ft = FileTool()
  File "/Users/kwsy/kwsy/coolpython/demo.py", line 5, in __call__
    raise TypeError('不能创建实例')
TypeError: 不能创建实例
```

类FileTool是元类MyMeta的一个示例，那么当执行FileTool()时，不正是在调用元类MyMeta的__call__方法么，而MyMeta的__call__方法偏偏抛出一个类型异常，这就导致FileTool不能被实例化，我们只能使用它的静态方法。

重载元类的__call__方法和类cat的__del__方法可以让我们控制类的实例化过程，我们可以控制一个类的实例数量

```python
class MyMeta(type):
    def __init__(self, *args, **kwargs):
        self.instance_count = 0
        super().__init__(*args, **kwargs)

    def __call__(self, *args, **kwargs):
        if self.instance_count < 3:
            self.instance_count += 1
            return type.__call__(self, *args, **kwargs)
        else:
            raise Exception("类{name}的实例总数超出限制".format(name=self.__name__))

    def __del__(self):
        self.instance_count -= 1

class Cat(metaclass=MyMeta):
    def __init__(self, name):
        self.name = name

    def __del__(self):
        Cat.instance_count -= 1


c1 = Cat('c1')
c2 = Cat('c2')
c3 = Cat('c3')
c4 = Cat('c4')
```

上面的代码中，当创c4的时候会抛出异常，因为实例的数量已经达到上限，想要创建c4，必须销毁一个之前创建的对象实例

```python
c1 = Cat('c1')
c2 = Cat('c2')
c3 = Cat('c3')
del c1
c4 = Cat('c4')
```

销毁c1时，类属性instance_count执行了减一操作，因此可以创建出c4。

### 5.小结

以上示例代码，不保证有工程实践意义，纯粹是为了讲解元类的功能作用而认为制造出来的，坦率的讲，在实际工作中，几乎用不到元类，但我仍然秉持一个观点：面试造火箭，工作拧螺丝的意义在于，能造火箭的人必然牛逼，你可以放心的把拧螺丝的工作交给他，至于是否浪费资源，如果你不会造火箭，那么请慎言，这还不是你这个层次所能讨论的问题。
