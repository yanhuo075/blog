---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.2 三元表达式
order: 1
---

# 三元表达式

严格讲，python并没有三元表达式，但可以通过if语句配合else来模拟实现, 形式类似于 if ... else ... , 这个if语句构成的表达式最终会得到一个结果, if else 可以多层嵌套, 只是在书写时需要在同一行

在java语言中，你可以这样来实现一个三元表达式

```java
int a = 1;
String b = "";
b =  a > 1? "expressionA":"expressionB"
System.out.println(b)
```

当a > 1这个表达式结果为真时，将"expressionA"赋值给b，反之将"expressionB"赋值给b。python中可以使用if ... else语句来实现类似的效果

```python
a = 1
b = 'expressionA' if a > 1 else 'expressionB'
print(b)
```

当表达式 a > 1 结果为True时，将'expressionA'赋值给b
