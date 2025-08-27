---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.4 功能强大的f-string
order: 3
---

# f-string 远比你想象的强大

f-string 自从python3.6 版本被引入以后，极大的提高了开发人员在字符串格式化上效率，我们现在都知道如何使用它，但它的一些深藏特性还不被人所熟知，今天就来介绍一下f-string 的强大功能

## 1. 使用strftime 语法格式化日期

```python
from datetime import datetime

now = datetime.now()
print(now.strftime("%Y-%m-%d"))     # 2022-04-22
print(f"{now:%Y-%m-%d}")            # 2022-04-22
```

## 2. 设置对齐方式

为了让多行数据在格式化以后能够更加美观，你可以设置格式化以后的对齐方式

```python
word = "hello"

print(f"{word:*^10}")   # 居中对齐
print(f"{word:*>10}")   # 右对齐
print(f"{word:*<10}")   # 左对齐
```

程序输出

```text
**hello***
*****hello
hello*****
```

- 10 表示格式化以后的字符串的长度
- ^ 表示居中，> 表示右对齐 < 表示左对齐
- \* 表示出了word以外部分的填充字符

为了演示方便，我填充了* , 你可以将其修改为其他字符，或者去掉填充字符

```python
word = "hello"

print(f"{word:^10}")
print(f"{word:>10}")
print(f"{word:<10}")
```

程序输出

```text
  hello   
     hello
hello    
```

## 3. 格式化类的对象

```python
class User:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name

    def __str__(self):
        return f"{self.first_name} {self.last_name}"

    def __repr__(self):
        return f"User's name is: {self.first_name} {self.last_name}"

user = User("小", "明")

print(f"{user}")
print(f"{user!r}")
```

对一个对象实例进行格式化，会自动调用其__str__方法，想要调用__repr__ 可以在表达式里添加!r。其实这种格式化方法在字符串的format方法里也是可行的。

## 4. 嵌套f-string

在格式化时，可以嵌套多层

```python
number = 324.4356

print(f'{f"{number:.2f}":>10}')
```

最里面一层格式化保留两位小数，最外层格式化要求格式化以后字符串长度为10且右对齐。

## 5. 实现__format__ 方法，更加灵活的格式化

```python
class AgeFormt():
    def __format__(self, format_spec):
        format_spec = int(format_spec)
        if format_spec < 14:
            return f"年龄小于14岁"
        else:
            return f"年龄大于等于14岁"

format = AgeFormt()
age = 9
print(f"{format:{age}}")

age = 15
print(f"{format:{age}}")
```

实现__format__方法，可以将格式化的逻辑写在专门的格式化类里，让主逻辑更加简洁。
