---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.8 attr
order: 7
---

# setattr, hasattr, getattr

## 1. setattr

setattr可以给对象的属性赋值，如果对象没有这个属性则会先创建出这个属性然后赋值,先来看一个简单示例

```python
class Stu:
    pass

stu = Stu()
setattr(stu, 'name', '小红')
setattr(stu, 'age', 14)

print(stu.name, stu.age)
```

定义的Stu类没有任何对象属性，但可以通过setattr函数为对象创建出新的属性并赋值。在定义类时，如果对象属性非常多，就可以使用setattr方法为对象进行初始化操作

```python
class Stu:
    def __init__(self):
        field_lst = ['name', 'age', 'course', '_class']
        for field in field_lst:
            setattr(self, field, None)  # 所有属性先初始化为None
```

## 2. hasattr

hasattr可以判断某个对象是否具有某属性

```python
class Stu:
    def __init__(self):
        field_lst = ['name', 'age', 'course', '_class']
        for field in field_lst:
            setattr(self, field, None)  # 所有属性先初始化为None

stu = Stu()
print(hasattr(stu, 'name'))
print(hasattr(stu, 'score'))
```

程序输出结果

```text
True
False
```

## 3. getattr

getattr的作用于setattr的作用相反，它从对象里获取指定属性的值

```python
class Stu:
    pass

stu = Stu()
setattr(stu, 'name', '小红')
setattr(stu, 'age', 14)

print(getattr(stu, 'name'))
print(getattr(stu, 'age'))
```
