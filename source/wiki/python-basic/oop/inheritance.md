---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 11.4 类的继承
order: 3
---

# python面向对象---类的继承

继承是python面向对象的三大特性之一，是一种创建新类的方式，python中的继承，可以继承一个或者继承多个父类，新建的类被称之为派生类或者子类，被继承的类是父类，可以称之为基类，超类，继承是实现代码重用的重要方式。

## 1. 单继承

先看单继承的例子

```python
class Car(object):
    def __init__(self, speed, brand):
        self.speed = speed
        self.brand = brand

    def run(self):
        print("{brand}在行驶".format(brand=self.brand))

# 燃油车
class Gasolinecar(Car):
    def __init__(self, speed, brand, price):
        super().__init__(speed, brand)
        self.price = price


class Audi(Gasolinecar):
    pass


honda = Gasolinecar(130, '本田', 13000)
honda.run()

audi_car = Audi(100, '奥迪', 10000)
audi_car.run()

print(issubclass(Audi, Gasolinecar))        # 判断Audi是Gasolinecar的子类
print(issubclass(Gasolinecar, Car))
```

代码执行结果

```text
本田在行驶
奥迪在行驶
True
True
```

## 2. 继承发生了什么

继承，意味着子类将“拥有”父类的方法和属性，同时可以新增子类的属性和方法。在Gasolinecar类里，我没有写run方法，但是Gasolinecar的父类定义了run方法，因此，Gasolinecar也有这个方法，因此这个类的对象honda可以使用run方法。

Audi类没有定义任何方法，但是它继承了Gasolinecar，因此，Gasolinecar有的属性和方法，它都拥有，这里就包括了__init__方法。

super()可以用来调用父类的方法，Gasolinecar多传了一个price属性，其父类的__init__方法里有两个参数，因此，可以先调用父类的__init__方法初始化speed, brand，然后在初始化price。

## 3. 多继承

```python
class Person():
    def person_walk(self):
        print("走路")


class Wolf():
    def wolf_run(self):
        print("奔跑")


class WolfMan(Person, Wolf):
    def __init__(self, state):
        # 1表示狼形态,2表示人形态
        self.state = state
    def change(self):
        if self.state == 1:
            self.state = 2
        else:
            self.state = 1

    def run(self):
        if self.state == 1:
            self.wolf_run()
        else:
            self.person_walk()

wolf_man = WolfMan(1)

wolf_man.run()
wolf_man.change()
wolf_man.run()
```

## 4. 继承的优势

有了继承，创建一个新的类就非常的方便，因为只需要继承以前的父类就可以了，继承之后，子类拥有了父类的属性和方法，这样不仅仅是节省了代码，更重要的是，配合多态，让类的行为更加丰富
