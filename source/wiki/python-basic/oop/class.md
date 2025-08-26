---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 11.2 类的定义与实例化
order: 1
---

# python面向对象---类的定义与实例化

python是一种面向对象编程语言，自然也有类的概念。python中的类通过class 关键字定义，提供了面向对象的所有标准特性，例如允许一个类继承多个基类， 子类可以覆盖父类的方法，封装，继承，多态 面向对象的三大特性python一样不少。

## 1.类的定义

定义一个类使用class关键字

```python
class Stu:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def run(self):
        print("{name} is running".format(name=self.name))
```

关于这段代码有很多新的概念需要你了解

### 1.1 类的名字

上面所定义的类的名字叫Stu，你可以使用Stu的__name__属性来访问它

```python
print(Stu.__name__)
```

### 1.2 实例方法

在函数一章，你已经学习使用def来定义函数，在类里面，同样用def关键字来定义函数，但是我们管类里的函数叫方法，在这个示例中所有的方法都是实例方法，如不特殊说明，本教程中所说的方法均指实例方法。

### 1.3 初始化函数与实例属性

注意看这个方法

```python
def __init__(self, name, age):
    self.name = name
    self.age = age
```

单词init是初始化的意思，__init__是初始化函数，当实例被构造出来以后，使用__init__方法来初始化实例属性。

name 和 age都是实例的属性，属性就是数据。说了半天实例，到底谁是实例，在类的方法里，self就是实例，如果你有其他编程语言的经验，python中的self等同于其他编程语言里的this。

## 2. 实例化

类是一种封装技术，我们把数据和方法都封装到了类里，如果你想使用数据和方法，那么你需要用类创建出一个实例来，这个过程就叫做实例化

```python
class Stu:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def run(self):
        print("{name} is running".format(name=self.name))


s = Stu('小明', 18)
s.run()
```

代码执行结果

```text
<class '__main__.Stu'>
小明 is running
```

s 就是创建出来的实例，s是类Stu的一个实例。在定义Stu的时候，要求实例必须有name和age属性，因此，在创建示例时，我们必须提供这些属性，按照__init__函数的参数列表传入'小明' 和 18。

s.run() 这行代码是在调用实例的run方法，不同于之前所学习的函数，类里的方法只能通过实例或者类来调用，调用的方式就如代码里演示的那样。

## 3. self是什么？

你或许已经注意到，类里定义的每个方法都有一个self参数，它并不是默认参数，但这个参数在方法被调用时是不需要传参的，这与之前所学过函数内容相违背。

self是方法的一个参数，在方法调用时，这个参数是默认传参的。

```python
class Stu:
    def __init__(self, name, age):
        print("在__init__方法中", id(self))
        self.name = name
        self.age = age

    def run(self):
        print("在run方法中", id(self))
        print("{name} is running".format(name=self.name))


s = Stu('小明', 18)
print("s的内存地址", id(s))
s.run()
```

程序执行结果

```text
在__init__方法中 4336221600
s的内存地址 4336221600
在run方法中 4336221600
小明 is running
```

self的内存地址，和s的内存地址是一致的，这可以证明，self就是s。函数被定义以后，在任何地方都可以调用，没有调用的主体，而类的方法，则需要一个调用主体，哪个实例调用了类的方法，self就会绑定为哪个实例，self这个参数就是哪个实例。

分析上面的输出结果，在__init__方法被调用时，s这个对象就已经被创建好了，这可以证明它不是构造方法，真正的构造方法是__new__，在没有调用构造方法前，对象是不存在的。

## 4.对象属性的访问

对象属性的访问与修改，只能通过对象来进行

```python
class Stu:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def info(self):
        print("{name}今年{age}岁".format(name=self.name, age=self.age))

    def run(self):
        print("{name} is running".format(name=self.name))

    def print(self):
        print("ok")

s = Stu("小刚", 18)
s.info()

print(s.name)
s.age = 20
s.info()
```

代码输出结果

```text
小刚今年18岁
小刚
小刚今年20岁
```

## 5. 使用类组合数据和方法

一个名为 成绩单 的文件中保存了学生的考试成绩，内容如下

```text
姓名  语文  数学  英语
小红  90   95    90
小刚  91   94    93
```

请编写程序读取该文件，使用合适的数据类型保存这些数据，输出每一个人的各个科目和分数，并计算每个学生的总的分数。

### 6.1 面向过程

```python
stus = []
with open('成绩单', 'r', encoding='utf-8') as file:
    lines = file.readlines()
    for i in range(1, len(lines)):
        line = lines[i]
        arrs = line.split()
        stus.append(arrs)

# 记住,每个列表里,第一个元素是姓名,第二个元素语文分数,后面的是数学分数,英语分数
for stu in stus:
    msg = "{name}的各科成绩如下: 语文:{yw_score}, " \
          "数学:{sx_score}, 英语:{en_score}".format(name=stu[0],
                                                yw_score=stu[1],
                                                sx_score=stu[2],
                                                en_score = stu[3]
                                                )
    print(msg)
    msg = "{name}的总成绩是{socre}".format(name=stu[0],
                                      socre=int(stu[1])+int(stu[2])+int(stu[3]))

    print(msg)
```

### 6.2 面向对象

