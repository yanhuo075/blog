---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 6.5 内置函数
order: 4
---

# python内置函数深入讲解

python解释器内置了很多实现常用功能的函数和类型，他们统称为内置函数，这些内置函数无需使用import导入，在任意位置都可以直接使用。内置函数存在于__builtins__模块中，因此，内置函数的作用域是内置作用域,每个python脚本都会自动加载该模块，这正是可以随意使用内置函数的原因。

下面的代码展示如何通过获得__builtins__模块来调用内置函数。

```python
builtins = globals()['__builtins__']
print(type(builtins))

max_function = builtins.max
print(max_function([1, 4, 6, 9]))
print(builtins.input("请输入一个整数:"))
```

内置函数globals以字典的形式返回当前模块命名空间里的全局变量，这其中就包含__builtins__模块。

有些所谓的内置函数，例如list, dict等，其本质是类，而非函数。但这些类的命名并没有遵守类的命名规范，而是采用了函数的命名规范，首字母是些小而非大写。由于函数与类都是可调用对象，因此从语法上不能对他们的类型做出区分，这似乎这是python开发人员有意而为之。众多教程将它们统称为内置函数，虽然不是最准确的理解，但并不影响使用。

python的内置函数大约有70个左右，熟练的掌握并使用它们，将极大的简化你的代码，提高你的编程效率，下面是按照函数名称首字母进行分类整理的内置函数表：

| [abs](http://www.coolpython.net/method_topic/builtin-func/abs.html) | [all](http://www.coolpython.net/method_topic/builtin-func/all.html) | [any](http://www.coolpython.net/method_topic/builtin-func/any.html) | [bytes](http://www.coolpython.net/method_topic/builtin-func/bytes.html) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [bin](http://www.coolpython.net/method_topic/builtin-func/bin.html) | [bool](http://www.coolpython.net/method_topic/builtin-func/bool.html) | [bytearray](http://www.coolpython.net/method_topic/builtin-func/bytearray.html) | [compile](http://www.coolpython.net/method_topic/builtin-func/compile.html) |
| [chr](http://www.coolpython.net/method_topic/builtin-func/chr.html) | [callable](http://www.coolpython.net/method_topic/builtin-func/callable.html) | [dict](http://www.coolpython.net/method_topic/builtin-func/dict.html) | [divmod](http://www.coolpython.net/method_topic/builtin-func/divmod.html) |
| [delattr](http://www.coolpython.net/method_topic/builtin-func/delattr.html) | [dir](http://www.coolpython.net/method_topic/builtin-func/dir.html) | [eval](http://www.coolpython.net/method_topic/builtin-func/eval.html) | [exec](http://www.coolpython.net/method_topic/builtin-func/exec.html) |
| [enumerate](http://www.coolpython.net/method_topic/builtin-func/enumerate.html) | [filter](http://www.coolpython.net/method_topic/builtin-func/filter.html) | [float](http://www.coolpython.net/method_topic/builtin-func/float.html) | [frozenset](http://www.coolpython.net/method_topic/builtin-func/frozenset.html) |
| [getattr](http://www.coolpython.net/method_topic/builtin-func/getattr.html) | [globals](http://www.coolpython.net/method_topic/builtin-func/globals.html) | [hash](http://www.coolpython.net/method_topic/builtin-func/hash.html) | [hex](http://www.coolpython.net/method_topic/builtin-func/hex.html) |
| [hasattr](http://www.coolpython.net/method_topic/builtin-func/hasattr.html) | [isinstance](http://www.coolpython.net/method_topic/builtin-func/isinstance.html) | [issubclass](http://www.coolpython.net/method_topic/builtin-func/issubclass.html) | [iter](http://www.coolpython.net/method_topic/builtin-func/iter.html) |
| [int](http://www.coolpython.net/method_topic/builtin-func/int.html) | [input](http://www.coolpython.net/method_topic/builtin-func/input.html) | [id](http://www.coolpython.net/method_topic/builtin-func/id.html) | [locals](http://www.coolpython.net/method_topic/builtin-func/locals.html) |
| [list](http://www.coolpython.net/method_topic/builtin-func/list.html) | [len](http://www.coolpython.net/method_topic/builtin-func/len.html) | [max](http://www.coolpython.net/method_topic/builtin-func/max.html) | [min](http://www.coolpython.net/method_topic/builtin-func/min.html) |
| [map](http://www.coolpython.net/method_topic/builtin-func/map.html) | [next](http://www.coolpython.net/method_topic/builtin-func/next.html) | [oct](http://www.coolpython.net/method_topic/builtin-func/oct.html) | [open](http://www.coolpython.net/method_topic/builtin-func/open.html) |
| [pow](http://www.coolpython.net/method_topic/builtin-func/pow.html) | [print](http://www.coolpython.net/method_topic/builtin-func/print.html) | [reversed](http://www.coolpython.net/method_topic/builtin-func/reversed.html) | [round](http://www.coolpython.net/method_topic/builtin-func/round.html) |
| [range](http://www.coolpython.net/method_topic/builtin-func/range.html) | [repr](http://www.coolpython.net/method_topic/builtin-func/repr.html) | [set](http://www.coolpython.net/method_topic/builtin-func/set.html) | [setattr](http://www.coolpython.net/method_topic/builtin-func/setattr.html) |
| [slice](http://www.coolpython.net/method_topic/builtin-func/slice.html) | [sorted](http://www.coolpython.net/method_topic/builtin-func/sorted.html) | [staticmethod](http://www.coolpython.net/method_topic/builtin-func/staticmethod.html) | [str](http://www.coolpython.net/method_topic/builtin-func/str.html) |
| [sum](http://www.coolpython.net/method_topic/builtin-func/sum.html) | [super](http://www.coolpython.net/method_topic/builtin-func/super.html) | [tuple](http://www.coolpython.net/method_topic/builtin-func/tuple.html) | [type](http://www.coolpython.net/method_topic/builtin-func/type.html) |
| [vars](http://www.coolpython.net/method_topic/builtin-func/vars.html) | [zip](http://www.coolpython.net/method_topic/builtin-func/zip.html) |                                                              |                                                              |
