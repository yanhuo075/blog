---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 12.8 类可以描述数据之间的关系
order: 7
---

# 轻松学会python面向对象第7篇---类可以形象的描述数据之间的关系

不考虑特殊情况，一个班主任带一个班级，这位班主任老师与学生之间就建立起了一对多的关系。

![img](http://www.coolpython.net/pictures/python_primary/easy_oop/easy_oop_descript_relation-1615988189-0.jpg)

如果不用面向对象，你难以用代码来描述他们之间的关系。

类，是一种约定，是对内容的约定，对关系的约定，对行为的约定。在没有实例被创建出来之前，仅仅通过约定，就能够让你对数据有一个非常全面且形象的理解。

```python
class Student():
    def __init__(self, name, yw_score, sx_score):
        self.name = name            # 姓名
        self.yw_score = yw_score    # 语文分数
        self.sx_score = sx_score    # 数学分数
        self.teacher = None         # 老师


class Teacher():
    def __init__(self, name):
        self.name = name            # 姓名
        self.students = []          # 学生
```

先不要急着考虑如何使用者两个类，这不是本文的重点。本文的重点是向你介绍类如何描述数据之间的关系。

类Student定义了一个teacher属性，而类Teacher里定义了一个students属性，类型是列表，结合现实中老师与学生的关系，你应当已经猜到，这两个属性将被赋值特定的对象。

```python
stu = Student('小明', 98, 99)
teacher = Teacher('张老师')

stu.teacher = teacher
teacher.students.append(stu)

print(stu.teacher.name)
print(teacher.students)
```

在现实中，你可以通过一个学生找到他的老师，也可以通过一个老师了解到他的学生。

通过面向对象技术，在代码构建出的虚拟逻辑世界中，你仍然可以通过学生找到老师，通过老师找到学生。

用面向对象的方式可以更好的来描述具体的世界，也就是建模。

类让原本抽象的代码，逻辑，概念，都具象化了，你只需要将类的描述与现实世界中对应的事物进行关联，就可以自然且顺畅的理解代码意图。

反过来，当你想用代码去完成某件事情时，就可以先思考，是否应当先定义一个或者几个类，来描述或者定义你想做的事情。
