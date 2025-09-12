---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: python内置函数介绍
order: 0
---

# python内置函数介绍



## a开头的内置函数



### python内置函数之abs，返回数字的绝对值

#### 1. abs函数的作用

python内置函数abs返回一个数的绝对值，参数可以是整数，浮点数，或者是实现了__abs__方法的对象。

#### 2. abs语法

```text
abs(x)
```

#### 3. 参数

abs接收一个参数，参数类型可以是int，float, 或者是实现了__abs__方法的对象。

#### 4. 示例

```python
print(abs(-1))          # 1
print(abs(-3.4))        # 3.4 
```

下面举一个实现了__abs__ 方法对象的例子

```python
class Demo():
    def __init__(self, value):
        self.value = value

    def __abs__(self):
        value = float(self.value)
        if value < 0 :
            return -1*value

demo = Demo("-34")
print(abs(demo))        # 34.0
```

demo对象实现了__abs__方法，当demo作为abs的参数时，abs函数会调用demo的__abs__方法，将方法的返回值做为abs函数的返回值。



### python内置函数all详解

#### 1. all函数作用

python内置函数all可用于判断传入的可迭代参数 iterable 中的所有元素是否都为True，如果是则返回True，反之返回False。如果可迭代对象是空的，也会返回True。

在判断元素是否为True时，只要元素不是0、空、None、False，就视为True。

#### 2. 语法

```python
all(iterable)
```

#### 3. 参数

iterable 是可迭代对象，通常传入的是列表或者元组

#### 4. 示例代码

```python
print(all([True, 4, 7]))    # True
print(all((True, False)))   # False， 因为有一个元素不是True

print(all([]))          # True
```

如果传入的是一个空列表或者空元组，all函数会返回True，这的确有点出乎意料，但all函数确实如此。



### python内置函数any详解

#### any函数作用

python内置函数any可判断传入的可迭代参数 iterable中是否至少有一个元素是True，如果是则返回True，如果一个为True的元素都没有，就返回False。如果iterable 为空，则返回Fasel，判断元素是否为True，0，None，空字符串等价于False，除此以外，等价于True。

#### any 语法

```python
any(iterable)
```

#### 参数

any函数只接受一个参数，iterable 是可迭代对象，通常传入列表和元组

#### 返回值

返回bool类型

#### 示例代码

```text
print(any([3, 5, 8, False]))        # True
print(any([]))                      # False
```



## b开头的内置函数



### python内置函数bytes详解

#### bytes功能作用

python内置函数bytes返回一个新的bytes类型的对象，bytes类型对象是不可变序列，包含范围为 0 <= x < 256 的整数。bytes可以看做是bytearray的不可变版本，它同样支持索引和切片操作 bytes语法 class bytes([source[, encoding[, errors]]])

#### bytes语法

```python
class bytes([source[, encoding[, errors]]])
```

#### 参数

可选形参source可以传入字符串，int,iterable 可迭代对象, 遵循 缓冲区接口 的对象, 不同的类型将有不同的效果：

- string ，如果source是字符串，则必须指定encoding参数，bytearray() 会使用 str.encode() 方法来将 string 转变成 bytes
- int ,如果source是int，会初始化大小为该数字的数组，并使用 null 字节填充
- 如果是一个遵循 缓冲区接口 的对象，该对象的只读缓冲区将被用来初始化字节数组
- iterable 可迭代对象， 要求元素的范围必须是 0 <= x < 256 的整数，它会被用作数组的初始内容
- 如果没有传入source参数，则返回一个长度为0的bytes数组

#### 返回值

bytes类型对象

#### 示例代码

```python
print(bytes())
print(bytes("python", encoding='utf-8'))
print(bytes(4))
print(bytes([1, 2, 3]))
```

程序输出结果

```text
b''
b'python'
b'\x00\x00\x00\x00'
b'\x01\x02\x03'
```



## python内置函数bin，返回int数值的二进制字符串

#### 1. 函数bin的作用

python内置函数bin以字符串的形式返回int类型数据的二进制表示，返回值以0b开头，紧跟着的是int的二进制表示。

#### 2. bin语法

```python
bin(x)
```

#### 3. 参数与返回值

传入的参数x的类型必须是int，bin的返回值是字符串类型

#### 4. 示例代码

```python
print(bin(3))       # 0b11
```



### python内置函数bool详解

#### 1. bool函数作用

python内置函数bool可将给定参数转换为bool类型，bool函数的返回值要么是True，要么是False，在做转换时，0， None，空字符串，空列表,空元组，空集合，空字典都会被转换为False，除此以外都转换为True。

#### 2. bool语法

```python
class bool([x])
```

#### 3. 参数与返回值

bool函数可以传入任意类型的参数，也可以不传参数，如果不传，则返回False。bool 类型是int的子类，它只有两个值，True和False

#### 4. 示例代码

```python
print(bool([]))     # False
print(bool())       # False
print(bool(1))      # True
print(bool(-1))     # True
print(bool(0))      # False
```



### python内置函数bytearray详解

#### bytearray功能作用

python的内置函数bytearray返回一个新的 bytes 数组，bytearray 类是一个可变序列，包含范围为 0 <= x < 256 的整数。

#### bytearray语法

```python
class bytearray([source[, encoding[, errors]]])
```

#### 参数

可选形参source可以传入字符串，int,iterable 可迭代对象, 遵循 缓冲区接口 的对象, 不同的类型将有不同的效果：

- string ，如果source是字符串，则必须指定encoding参数，bytearray() 会使用 str.encode() 方法来将 string 转变成 bytes
- int ,如果source是int，会初始化大小为该数字的数组，并使用 null 字节填充
- 如果是一个遵循 缓冲区接口 的对象，该对象的只读缓冲区将被用来初始化字节数组
- iterable 可迭代对象， 要求元素的范围必须是 0 <= x < 256 的整数，它会被用作数组的初始内容
- 如果没有传入source参数，则返回一个长度为0的bytes数组

#### 返回值

bytes数组

#### 示例代码

```python
print(bytearray())
print(bytearray("python", encoding='utf-8'))
print(bytearray(4))
print(bytearray([1, 2, 3]))
```

程序输出结果

```text
bytearray(b'')
bytearray(b'python')
bytearray(b'\x00\x00\x00\x00')
bytearray(b'\x01\x02\x03')
```



## C开头的内置函数



### python内置函数compile详解

#### compile函数功能作用

python内置函数compile 可以将字符串编译成字节代码或者AST对象，字节代码对象可以被exec() 或 eval() 执行。

#### compile函数语法

```python
compile(source, filename, mode, flags=0, dont_inherit=False, optimize=- 1)
```

#### 参数