```python
class Stu:
    def __init__(self, name, yw, sx, en):
        self.name = name
        self.yw = yw
        self.sx = sx
        self.en = en

    def score_sum(self):
        return self.yw + self.sx + self.en

    def print_score(self):
        msg = "{name}的各科成绩如下: 语文:{yw_score}, " \
          "数学:{sx_score}, 英语:{en_score}".format(name=self.name,
                                                yw_score = self.yw,
                                                sx_score = self.sx,
                                                en_score = self.en
                                                )
        print(msg)
        msg = "{name}的总成绩是{score}".format(name=self.name, score=self.score_sum())
        print(msg)

stus = []
with open('成绩单', 'r', encoding='utf-8') as file:
    lines = file.readlines()
    for i in range(1, len(lines)):
        line = lines[i]
        arrs = line.split()
        s = Stu(arrs[0], int(arrs[1]), int(arrs[2]), int(arrs[3]))
        stus.append(s)

for stu in stus:
    stu.print_score()
```

如果仅仅从代码量上来看，使用类并没有减少代码，但是从代码的可阅读性上看，使用类组织数据和方法显然更具有优势，而且，类这种概念，更加符合我们人类的思维。

我们人喜欢把各种东西归类，爬行动物，恒温动物，冷血动物，猫科动物，犬科动物，这些都是类。

狮子也是一个类，是猫科动物的子类，具体到一个真实存在的狮子，我们可以认为这个真实存在的狮子是狮子这个类的一个实例，或者说狮子这个类的一个对象。

使用类组织数据和方法，扩展性更好，可以随时添加新的属性和方法，比如，我想输出学生最高的科目分数,那么，我只需要修改类就可以了

```python
class Stu:
    def __init__(self, name, yw, sx, en):
        self.name = name
        self.yw = yw
        self.sx = sx
        self.en = en

    def score_sum(self):
        return self.yw + self.sx + self.en

    def get_max_score(self):
        if self.yw > self.sx:
            if self.yw > self.en:
                return self.yw
            else:
                return self.en
        else:
            if self.sx > self.en:
                return self.sx
            else:
                return self.en

    def print_score(self):
        msg = "{name}的各科成绩如下: 语文:{yw_score}, " \
          "数学:{sx_score}, 英语:{en_score}".format(name=self.name,
                                                yw_score = self.yw,
                                                sx_score = self.sx,
                                                en_score = self.en
                                                )
        print(msg)
        msg = "{name}的总成绩是{score}".format(name=self.name, score=self.score_sum())
        print(msg)

        msg = "最高分数是{max_score}".format(max_score = self.get_max_score())
        print(msg)
```

在程序里，调用对象print_score方法的地方不用做任何的修改。反观6.1 的写法，你不得不在for循环里做大量的修改，这样的代码可维护性是很差的。

## 5. 实例方法，类方法，静态方法， 类属性

类里的方法有3种，在第一小节中所定义的类中，方法都是实例方法。这3种方法有各自的定义方式和权限

| 名称     | 定义方法                                                     | 权限                                               | 调用方法                       |
| -------- | ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------ |
| 实例方法 | 第一个参数必须是示例，一般命名为self                         | 可以访问实例的属性和方法，也可以访问类的实例和方法 | 一般通过示例调用，类也可以调用 |
| 类方法   | 使用装饰器@classmethod修饰，第一个参数必须是当前的类对象，一般命名为cls | 可以访问类的实例和方法                             | 类实例和类都可以调用           |
| 静态方法 | 使用装饰器@staticmethod修饰，参数随意，没有self和cls         | 不可以访问类和实例的属性和方法                     | 实例对象和类对象都可以调用     |

下面修改类Stu的定义来向你展示这3种方法的区别

```python
class Stu:
    school = '湘北高中'   # 类属性
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def play_basketball(self):
        print("{name} 正在打篮球".format(name=self.name))

    @classmethod
    def sport(cls):
        print("{school}的同学都喜欢篮球".format(school=cls.school))

    @staticmethod
    def clean():
        print("打扫教室卫生")


stu = Stu("樱木花道", 17)
stu.play_basketball()       # 通过实例调用实例方法
Stu.play_basketball(stu)    # 通过类调用实例方法

Stu.sport()     # 通过类调用类方法
stu.sport()     # 通过示例调用类方法

Stu.clean()     # 通过类调用静态方法
stu.clean()     # 通过实例调用静态方法

print(stu.school)       # 通过实例访问类属性
stu.school = '山王工业'     # 只是增加了一个实例属性,类属性不会被修改
print(Stu.school)       # 通过类访问类属性
```

代码执行结果

```python
樱木花道 正在打篮球
樱木花道 正在打篮球
湘北高中的同学都喜欢篮球
湘北高中的同学都喜欢篮球
打扫教室卫生
打扫教室卫生
湘北高中
湘北高中
```

理解这3种方法的核心在于理解他们的权限。

在面向对象的设计理念中，方法必定属于某个类，如果这个方法不属于某个类，那么它就是函数了，就回归到了面向过程编程。方法属于某个类，但这个方法可能不会访问实例和类的任何属性或其他方法，对于这种方法，我们就应该把它设计成静态方法。

还有一种可能，一个方法会访问到类的属性，就像本示例中的sport方法，整个湘北高中的学生都喜欢篮球，这个方法就不是某个学生所特有的，而是整个类所拥有的一个方法，那么就需要把它设计成类方法。school是类属性，既然类拥有这个属性，那么类实例化出来的对象也自然拥有这个属性，通过类和示例都可以访问到这个属性，但是要注意，这个属性是类的，因此不能通过实例对它进行修改。
