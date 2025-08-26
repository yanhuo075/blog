---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.8 Match-Case语法
order: 7
---

# Match-Case语法

## 1. match-case

总所周知，python没有像其他编程语言一样提供swith-case语法，而是通过if/elif/else 来实现相同的效果，但python 3.10 版本引入了Match-Case语法，这个新的语法叫做结构模式匹配，它能够提供的功能大于swith-case语法。

下面是一个简单的示例

```python
def check_number(num):
    match num:
        case 1:
            print("One")
        case 2:
            print("Two")
        case _:
            print("Other")

check_number(1)     # one
check_number(10)    # Other    
```

从使用习惯上来说，和其他语言提供的swith-case是一致的，不同之处是使用下划线来匹配任意模式，相当于default的作用，如果仅仅是提供了与其他语言中wsith-case一样的功能，我相信，也就不会引入pyhon正式发行版了，接下来，一起来看一下它有什么独特之处。

## 2. 结合或操作

```python
def check_number(num):
    match num:
        case 1 | 2:
            print("1或者2")
        case 3 | 4:
            print("3或者4")
        case _:
            print("Other")

check_number(1)
check_number(4)
```

在case语句中，你可以使用或操作，num命中其中任意一个即视为命中case的条件

## 3. 结合if

```python
def check_number(num):
    match num:
        case num if num > 0:
            print("大于0的数")
        case num if num < 0:
            print("小于0的数")
        case _:
            print("zero")

check_number(1)
check_number(-1)
```

这看起来就像是在写一个if/elif/else语句一样。

## 3. 匹配列表

```python
def check_number(list):
    match list:
        case 1, *others:
            print(f"第一个元素是1的列表, 剩余元素{others}")
        case 2, 3, *others:
            print(f"前两个元素是2和3的列表, 其余元素是{others}")
        case _:
            print("nothing")

check_number([1])           # 命中case1
check_number([1, 3])        # 命中case1
check_number([2, 3, 4])     # 命中case2
```

case 语句中，你可以用任意模式来解包列表，当列表里的数据符合这个模式时就会命中case，如果list传入的是元组，解包方法相同。

## 4. 匹配字典

除了可以解包列表，元组，还可以解包字典

```python
def run_match(data):
    match data:
        case {"name": name, "age": age}:
            print(f"{name} is {age} years old.")
        case {"name": name, "salary": salary}:
            print(f"{name}'s salary is {salary}.")
        case {"name": name,  **others}:
            print(f"{name} has other information:")
            print(others)
        case _:
            print("nothing")

run_match({"name": "小明", "age":20})                     # 命中case1
run_match({"name": "小明", "salary": "1万"})              # 命中case2
run_match({"name": "小明",  "sex": "男"})                 # 命中case3
run_match({"name": "小明", "salary": "1万", "age": 20})   # 命中case1
```

- 只要字典中包含了name 和 age 这两个key，就会命中case1
- name变量没有事先声明，但只要命中了case，就会为其赋值，case具有绑定变量值的作用

## 5. 匹配类的实例

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"(x={self.x}, y={self.y})"

def run_match(instance):
    match instance:
        case Point(x=1, y=2):
            print(instance)
        case Point(x=3, y=4):
            print(instance)
        case _:
            print("nothing")

point1 = Point(1, 2)
point2 = Point(3, 4)

run_match(point1)
run_match(point2)
```

这里要注意一点，case语句中，一定不能写成Point(1, 2)， 这等于是创建了一个新的实例，和point1完全不同的实例，match-case是结构模式匹配，你需要提供的是一个模式而不是一个具体的对象，在case语句中，Point(1, 2)会抛TypeError异常。

## 6. 总结

3.10 引入的match-case语法填补了swith-case的缺失，这使得其他编程语言从业者在学习python时不至于因缺少swith-case而感到不适应，它提供了一些新的特性，将丰富我们处理复杂逻辑的手段。