- source 可以是字符串或者字节字符串，后者AST对象
- filename 是代码读取的文件名，source是从哪个文件里读取的，就用哪个文件的名称，如果不是从文件读取的source，则填写一个可辨识值，一般用''
- mode 编码模式，如果source是语句，则mode可以是exec，如果是单一表达式，则可以是eval，如果source是单个交互式语句，mode可以是single
- flags 和 dont_inherit 用来控制编译源码时的标志，一般情况下使用默认值

#### 返回值

字节代码或AST对象

#### 示例代码

**单一表达式**

```python
>>> exp = "3*3 + 7 + 8"                    
>>> code = compile(exp, '<string>', 'eval')
>>> code
<code object <module> at 0x2b9c88626ed0, file "<string>", line 1>
>>> eval(exp)
24
```

**python语句**

```python
code_string = """
def test():
    print("ok")

test()
"""

code = compile(code_string, '<string>', 'exec')
exec(code)
```

执行exec(code) 就如同执行了code_string所代表的代码，test函数会被调用，输出"ok"。

如果source只是一个表达式，那么mode应当使用eval，执行编译后的字节码也用eval；如果source是python语句，那么mode应当使用exec，执行编译后的字节码也要用exec。

最后举一个source为ast对象的例子

```python
import ast
ast_object = ast.parse("print('Hello world!')")
code = compile(ast_object, '<string>', 'exec')
exec(code)
```

程序最终输出Hello world!



### python内置函数chr详解

#### chr函数功能作用

python内置函数chr根据传入的int类型参数返回对应的Unicode 码位的字符，比如chr(65) 返回的是字符串A，在ascii码表里，字符A的十进制值是65，chr的参数值大小可以超过ascii码表的范围，chr是ord的逆函数。

#### chr函数语法

```python
chr(i)
```

#### 参数

- i int类型数值

#### 返回值

i 对应的Unicode 码位的字符

#### 实例代码

```python
>>> chr(65)
'A'
>>> chr(66)
'B'
```



### python内置函数callable详解

#### callable函数功能作用

callable函数可用于判断一个对象是否可以被调用，若对象可以被调用则返回True,反之则返回False。所谓可调用，是指代码里可以在对象后面跟上一对小括号，函数，方法，类都是可以被调用，实现了__call__方法的对象也可以被调用。

#### callable函数语法

```python
callable(object)
```

#### 参数

- object 被判断的对象

#### 返回值

返回值是bool类型，如果object可以被调用，则返回True，反之返回False

#### 实例代码

```python
print(callable(sum))            # True 内置函数sum是callable的

class Test():
    def output(self):
        print('ok')
    def __call__(self, *args, **kwargs):
        print('ok')

print(callable(Test))           # 类Test可以被调用
print(callable(Test.output))    # Test类的output方法可以被调用

test = Test()
print(callable(test))           # test对象实现了__call__ 方法，可以被调用
```



## d开头的内置函数



### python内置函数dict详解

#### dict函数功能作用

python内置函数dict可用于生成字典对象，其实dict是一个类而非函数，但命名遵守了函数命名原则而没有采用类的命名原则，使用时与函数调用无异，因此习惯性被称之为内置函数。

#### dict函数语法

```python
class dict(**kwarg)
class dict(mapping, **kwarg)
class dict(iterable, **kwarg)
```

#### 参数

- kwarg 关键字参数
- mapping 关联式的容器类型
- iterable 可迭代对象

#### 返回值

一个新字典

#### 示例代码

**传入一个字典**

```python
>>> new_dict = dict({'name': '小明', 'age': 14})
>>> new_dict
{'name': '小明', 'age': 14}
```

**传入关键字参数**

```python
>>> new_dict = dict(name='小明', age=14)
>>> new_dict
{'name': '小明', 'age': 14}
```

**传入可迭代对象**

```python
>>> new_dict = dict([('name', '小明'), ('age', 14)])
>>> new_dict
{'name': '小明', 'age': 14}
```



### python内置函数divmod详解

#### divmod函数功能作用

