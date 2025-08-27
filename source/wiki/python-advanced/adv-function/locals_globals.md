---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.3 locals与globals
order: 2
---

# locals与globals

## 1. globals函数

globals函数以字典的形式返回当前位置的全部全局变量， 而且对这个globals的返回值进行修改会影响全局命名空间。

在一个模块中执行

```python
print(globals())
```

你会得到一个字典，包含了当前模块的全部全局变量，这里面有你定义的函数，全局变量，还有一些是系统自带的全局变量，例如__name__, __file__, __doc__

- __name__ 模块名称
- __file__ 当前脚本的绝对路径
- __doc__ 当前模块的注释

```python
'''
当前模块的注释说明
'''
def test():
    print("调用函数test")


g_var = globals()
print(g_var)

print(__file__)
print(__name__)
print(__doc__)

g_var['test']()   # 可以调用test函数
```

函数test 也是对象，而且是全局变量，g_var是字典，字符串 'test' 作为key，其value就是函数test，因此g_var['test']()等价于test(), 这里你一定要牢牢记住一个概念，函数是对象，而且是一等公民，在函数后面使用小括号，就是在调用执行这个函数。

### 1.1 globals函数有什么用

globals函数有什么用呢？最初接触到这个函数时，我也有这样的疑问，遗憾的是，能够找到的技术文章都只讲这个函数的功能，却不讲这个函数如何在实际应用中使用，我在编写这套教程时，就决心避免这种千篇一律的讲解方式，我总是努力和你分享这些知识在实际工作中如何应用，这将成为本教程的一大特点。

说道有什么用，本质上取决于你自己在工作中遇到什么问题。我在python基础教程中的一篇名为[《为什么python没有switch/case》](http://www.coolpython.net/python_primary/logic_control/python_has_no_switch.html) 的文章中应用globals函数来寻找指定函数进行调用，你可以看这篇文章来学习如何在实践中应用globals。

## 2. locals函数

locals函数以字典的形式返回当前所在作用域的全部变量，如果你在一个模块里执行locals函数，那么它返回的与globals函数返回值相同，如果你在一个函数中执行locals函数，就只能返回这个函数所形成的局部作用域里的变量。

```python
def test(a, b):
    c = a + b
    print(locals())     # 以字典形式返回局部作用域里的变量
    return c

test(3, 5)
```

程序输出结果是

```python
{'c': 8, 'b': 5, 'a': 3}
```

locals函数在实践中有什么应用么？先来看一段简单的代码

```python
def create_sql(table, id):
    sql = "select * from {table} where id={id}".format(table=table, id=id)
    return sql

print(create_sql('user', 3))
```

函数create_sql根据传入的table 和 id拼接一个sql语句，上面代码可以使用locals函数进行小小的修改

```python
def create_sql(table, id):
    sql = "select * from {table} where id={id}".format(**locals())
    return sql

print(create_sql('user', 3))
```

locals函数返回的是一个字典，内容为

```python
{'id': 3, 'table': 'user'}
```

使用两个*表示解包，解包后作为参数传入format方法，**locals()等价于table=table, id=id。

究竟如何在实际应用中使用，还是要看实际工作中遇到了什么问题，千万不要以为，一个函数有着极为固定的用法，只要像背九九乘法表那样背下来就可以成为高手了，高手知识储备量高于常人，但高手之所以是高手，不是因为他们记住的东西多，而是他们有着超出常人的探索能力。
