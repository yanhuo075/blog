---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.2 compile
order: 1
---

# compile

compile() 函数将一个字符串编译为字节代码

## 1. eval模式

下面是一段简单的示例代码

```python
exp = "3 + 8 * 2"
c = compile(exp, '', 'eval')
res = eval(c)
print(res)      # 19
```

compile的语法如下

```python
compile(source, filename, mode[, flags[, dont_inherit]])
```

我们一般只用到前3个参数

1. source: 字符串或者AST（Abstract Syntax Trees）对象
2. filename: 脚本名称，如果source不是从某个脚本里读取的，则提供一个可辨认的值
3. mode 编译代码的种类。可以指定为 exec, eval, single

在上面的示例中，mode设置为eval，将source编译成一个表达式，exp的内容看上去就是一个表达式。

## 2. single模式

```python
exp = 'print("hello world")'
c = compile(exp, '', 'single')  # 单条交互式语句
exec(c)
```

程序输出结果

```python
hello world
```

python源码里对single解释是

```python
single' to compile a single (interactive) statement
```

如果只是编译一个单条交互式语句，似乎用处不大。

## 3. exec模式

exec模式是模块模式，如果你想将一段字符串编译成模块，那么需要采用exec模式，先看一个简单示例

```python
exp = 'for i in range(3):print(i)'
c = compile(exp, '', 'exec')
exec(c)
```

程序输出结果

```python
0
1
2
```

接下来看一个更加复杂，更加具有实际应用价值的示例

```python
string='''
def max(a, b):
    max_value = b
    if a > b:
        max_value = a
    print('max value is {value}'.format(value=max_value))
'''

c = compile(string, 'test.py', 'exec')
exec(c)     # 编译后,在全局作用域里可以找到max函数

max_function = globals()['max']
print(max_function.__name__)
max_function(3, 6)
```

使用exec，将字符串string编译成了一个模块，编译发生在当前模块，因此无需引入，max函数就已经存在于全局变量中了。