python内置函数divmod以元组的形式返回两个数字a，b 做除法的商和余数，当a和b均为整数时返回值等价于(a//b, a%b)

#### divmod函数语法

```python
divmod(a, b)
```

#### 参数

- a 数字
- b 数字

#### 返回值

元组

#### 实例代码

```text
>>> divmod(9 ,3)
(3, 0)
>>> divmod(9, 2)
(4, 1)
```



### python内置函数delattr详解

#### delattr函数功能作用

python内置函数delattr可以将对象的一个属性删除，如果对象本身没有这个属性，delattr会报AttributeError错误，被删除后，如果试图访问对象的这个属性，将会引发AttributeError错误。

#### delattr函数语法

```python
def delattr(x, y): # real signature unknown; restored from __doc__
    """
    Deletes the named attribute from the given object.

    delattr(x, 'y') is equivalent to ``del x.y``
    """
    pass
```

#### 参数

- object 任意对象
- name 字符串，希望被删除的对象属性

#### 返回值

无

#### 示例代码

```python
class Stu():
    def __init__(self, name, age):
        self.name = name
        self.age = age

stu = Stu('小明', '14')
print(stu.name)

delattr(stu, 'name')
print(stu.name)
```

程序输出结果

```text
小明
Traceback (most recent call last):
  File "C:/Users/zhangdongsheng/PycharmProjects/liepin/test2.py", line 10, in <module>
    print(stu.name)
AttributeError: 'Stu' object has no attribute 'name'
```

第一个print语句可以输出sut对象的name属性，delattr删除了stu的name属性，因此第二个print语句会引发AttributeError异常，如果尝试删除一个不存在的属性，也会引发AttributeError异常，如下面的语句

```text
delattr(stu, 'address')
```



### python内置函数dir详解

#### dir函数功能作用

python内置函数在没有实参时返回当前作用域中的名称列表，有实参时，返回该实参对象的属性列表，通常使用dir函数查看一个对象的属性和方法。

#### dir函数语法

```python
dir([object])
```

#### 参数

- object 可选参数，可传入一个对象

#### 返回值

不传object实参时，返回当前作用域的名称列表，有实参时，返回实参对象的属性列表

#### 示例代码

**不传实参**

```python
name = "小明"
age = 14

print(dir())
```

程序输出

```text
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__',
'__name__', '__package__', '__spec__', 'age', 'name']

列表里有很多变量是双下划线开头，这些变量都是模块的内置变量。
```

**传入实参**

```python
class Test():
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def output(self):
        print("ok")

t = Test('小明', 14)
print(dir(t))
```

程序输出结果

```text
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', 
'__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__',
'__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__',
'__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
'__str__', '__subclasshook__', '__weakref__', 'age', 'name', 'output']
```



## e开头的内置函数



### python内置函数eval详解

#### eval函数功能作用

python内置函数eval解析字符串并将其当做有效的表达式来求值并返回计算结果，由于eval不对表达式进行检查，因此存在安全漏洞，容易被不法人员利用，官方推荐使用ast.literal_eval来替代eval函数

#### eval函数语法

```python
eval(expression[, globals[, locals]])
```

#### 参数

- expression 字符串，内容为表达式
- globals 字典，默认使用当前作用域的全局命名空间
- locals 映射类型对象，默认使用当前作用域的局部命名空间

#### 返回值

表达式计算结果

#### 示例代码

```python
exp = "3 + 4*5"
print(eval(exp))   # 23
```

exp是一个字符串，eval能够解析并执行它，eval的功能非常强大，只要表达式是合法的，就能执行。

```python
def test():
    print("执行函数")

exp = "test()"
eval(exp)       # 执行函数
```

exp的内容是一个表达式，test() 调用执行函数test，eval对这种函数调用的表达式也可以解析并执行。

eval可以视作一种动态执行python代码的方法，在不提供固定编码的前提下，通过生成exp表达式然后用eval执行的方式动态执行python代码。



### python内置函数exec详解

#### exec函数功能作用

python内置函数exec支持动态执行python代码，传入exec函数的object实参可以是字符串，也可以是字节码对象。如果object实参是字符串则会被exec函数编译并执行，如果是字节码对象则会被直接执行。通常exec配合compile函数来使用。

#### exec函数语法

```python
exec(object[, globals[, locals]])
```

#### 参数

- object 可以是字符串或者字节码对象
- globals 可选参数，如果提供了必须是字典对象，若不提供，则使用当前作用域里的globals
- locals 可选参数，任意字典映射对象

一旦提供了globals 和 locals， 代码执行起来就像嵌入到某个类定义中一样

#### 返回值

无

#### 示例代码

object实参是字符串

```python
exec("print('ok')")     # ok
exec("4 + 5")
```

虽然字符串"4 + 5" 也能被exec执行，但并没有返回结果，这一点一定要注意

object实参是字节码

```python
>>> exp = "3*3 + 7 + 8"                    
>>> code = compile(exp, '<string>', 'eval')
>>> code
<code object <module> at 0x2b9c88626ed0, file "<string>", line 1>
>>> eval(exp)
24
```



### python内置函数enumerate详解

#### enumerate函数功能作用

python内置函数enumerate一般是for循环语句中使用，用于遍历可迭代对象，enumerate函数返回一个迭代器，迭代器的__next__方法返回一个元组，元组的第一个元素是从0开始的计数值，第二个元素是计数值所对应的可迭代对象里的元素。

#### enumerate函数语法

```python
enumerate(iterable, start=0)
```

#### 参数

- iterable 可迭代对象，如列表，元组，字符串
- start 默认从0开始，start不会影响对可迭代对象的遍历过程，只是改变计数值的起始值

#### 返回值

迭代器

#### 示例代码

enumerate函数返回的是一个迭代器，可以使用list函数将其转为列表

```python
>>> lst = ['python', 'php', 'java']
>>> list(enumerate(lst))
[(0, 'python'), (1, 'php'), (2, 'java')]
>>> list(enumerate(lst, start=1))
[(1, 'python'), (2, 'php'), (3, 'java')]
```

并没有因设置start=1而导致enumerate的返回值有所减少，只是各项遍历的值的计数值发生了变化。，

在for循环中使用enumerate，可以在遍历的同时得到对象的索引

```python
lst = ['python', 'php', 'java']

for index, item in enumerate(lst):
    print(f"{item} 的索引是{index}")
```

程序输出

```python
python 的索引是0
php 的索引是1
java 的索引是2
```



## e开头的内置函数



### python内置函数filter详解

### filter函数功能作用

python内置函数filter的语法是filter(function, iterable)，它使用function对iterable中的数据进行过滤，只保留那些返回True的元素。

filter(function, iterable)相当于一个生成器表达式，当function参数不为None时，等价于(item for item in iterable if function(item))，当function为None时，等价于(item for item in iterable if item)。

#### filter函数语法

```python
filter(function, iterable)
```

#### 参数

- function 必须是函数
- iterable iterable 可以是一个序列，一个支持迭代的容器，或一个迭代器

#### 返回值

返回迭代器

#### 示例代码

**function为None**

```python
lst = [1, False, 2, 0, True]
for item in filter(None, lst):
    print(item)
```

程序输出

```text
1
2
True
```

**function为lambda函数**

```python
lst = [1, 2, 3, 4, 5]

for item in filter(lambda x: x%2==0, lst):
    print(item)
```

程序输出

```python
2
4
```

**function为普通函数**

```python
lst = [1, 2, 3, 4, 5]

def bt_3(x):
    if x > 3:
        return True
    return False

for item in filter(bt_3, lst):
    print(item)
```

程序输出

```text
4
5
```



### python内置函数float详解

#### float函数功能作用

python内置函数将数字或字符串转为float类型参数，如果不传入实参，则返回0.0，如果传入的实参是字符串，那么字符串在字面上应当符合数字的定义，且允许字符串开头是+号或者-号。

#### flaot函数语法

```text
class float([x])
```

#### 参数

- x 可选参数，不传时，float返回0.0， x可以是int，float，bool， 字符串类型

#### 返回值

float类型数据

#### 示例代码

```python
>>> float()
0.0
>>> float(1)
1.0
>>> float(2.0)
2.0
>>> float("3")
3.0
>>> float("4.")
4.0
>>> float("-5")
-5.0
```



### python内置函数frozenset详解

#### forzenset函数功能作用

python内置函数forzenset 返回一个不可改变的集合，frozenset集合一旦被创建就无法被修改，既不能增加新的元素也不能删除元素。由于无法修改，frozenset可以做字典的key或者另一个集合的元素。

#### forzenset函数语法

```text
frozenset([iterable])
```

#### 参数

- iterable 可选参数，可迭代对象

#### 返回值

frozenset，不可修改的集合对象

#### 实例代码

```python
>>> immutable_set = frozenset([1, 2, 3])
>>> immutable_set
frozenset({1, 2, 3})
>>> immutable_set.add(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'add'
```

frozenset对象没有add方法，不能添加新的元素。



## e开头的内置函数



### python内置函数getattr详解

#### getattr函数功能作用

python内置函数getattr根据属性名称返回对象的属性值，如果属性不存在，可以在getattr的参数里指定返回的默认值，如果没有指定default，引发AttributeError。

#### getatter函数语法

```python
getattr(object, name[, default])
```

#### 参数

- object 对象
- name 字符串，对象的属性名称
- default 当属性不存在时默认的返回值

#### 返回值

对象的属性值

#### 示例代码

```python
class Stu():
    def __init__(self, name):
        self.name = name

stu = Stu("小明")
print(getattr(stu, 'name'))         # 小明
print(getattr(stu, 'age', 14))      # 14
```



### python内置函数globals详解

#### globals函数功能作用

python内置函数globals以字典的形式返回当前模块命名空间里的全局变量，这些全局变量包含有关程序的所有必要信息，比如__name__, __builtins__

#### globals函数语法

```python
globals()
```

#### 参数

globals函数没有参数

#### 返回值

字典

#### 示例代码

```python
class Stu():
    def __init__(self, name):
        self.name = name

g_dict = globals()
print(g_dict)
```

程序输出

```python
{'__name__': '__main__', '__doc__': None, '__package__': None,
'__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x000001D46DF70548>, '__spec__': None, '__annotations__': {},
'__builtins__': <module 'builtins' (built-in)>, '__file__': 'C:/Users/zhangdongsheng/PycharmProjects/liepin/test2.py',
'__cached__': None, 'Stu': <class '__main__.Stu'>, 'g_dict': {...}}
```

除了模块自身的全局变量，globals还返回了我定义的Stu和g_dict。



## h开头的内置函数



### python内置函数hash详解

#### hash函数功能作用

python内置函数hash返回一个对象的hash值，hash值在字典查找元素用来快速比较字典的键，相同大小的数字例如1和1.0 拥有相同的hash值，尽管他们是不同的类型。

并不是所有的对象都有哈希值，比如列表，字典，集合这类可变对象就没有哈希值。

#### hash函数语法

```python
hash(object)
```

#### 参数

- boject 被求hash值的对象

### 返回值

hash值，int类型

#### 示例代码

```python
print(hash(1))      # 1
print(hash(1.0))    # 1
print(hash("1"))    # 3912875043100631943
```



### python内置函数hex详解

#### hex函数功能作用

python内置函数hex以字符串的形式返回int数据的16进制表示形式，字符串以0x为前缀，如果传入hex的参数不是int类型，那么这个对象必须实现__index__()方法，hex将调用该方法并返回。

#### hex函数语法

```python
hex(x)
```

#### 参数

- x int类型对象或者是实现了__index__()方法的对象

#### 返回值

对象的16进制表示形式

#### 示例代码

```python
>>> hex(4)
'0x4'
>>> hex(16)
'0x10'
>>> hex(255)
'0xff'
>>> hex(-255)
'-0xff'
```



### python内置函数hasattr详解

#### hasattr函数功能作用

python内置函数hasattr判断对象是否拥有某个属性，如果有则返回True，若没有则返回False。

#### hasattr函数语法

```python
hasattr(object, name)
```

#### 参数

- object 对象
- name 属性的名称

#### 返回值

返回True或者False

#### 示例代码

```python
class Stu():
    def __init__(self, name):
        self.name = name

stu = Stu("小明")
print(hasattr(stu, 'name'))     # True
print(hasattr(stu, 'False'))    # False
```



## i开头的内置函数



### python内置函数isinstance详解

#### isinstance(object, classinfo)函数功能作用

python内置函数isinstance用于判断一个object对象是否是给定classinfo的实例对象，如果是则返回True，反之返回False。classinfo可以是类，也可以是由类组成的元组。

#### isinstance函数语法

```python
isinstance(object, classinfo)
```

#### 参数

- object 实例对象
- classinfo 给定的类，或者是由类组成的元组

#### 返回值

bool类型

#### 示例代码

**基础数据类型的例子**

```python
>>> isinstance(1, int)
True
>>> isinstance('', str)
True
```

**自定义类的例子**

```python
class A():
    pass

class B(A):
    pass

b = B()
print(isinstance(b, B))     # True
print(isinstance(b, A))     # True
```

b是B的示例对象，但A是B的父类，因此isinstance(b, A)也同样返回True。

**classinfo是元组的例子**

```python
>>> isinstance(1, (int, str))
True
```

classinfo传入的实参是元组(int, str)， 这里有两个类，object如果是其中某个一个类的实例对象，isinstance就会返回True。



### python内置函数issubclass详解

#### issubclass(class, classinfo)函数功能作用

python内置函数issubclass判断一个类是否是另一个类的子类，判断规则如下：

1. 一个类可以认为是自己的子类
2. 两个类存在间接继承关系，认为是子类
3. classinfo可以是一个元组，class是其中某一个类的子类，函数就返回True。

#### issubclass函数语法

```text
def issubclass(class, classinfo):
```

#### 参数

- class 类
- classinfo 类或者元组

#### 返回值

True或者False

#### 示例代码

```python
print(issubclass(bool, int))        # True

class A:
    pass

class B(A):
    pass

class C(B):
    pass

print(issubclass(C, A))                 # True A 是B的父类，B是C的父类
print(issubclass(C, (A, list)))         # True C是元组中某个类的子类
```



### python内置函数iter详解

#### iter函数功能作用

python内置函数iter返回一个迭代器对象，iter函数有两个参数，object 和 sentinel，如果没有sentinel实参，那么object必须是可迭代对象即必须实现__iter__() 方法。sentinel ，object必须是可调用对象。

#### iter函数语法

```python
iter(object[, sentinel])
```

#### 参数

- object 可迭代对象或者可调用对象
- sentinel 当传入sentinel实参是，object必须是可调用对象，iter会一直调用object直到返回sentinel

#### 示例代码

**不传sentinel**

```python
lst = [1, 2, 3 ,4]

lst_iter = iter(lst)
print(next(lst_iter))
print(next(lst_iter))
print(next(lst_iter))
print(next(lst_iter))
```

iter返回的是一个迭代器，使用next函数可以从迭代器中取出具体的值。

**传sentinel**

```python
from functools import partial

with open('source_id', 'rb') as f:
    for block in iter(partial(f.read, 64), b''):
        print(len(block))
```

partial(f.read, 64) 返回一个偏函数，是可调用对象，一次读取64字节的数据。在for循环中，iter会一直调用这个偏函数，直到偏函数返回的内容是b''， 这代表文件内容读取到了末尾。



### python内置函数int详解

#### int函数功能作用

python内置函数int将数字或者字符串转换成int类型数据。

#### int函数语法

```python
class int(x, base=10)
```

#### 参数

- x 数字或者字符串类型
- base 默认是10，表示10进制，还可以取值 0 或者 2-36 ,如果提供了base， x必须是字符串

#### 返回值

int类型数据

#### 示例代码

```python
>>> int(3.4)
3
>>> int("54")
54
>>> int("13", base=8)
11
>>> int('101', base=2)
5
```

前两个例子容易理解，后两个例子需要你能够了解如何进行进制转换，8进制是逢8进1，因此8进制13中的1表示的是8的1次方，3 就表示3,8+3 = 11，正是int函数的转换结果。



### python内置函数input详解

#### input函数功能作用

python内置函数input从标准输入中读取一行数据，将其转换为字符串（除了末尾的换行符）并返回，input函数永远返回字符串。

#### input函数语法

```python
input([prompt])
```

#### 参数

- prompt 可选参数，如果存在则写入到标准输出，起到提示用户的目的

#### 返回值

以字符串的方式返回用户输入的一行数据

#### 示例代码

```python
>>> value = input("请输入一个整数:")
请输入一个整数:34
>>> print(value)
34
```



### python内置函数id详解

#### id函数功能作用

python内置函数id返回对象的唯一标识符，这个值是一个整数，在对象的整个生命周期中这个值是唯一的而且不变，在cpython实现里，id函数返回的是对象的内存地址。

#### id函数语法

```python
id(object)
```

#### 参数

- object 任意对象

#### 返回值

对象的唯一标识符，在cpython中，唯一标识符就是对象的内存地址

#### 示例代码

```python
>>> a = []
>>> b = []
>>> id(a)
47541564134600
>>> id(b)
47541564136520
```

变量a和变量b 都是空列表，但他们的内存地址是不相同的，内存地址是否相同可以作为判断两个对象是否是同一个对象的依据。

```python
>>> a == b
True
>>> a is b
False
```



## l开头的内置函数



### python内置函数locals详解

#### locals函数功能作用

python内置函数locals以字典的形式返回当前作用域的变量，在全局作用域中调用locals函数的返回值与调用globals函数相同

#### locals函数语法

```python
locals()
```

#### 参数

locals函数没有参数

#### 返回值

当前作用域的变量

#### 示例代码

```python
def test():
    a = 4
    b = 5
    print(locals())     # {'a': 4, 'b': 5}

test()
print(locals())
```

在全局作用域中调用locals函数，得到的返回值与调用globals相同。



### python内置函数list详解

#### list函数功能

python内置函数list将可迭代对象转换为list类型，可以转换的对象有字符串，列表，元组，range函数的返回值，字典keys，values方法的返回值。

#### list函数语法

```text
class list([iterable])
```

#### 参数

- iterable 可迭代对象

#### 返回值

列表

#### 示例代码

```python
>>> list("abc")
['a', 'b', 'c']
>>> list(("ab", 2, "cd"))
['ab', 2, 'cd']
>>> list([4, 5, 6, 8,])
[4, 5, 6, 8]
>>> dic = {"name": '小明', 'age': 14}
>>> list(dic.keys())
['age', 'name']
>>> lsit(dic.values())
>>> list(dic.values())
[14, '\xe5\xb0\x8f\xe6\x98\x8e']
```



### python内置函数len详解

#### len函数功能作用

python内置函数len返回对象的长度，实参可以序列，例如列表，元组，字符串，字节串，也可以是集合，例如字典，set，frozen set。

关于len函数，一直存在一个讨论话题，其他编程语言都将这个功能设计为对象的方法，比如java中string.length，为什么python为什么要设计成内置函数呢。

迄今为止，最有说服力的解释是这样回答的：practicality beats purity，这是python之禅中的一条，翻译成中文是：实用胜过纯粹。当你使用len函数获取一个字符串的长度时，python解释器会在内存中直接从字符串底层的一个c 结构体里取出长度值，这比调用方法要快的多。获取对象的长度是一个很常见的操作，因此必须高效。

#### len语法

```python
len(s)
```

#### 参数

- s 序列对象或者集合对象

#### 示例

```python
>>> len([1, 3, 4])
3
>>> len("sdfsf")
5
>>> len({'name': 'python'})
1
>>> len(set([3, 4, 5, 6]))
4
```



## m开头的内置函数



### python内置函数min详解

#### min函数功能作用

python内置函数min返回可迭代对象的最小元素或者多个实参中的最小值。如果min函数只传入一个位置参数，那么这个实参必须是可迭代对象，min函数返回可迭代对象的最小值，如果传入多个位置参数，min函数返回他们中的最小值。

#### min函数语法

```python
min(iterable, *[, key, default])
min(arg1, arg2, *args[, key])
```

#### 参数

- iterable 可迭代对象，必须非空
- arg1, arg2 多个位置参数

#### 返回值

- 只传入一个可迭代对象时，返回可迭代对象的最小值
- 传入多个位置参数时，返回他们中的最小的

#### 示例代码 

**传入可迭代对象**

```
`python >>> min([4, 6, 1, 3])`
 `1`
 `>>> min((4, 28, 4, 88))`
 `4`
 `>>> min("abcd")`
 `'a'`
 `>>> min(set([4, 6, 8, 10]))`
 `4`
 `>>> min([])`
 `Traceback (most recent call last):`
 `File "<stdin>", line 1, in <module>`
 `ValueError: min() arg is an empty sequence`
```

可迭代对象必须有值，不能为空，否则会报ValueError错误。

**传入多个位置参数**

```python
>>> min(4, 6, 3)
3
```



## python内置函数max详解

#### max函数功能作用

python内置函数max返回可迭代对象最大的元素或者返回多个实参中的最大值。如果max函数只传入了一个位置参数，那么这个实参必须是**非空**的可迭代对象，max返回可迭代对象的最大致，如果传入了多个位置参数，则返回最大的位置参数。

#### max函数语法

```python
max(iterable, *[, key, default])
max(arg1, arg2, *args[, key])
```

#### 参数

- iterable 可迭代对象
- arg1, arg2 多个位置参数

#### 示例代码

**传入可迭代对象**

```python
>>> max([4, 6, 1, 3]) 
6
>>> max((4, 28, 4, 88))
88
>>> max("abcd")
'd'
>>> max(set([4, 6, 8, 10]))
10
>>> max([])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: max() arg is an empty sequenc
```

四个例子，我分别传入了列表，元组，字符串，集合，注意，他们必须都是非空的，否则就会报ValueError错误。

**传入多个位置参数**

```python
>>> max(4, 6, 7, 38)
38
```

这种用法并不多见，如果有多个实参，通常人们会将其放入列表或者元组中传入max函数。



### python内置函数map详解

#### map函数的功能作用

python内置函数map的原型是

```text
map(function, iterable, ...)
```

map函数返回一个将 function 应用于 iterable 中每一项并输出其结果的迭代器，map函数的第一个参数可以是lambda函数也可以是自定一函数，第二个参数是可变参数，可以传入多个可迭代对象。

#### map函数语法

```python
map(function, iterable, ...)
```

#### 参数

- function 函数，可以是lambda函数，也可以是自定义函数
- iterable 可迭代对象，当有多个可迭代对象时，最短的可迭代对象耗尽则整个迭代就结束。

#### 返回值

迭代器

#### 示例代码

**function 参数是lambda函数**

```python
lst1 = [1, 2, 3]

lst = map(lambda x: x*2, lst1)
print(lst)

for item in lst:
    print(item)
```

执行结果

```text
<map object at 0x105664358>
2
4
6
```

**传入多个可迭代对象**

```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]

lst = map(lambda x, y: x*y, lst1, lst2)
print(lst)

for item in lst:
    print(item)
```

这段代码里，传入了lst1,lst2两个可迭代对象，在map的内部，会同时遍历他们，从这两个列表里各自取出一个元素作为参数传给lambda函数，最终的效果就是1*4， 2*5，3*6， 程序输出结果

```text
<map object at 0x1032f1400>
4
10
18
```

**传入自定义函数**

```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]

def func(x, y):
    return x*y

lst = map(func, lst1, lst2)
print(lst)

for item in lst:
    print(item)
```

几乎与上一段代码一模一样，只是用func函数替代了lambda函数，如果处理逻辑较为复杂，那么使用自定义函数是合理的选择，毕竟lambda函数只适合编写简单的逻辑。



## n开头的内置函数



### python内置函数next详解

#### next函数功能作用

python内置函数next通过调用对象的__next__方法从迭代器中取值，当迭代器耗尽又没有为next函数设置default默认值参数，使用next函数将引发StopIteration异常。

#### next函数语法

```python
next(iterator[, default])
```

#### 参数

- iterator 迭代器
- default 可选参数，当迭代器耗尽时的默认返回值

#### 返回值

迭代器中的元素

#### 示例代码

```python
>>> lst = [1, 2, 3]
>>> lst_iter = iter(lst)
>>> next(lst_iter)
1
>>> next(lst_iter)
2
>>> next(lst_iter)
3
>>> next(lst_iter)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> next(lst_iter, 0)
0
```

列表lst是可迭代对象，iter函数一个迭代器，next函数作用于迭代器，每一次都从迭代器中取出一个元素。列表里共有3个元素，3次调用next函数后，迭代器已经耗尽，因此第4次调用next函数将引发StopIteration异常。第5次调用next函数时，我为其指定了default参数，如果迭代器耗尽，则返回默认值0。



## o开头的内置函数



### python内置函数oct详解

#### oct函数功能作用

python内置函数oct将一个int类型数据转换成前缀为“0o”的八进制字符串，如果传入的对象不是int类型，则必须实现__index__方法返回一个int对象。

#### oct函数语法

```python
oct(x)
```

#### 参数

- x , int类型对象或者实现了__index__方法的对象

#### 返回值

前缀为“0o”的八进制字符串

#### 示例代码

```text
>>> oct(22)
'0o26'
>>> oct(-22)
'-0o26'
```

22的八进制表示方法是0o26。



### python内置函数open详解

#### open函数功能作用

python内置函数open打开文件并返回对应的file object，如果文件不存在则引发FileNotFoundError异常，如果文件存在却不能被打开，则引发OSError异常。open函数是python专门用于读写文件的函数。

#### open函数语法

```python
open(file, mode='r', buffering=- 1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

#### 参数

- file , path-like object， 可以是表示文件路径的字符串，也可以是件对应的整数类型文件描述符
- mode , 打开文件的模式，参见教程[文件读写](http://www.coolpython.net/python_primary/data_storage/file.html)
- buffering ，缓存方式，参见[python 写文件的buffer策略](http://www.coolpython.net/informal_essay/21-12/python-file-buffer.html)
- encoding , 以哪种编码方式打开文件
- errors, 用于指定如何处理编码和解码错误
- newline 控制 universal newlines 模式如何生效（它仅适用于文本模式）
- closefd 如果 closefd 为 False 且给出的不是文件名而是文件描述符，那么当文件关闭时，底层文件描述符将保持打开状态。如果给出的是文件名，则 closefd 必须为 True （默认值），否则将触发错误
- opener 自定义开启器

#### 返回值

file object

#### 示例代码

以读模式打开文件

```text
f = open('data.txt', 'r', encoding='utf-8')
```

以写模式打开文件

```python
f = open("data.txt", 'w', encoding="utf-8")
```

打开的文件对象一定要记得在使用结束后调用colse方法关闭文件，否则将引发内存泄漏，使用with 关键字可以避免遗忘关闭文件

```text
with open("data.txt", 'r', encoding="utf-8")as f:
    pass
```



## p开头的内置函数



### python内置函数pow详解

#### pow函数功能作用

python内置函数pow返回一个数的exp次幂，你可以利用pow函数计算一个数的平方，3次方等常见的操作。如果exp是负数，则是计算一个数的倒数。

#### pow函数语法

```python
pow(base, exp[, mod])
```

#### 参数

- base int 整数
- exp int整数，pow(base, exp) 等价于base**exp
- mod 如果提供了这参数，则先计算先进行幂运算，然后对mod取模，pow(base, exp, mod) 等价于(base**exp) % mod

#### 返回值

base的exp次幂

#### 示例代码

```python
>>> pow(2, 2)
4
>>> pow(2, 3)
8
>>> pow(2, 3, 2)
0
>>> pow(2, -1)
0.5
>>> pow(2, -2)
0.25
```



### python内置函数print详解

#### print函数功能作用

python内置函数print将数据输出到file参数指定的文本流中，参数file默认为sys.stdout 即标准输出。print函数是非常简单但同时又非常重要的内置函数，在编写python代码时可输出变量值验证逻辑是否正确，查找程序bug。

#### print函数语法

```python
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

#### 参数

- *objects 可变参数，可以传入多个对象用于输出打印
- sep 多个对象输出在一行，中间使用sep进行分割
- end print函数在一次调用后输出打印以end结尾，默认是以\n结尾，因此一次print会输出一行数据
- file， 指定print会将*objects输出到哪里，默认输出到标准输出，也可以输出到文件中
- flush 如果为True，则不会进行缓存，而是强制刷新，如果为False，是否缓存取决于file参数传入的对象

#### 返回值

无

#### 示例对象

**输出多个对象使用逗号分隔**

```python
>>> print(2, 3, 4, sep=',')
2,3,4
```

**输出数据以|结尾**

```python
print("ok", end="|")
print("ok", end="|")
```

执行一次print函数，输出的内容以| 结尾，连续两次调用print，输出的内容会在同一行

```text
ok|ok|
```

**print函数将数据输出到文件中**

```python
f = open('print.log', 'w')
print("输出到文件", file=f)
f.close()
```

违反以往的经验，你在终端里看不到任何输出，打开print.log可以看到print函数的输出内容。



## r开头的内置函数



### python内置函数range详解

#### range函数功能作用

python内置函数range以指定步长从start到stop产生一个整数序列，start默认为0，step默认为1。在python2中range函数返回的是列表，python3开始range函数返回的是一个可迭代对象。range函数多用于for循环和列表推导式。

#### range函数语法

```python
class range(stop)
class range(start, stop[, step])
```

#### 参数

- start int类型，序列起始值
- stop int类型，序列结束值
- step int类型，步长，可以为负数

产生的序列为左闭右开[start, stop)，序列的最大值一定小于stop。

#### 示例代码

**range函数仅传stop**

```python
>>> type(range(10))
<class 'range'>
>>> list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

range函数的返回值是range类型，是可迭代对象，使用list函数可将其转为列表，程序仅指定了stop，start默认为0， step 默认为1。

**range函数指定start和stop**

```python
>>> list(range(1, 5))
[1, 2, 3, 4]
```

step默认为1，产生的序列范围是[1, 5)，左闭右开，序列最大值小于stop。**为什么range所产生的序列的最大值小于stop呢，这里的原因与列表切片操作不包含结束索引是相同的。**

range函数产生序列不包含stop，有利于理解程序的行为，比如range(5)产生的序列是0，1，2，3，4， 序列从0开始，到4结束，一共5个数值，刚好于range传入的stop参数相呼应，你能一眼看出range函数产生序列的长度。但如果包含了stop，range(5) 产生的序列是从0到5，序列长度就是6，这样极容易产生误解和混乱。如果range(5) 产生的序列是从1到5，包含了5同时长度也是5，似乎也说得通，但编程语言的索引都是从0开始的，在实践中，我们产生从0开始的序列比产生从1开始的序列更加有用。

**在指定start和 stop的情况下，range函数产生序列的长度正好是stop - start。**

**range函数指定step**

```python
>>> list(range(1, 10, 2))
[1, 3, 5, 7, 9]
>>> list(range(10, 0, -2))
[10, 8, 6, 4, 2]
```



### python内置函数repr详解

#### repr函数功能作用

python内置函数repr返回对象的可打印形式字符串。如果对象实现了__repr__()方法，repr函数则调用对象的__repr__()得到返回值。

#### repr函数语法

```python
repr(object)
```

#### 参数

- object 对象

#### 返回值

对象的可打印形式字符串

#### 示例代码

**repr函数返回类和对象的可打印字符串形式。**

```python
class A():
    pass

a = A()
    
print(repr(A))   # <class '__main__.A'>
print(repr(a))   # <__main__.A object at 0x000002115DC7EA48>
```

**对象实现了__repr__()方法**

```python
class A():
    def __repr__(self):
        return f"我是一个{self.__class__}的对象，内存地址是{id(self)}"

a = A()

print(repr(A))
print(repr(a))
```

程序输出

```text
<class '__main__.A'>
我是一个<class '__main__.A'>的对象，内存地址是2159644609864
```



### python内置函数reversed详解

#### reversed函数功能作用

python内置函数reversed返回一个反向的迭代器，可用于反向遍历列表，元组，字符串，reversed函数要求传入的实参必须实现__reversed__()方法或者或者是支持该序列协议（具有从 0 开始的整数类型参数的 __len__() 方法和 __getitem__() 方法）

#### reversed函数语法

```python
reversed(seq)
```

#### 参数

- seq 序列对象

#### 返回值

反向的迭代器

#### 示例代码

```python
lst = [1, 2, 3, 4, 5]

reverse_iter = reversed(lst)
for item in reverse_iter:
    print(item)
```

程序输出结果

```text
5
4
3
2
1
```



### python内置函数round详解

#### round函数功能作用

python内置函数round返回 number 舍入到小数点后 ndigits 位精度的值，通常用于float类型数据，保留指定位数。不过由于大多数十进制小数实际上都不能以浮点数精确地表示，因此round函数的返回值有时并不能符合预期，比如round(2.675, 2) 返回的是2.67， 而非2.68。

如果 ndigits 被省略或为 None，则返回最接近输入值的整数。

#### round函数语法

```python
round(number[, ndigits])
```

#### 参数

- number 数字类型
- ndigits 保留精度位数

#### 返回值

数字类型

#### 示例代码

```python
>>> round(2.675, 2)
2.67
>>> round(2.675, 1)
2.7
>>> round(0.5)     
0
>>> round(0.6)
1
```



## s开头的内置函数



### python内置函数set详解

#### set函数功能作用

python内置函数set返回一个新的集合，它是内置类型，函数可以传入列表，字符串，集合，或其他可迭代对象。

#### set函数语法

```python
class set([iterable])
```

#### 参数

- iterable 可迭代对象

#### 返回值

集合

#### 示例代码

```python
>>> set([1, 3, 4])
{1, 3, 4}
>>> set((3, 4, 5,))
{3, 4, 5}
>>> set("1234")
{'3', '4', '1', '2'}
>>> set(range(0, 10))
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```



### python内置函数setattr详解

#### setattr函数详解

python内置函数setattr与getattr函数相对应，可以为一个对象的某个属性设置属性值。setattr函数极大的扩展了python面向对象的可操作性，使之更加的灵活。

#### setattr函数语法

```python
setattr(object, name, value)
```

#### 参数

- object 对象
- name 字符串，属性名称
- value 属性值

#### 示例代码

```python
class Dog():
    def __init__(self, name):
        self.name = name

dog = Dog("拉布拉多")
setattr(dog, 'age', 4)
print(dog.age)
```

通过setattr方法为dog对象增加了age属性，在代码设计时，可以用这种方法为对象临时增加一些属性。



### python内置函数slice详解

#### slice函数功能作用

python内置函数slice返回一个 slice 对象，代表range(start, stop, step) 指定索引集的切片，slice函数必须配合range来使用。

#### slice函数语法

```text
class slice(stop)
class slice(start, stop[, step])
```

#### 参数

- start 切片开始索引
- stop 切片结束索引
- step 步长

#### 示例代码

```python
cut = slice(5)
seq = range(10)
for i in seq[cut]:
    print(i)
```

程序输出结果

```text
0
1
2
3
4
```



### python内置函数sorted详解

#### sorted函数功能作用

内置函数sorted根据 iterable 中的项返回一个新的有序的列表，它不仅可以对列表进行排序，对其他的可迭代对象也能排序，因此相比于列表的sort方法有更广的适用范围。

#### sorted函数语法

```python
sorted(iterable, key=None, reverse=False)
```

#### 参数

- iterable 可迭代对象
- key 如果为None，则比较元素大小，key可以是一个函数，函数从每个元素中提取用于比较的键
- reverse 默认为False，表示从小到大进行排序

#### 返回值

列表

#### 示例代码

**比较元素大小**

```python
lst = [5, 2, 1, 3]
sort_lst = sorted(lst, reverse=True)
print(sort_lst)         # [5, 3, 2, 1]
```

如果列表里的元素无法直接比较大小，可以使用key参数从每个元素中提取用于比较大小的key。

```python
lst = [
    {'name': '小明', 'score': 97},
    {'name': '小红', 'score': 90},
    {'name': '小刚', 'score': 95},
]

sort_lst = sorted(lst, key=lambda x: x['score'])
for item in sort_lst:
    print(item)
```

列表里存储的是字典，两个字典之间不能直接比较大小，想要对他们进行排序，就必须指定两个字典之间比较大小时使用哪个具体的值，我在代码里指定使用score来代表字典进行大小比较。程序输出结果

```python
{'name': '小红', 'score': 90}
{'name': '小刚', 'score': 95}
{'name': '小明', 'score': 97}
```



### python内置函数staticmethod详解

#### staticmethod函数功能作用

内置函数staticmethod将类的普通方法转为静态方法，静态方法不会接收隐式的第一个参数，声明静态方法请使用staticmethod。

#### staticmethod函数语法

```python

```

#### 示例代码

```python
class Bird():
    @staticmethod
    def fly():
        print('it is flying')

Bird.fly()      # it is flying
```

静态方法可以直接通过类来调用。



### python内置函数str详解

#### str函数功能作用

内置函数str将其他类型的数据转换为字符串，如果传入的bytes类型数据，可以通过encoding来指定编码。

#### str函数语法

```python
class str(object='')
class str(object=b'', encoding='utf-8', errors='strict')
```

#### 参数

- object 几乎可以是任意类型
- encoding 当object是bytes类型时可以通过encoding指定编码，默认是utf-8
- errors 解码失败时如何响应

errors有6个可选值

1. strict - 默认响应，失败时引发UnicodeDecodeError异常
2. ignore - 从结果中忽略不可编码的 Unicode
3. replace - 将不可编码的 Unicode 替换为问号
4. xmlcharrefreplace - 插入 XML 字符引用而不是不可编码的 Unicode
5. backslashreplace - 插入一个\uNNNNespace 序列而不是不可编码的 Unicode
6. namereplace - 插入\N{...}转义序列而不是不可编码的 Unicode

#### 返回值

字符串

#### 示例代码

```python
>>> str(1)
'1'
>>> str(3.14)
'3.14'
>>> str([1, 2, 3])
'[1, 2, 3]'
>>> str({'key': 'value'})
"{'key': 'value'}"
>>> str(b'2334')
"b'2334'"
```



### python内置函数sum详解

#### sum函数功能作用

内置函数sum从start开始从左向右对可迭代对象的项进行求和并返回总和，sum函数通常用于计算列表或元组的各项元素总和，start默认为0

#### sum函数语法

```python
sum(iterable, start=0)
```

#### 参数

- iterable 可迭代对象
- start 求和计算的起始索引

#### 返回值

可迭代对象各项元素和

#### 示例代码

```python
>>> sum([])
0
>>> sum([1, 3, 5])  
9
>>> sum((1, 2, 4, 5))
12
>>> sum(range(10))
45
```



### python内置函数super详解

#### super函数详解

内置函数super多用于面向对象编程super函数返回一个代理对象，将方法调用委托给当前类的父类，可用于调用父类已经实现的方法。

#### super函数语法

```python
super() -> same as super(__class__, <first argument>)
super(type) -> unbound super object
super(type, obj) -> bound super object; requires isinstance(obj, type)
super(type, type2) -> bound super object; requires issubclass(type2, type)
```

super函数有4种用法，我目前只参悟出了第3种用法，平时也只用到super(type, obj) 这种语法。

#### 示例代码

```python
class Animal():
    def __init__(self, name):
        self.name = name

    def drink(self):
        print(f'{self.name} is drinking')


class Cat(Animal):
    def __init__(self, name, age):
        super(Cat, self).__init__(name)
        self.age = age

    def drink(self):
        super(Cat, self).drink()
        print("猫喝水时很安静")


cat = Cat("小花", 3)
cat.drink()
```



## t开头的内置函数



### python内置函数tuple详解

#### tuple函数功能作用

虽然称tuple为函数，但它实质上是一个不可变序列类型，从表面上看，它能将可迭代对象转成元组，例如列表，range函数返回值。

#### tuple函数语法

```python
class tuple([iterable])
```

#### 参数

- iterable 可迭代对象

#### 返回值

元组

#### 示例代码

```python
>>> tuple([1, 2, 4])
(1, 2, 4)
>>> tuple(range(5))
(0, 1, 2, 3, 4)
```



### python内置函数type详解

#### type函数功能作用

内置函数type可以返回一个对象的类型，通常与 object.__class__ 所返回的对象相同。type是一个看起来不起眼但是功能和作用十分强大的函数，由于python的变量不需要声明类型，因此在编程时可以通过type函数获取变量的类型，这对于理解程序是十分关键的。

#### type函数语法

```python
class type(object)
class type(name, bases, dict, **kwds)
```

type函数不仅可以返回对象的类型，还可以构建新的类，这部分功能可以参考我的另一篇文章[《一切皆对象》](http://www.coolpython.net/python_senior/senior_feature/everything_is_object.html)。

#### 参数

- object 对象

#### 示例代码

```python
>>> type([1, 2])
<class 'list'>
>>> type('323')
<class 'str'>
>>> type(int)
<class 'type'>
>>> type(type)
<class 'type'>
```

通过type函数获得int的类型，竟然是type，type的类型也是type，这就是python的面向对象比较神奇也比较难理解的地方。



## v开头的内置函数



### python内置函数vars详解

#### vars函数功能作用

内置函数vars可以返回对象的__dict__ 属性，对象可以是模块，类，示例，只要对象有__dict__ 属性，vars就可以返回。

#### vars函数语法

```python
vars([object])
```

#### 参数

- object 有__dict__ 属性的对象

#### 返回值

对象的__dict__ 属性

#### 示例代码

```python
class Cat():
    def __init__(self, name, age):
        self.name = name
        self.age = age


cat = Cat('小花', 3)
print(vars(cat))            # {'name': '小花', 'age': 3}
```

示例的属性保存在实例的__dict__ 字典中，vars函数返回cat示例的__dict__属性。



## z开头的内置函数



### python内置函数zip详解

#### zip函数功能作用

内置函数zip可以在多个迭代器上并行迭代，从每一个迭代器上返回一个元素组成一个元组，如果迭代器的长度不统一，那么就以最短的那个为准。

#### zip函数语法

```python
zip(*iterables)
```

#### 参数

- iterables 可迭代对象

#### 返回值

可迭代对象

#### 示例代码

```python
lst1 = [1, 2, 3, 4, 5, 6]
lst2 = ['一', '二', '三', '四', '五']

info = {}
for item1, item2 in zip(lst1, lst2):
    info[item1] = item2

print(info)
```

zip函数传入的可迭代对象可以是多个，例子里传入两个，你可以传入更多，两个列表的长度不同，当lst2遍历结束时，zip函数也随之结束，因此字典info的大小与lst2的大小相同。
