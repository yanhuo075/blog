---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.11 方法属于类，属性属于对象
order: 10
---

# 轻松学会python面向对象第10篇---方法属于类，属性属于对象

方法属于类，属性属于对象，这并不是一个完全正确的论断，然而，我还是建议你记住它，理解它，因为以此为起点，可以更好的理解类与对象之间的关系。

## 1. 方法属于类

如何理解这句话呢，方法的存在，不依赖于对象，而是依赖于类；属性的存在，不依赖于类，而是依赖于对象。

```python
class Dog():
    def __init__(self, name):
        self.name = name

    def biting(self):
        print(f'{self.name}在咬人')

    def biting_ex(self):
        print('咬人')
```

这是一段简单的代码，定义了一个Dog类，先来理解方法属于类，我增加一行代码

```python
Dog.biting_ex(None)
```

这行代码可以正确运行，我传入参数是None,你可以传入任意参数，都不是问题。我没有创建任何实例对象，但是我可以调用实例方法。

## 2. 属性属于对象

biting也可以这样调用么，我们来试一下

```python
Dog.biting(None)
```

结果报错了

```text
AttributeError: 'NoneType' object has no attribute 'name'
```

None没有name属性，属性依附于对象而存在，没有创建对象，也就没有name属性，修改一下代码

```python
dog = Dog('小黑')
Dog.biting(dog)
```

这样就没问题了，Dog.biting(dog) 等价于dog.biting()。我创建了一个对象，name属性也就存在了，更抽象的说法是在内存中创建了一个dog对象，dog对象里有一个name属性，没有创建对象dog之前，name属性自然也就不存在。

## 3. dog.biting与Dog.biting是同一个东西么

接下来要讲解的，属于比较深入的内容，如果感到吃力，可以放弃

我们通过查看他们的id来判断他们是否是同一个东西。

```python
dog = Dog('小黑')
print(id(dog.biting))
print(id(Dog.biting))
```

输出结果是

```text
1747144415176
1747147189376
```

竟然不是同一个东西，那对象dog的biting究竟从哪里来的呢？前面不是刚说过方法是属于类的么，那么按理说，对象所使用的方法应该就是类的方法。

这里的确是一个疑点，所以，我们要深入挖掘。

### 3.1 多个对象的biting一样么

我再创建出一个对象，看看多个对象的biting是不是同一个东西

```python
dog = Dog('小黑')
dog1 = Dog('嘿嘿')
print(id(dog.biting))       # 1610069776328 
print(id(dog1.biting))      # 1610069776328
```

不论创建多少个对象，他们的biting方法的内存地址都是相同的，那么他们从哪里来的呢，跟Dog.biting到底有没有关系呢，答案是有关系

## 3.2 bound method

```python
dog = Dog('小黑')
print(dog.biting)    
```

注意看输出结果

```text
<bound method Dog.biting of <__main__.Dog object at 0x00000245E1C3BD30>>
```

dog.biting 是Dog.biting函数的绑定方法，虽然不是很容易理解，但是可以明确，他们之间是存在关系的。

```python
print(dog.__dict__)
print(Dog.__dict__)
```

我输出对象dog和类Dog的__dict__， 可以看到，dog的属性只有name，而类Dog的属性有很多，包括了biting。这再次印证我们的观点，方法属于类，属性属于对象。

结合bound method 这个描述，我们推断biting是一个描述器，那么dog.biting 就等价于 Dog.biting.__get__(dog, Dog))，实验来证明一切

```python
dog = Dog('小黑')
print(id(dog.biting))                       # 1230893225928
print(id(Dog.biting.__get__(dog, Dog)))     # 1230893225928
```

两次输出的内存地址是一样的，真想大白于天下。

### 3.3 事情的真相

对象dog并没有biting属性，那么在执行dog.biting时，对象的__dict__找不到，就要去其类的__dict__中寻找。

而Dog类中找到的biting是描述器，根据描述器协议

```text
self.descr = descr.__get__(self, obj, type=None) --> value
```

因此dog.biting 就等价于 biting.__get__(dog, Dog))，返回的是Dog.biting绑定后的方法，

如果事情果真如此，那么我可以将biting方法替换掉

```python
dog = Dog('小黑')

def too_simple():
    print("惊不惊喜")

dog.__dict__['biting'] = too_simple
dog.biting()        # 惊不惊喜
```

惊不惊喜，意不意外！能在dog.__dict__找到biting，就不会从类里寻找了。

### 3.5 继续思考

为什么会是这样呢？嗯，这样很合理。

Dog类只有一个，如果方法属于对象，那岂不是有多少个对象，就对应着有多少个biting方法了么，可这些方法是完全一样的呀，只需要存在一个就可以了，那么就让它存在于类中。

属性呢？一万只狗，就应该有一万个名字啊，因此属性name应当是跟随对象而存在的。
