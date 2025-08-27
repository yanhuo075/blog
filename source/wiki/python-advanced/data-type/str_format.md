---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 1.3 字符串格式化高级用法
order: 2
---

# 字符串格式化高级用法

字符串的格式化主要有三种方式，分别是使用 %格式化，format方法格式化，fstring格式化。

## 1. %操作符

```python
str_format = "I'm %s. I'm %d years old"
string = str_format % ('lilei', 14)
print(string)
```

str_format是模板，在模板里有格式符，示例中有两个格式符，分别是%s, %d。这些格式符为真实值预留了位置，并说明了真实值应该呈现的格式，%s表示字符串，%d表示整数，第二行 使用 % 进行格式化时，后面紧跟一个元组，这个元组将多个值传递给模板，每个值依次对应到模板里的一个格式符。上面的例子输出的结果是

```text
I'm lilei. I'm 14 years old
```

### 1.1 格式符

格式符有很多种，你需要根据实际需要来选择模板里的格式符

%s 字符串 (采用str()的显示)

%r 字符串 (采用repr()的显示)

%c 单个字符

%b 二进制整数

%d 十进制整数

%i 十进制整数

%o 八进制整数

%x 十六进制整数

%e 指数 (基底写为e)

%E 指数 (基底写为E)

%f 浮点数

%F 浮点数，与上相同

%g 指数(e)或浮点数 (根据显示长度)

%G 指数(E)或浮点数 (根据显示长度)

%% 字符"%"

## 2. format

我个人十分喜欢使用format方法对字符串进行格式化操作，因为相比于使用%操作符，format方法更加直观和简单，尤其是在格式化时与字典，元组，对象相结合的情况下，简直爽歪了。

### 2.1 format一般用法

```python
log = "{id}-{type}-{msg}"
print(log.format(id=1,type='debug',msg=u'测试连接'))
```

同样需要一个格式化模板，与使用%操作符格式化不同的时，使用format方法进行格式化时，使用{}来为真实值预留位置，大括号内你可以随意编写变量名称，然后在format方法里为这些变量赋予真实值。

### 2.2 与字典结合的用法

如果真实值存储在一个字典里，且模板里预留了好多位置需要填充真实值，那我真的不想在format方法里为那么多参数赋值

```python
info = {
    'id':1,
    'type':'debug',
    'msg':u'测试连接'
}
log = "{0[id]}-{0[type]}-{0[msg]}".format(info)
print(log)

log = "{info[id]}-{info[type]}-{info[msg]}".format(info=info)
print(log)
```

这两种格式化的方式都可行，我个人推荐使用第二种。

### 2.3 与tuple结合的方法

```python
info = (1,'debug',u'测试连接')
log = "{0[0]}-{0[1]}-{0[2]}".format(info)
print(log)

log = "{info[0]}-{info[1]}-{info[2]}".format(info=info)
print(log)
```

### 2.4 与对象结合的方法

除了字典和元组，还可以使用对象

```python
class Info(object):
    def __init__(self, id, type, msg):
        self.id = id
        self.type = type
        self.msg = msg

info = Info(1,'debug',u'测试连接')
log = "{info.id}-{info.type}-{info.msg}".format(info=info)
print(log)
```

### 2.5 不指定关键字参数

使用format方法时，如果为了方便，也可以不指定关键字参数，而是依据参数顺序格式化字符

```python
log = "{}-{}-{}"
print(log.format(1, 'debug', '测试连接'))
```

format方法里的参数会依次填入字符串中的{}中

### 2.6 控制宽度

在格式化字符串时，有时需要控制一下宽度，比如下面这段打印九九乘法表的代码

```python
for i in range(1, 10):
    line = ""
    for j in range(1, i+1):
        line += "{left} * {right} = {product} ".format(left=i, right=j, product=i*j)

    print(line)
```

输出结果

```python
1 * 1 = 1
2 * 1 = 2  2 * 2 = 4
3 * 1 = 3  3 * 2 = 6  3 * 3 = 9
4 * 1 = 4  4 * 2 = 8  4 * 3 = 12  4 * 4 = 16
5 * 1 = 5  5 * 2 = 10  5 * 3 = 15  5 * 4 = 20  5 * 5 = 25
6 * 1 = 6  6 * 2 = 12  6 * 3 = 18  6 * 4 = 24  6 * 5 = 30  6 * 6 = 36
7 * 1 = 7  7 * 2 = 14  7 * 3 = 21  7 * 4 = 28  7 * 5 = 35  7 * 6 = 42  7 * 7 = 49
8 * 1 = 8  8 * 2 = 16  8 * 3 = 24  8 * 4 = 32  8 * 5 = 40  8 * 6 = 48  8 * 7 = 56  8 * 8 = 64
9 * 1 = 9  9 * 2 = 18  9 * 3 = 27  9 * 4 = 36  9 * 5 = 45  9 * 6 = 54  9 * 7 = 63  9 * 8 = 72  9 * 9 = 81
```

注意看第3列，明显没有对齐，原因在于第2列的乘积出现了大于10的情况，挤占了空间，为了能更美观的显示效果，我们需要控制格式化字符串时的宽度，只需要将待格式化的字符串稍稍做一下修改即可

```python
line += "{left} * {right} = {product:3}  ".format(left=i, right=j, product=i*j)
```

我指定product被格式化以后站3个字符的宽度，这样，不论乘积是否大于10，都占3个字符宽度

```python
1 * 1 =   1
2 * 1 =   2  2 * 2 =   4
3 * 1 =   3  3 * 2 =   6  3 * 3 =   9
4 * 1 =   4  4 * 2 =   8  4 * 3 =  12  4 * 4 =  16
5 * 1 =   5  5 * 2 =  10  5 * 3 =  15  5 * 4 =  20  5 * 5 =  25
6 * 1 =   6  6 * 2 =  12  6 * 3 =  18  6 * 4 =  24  6 * 5 =  30  6 * 6 =  36
7 * 1 =   7  7 * 2 =  14  7 * 3 =  21  7 * 4 =  28  7 * 5 =  35  7 * 6 =  42  7 * 7 =  49
8 * 1 =   8  8 * 2 =  16  8 * 3 =  24  8 * 4 =  32  8 * 5 =  40  8 * 6 =  48  8 * 7 =  56  8 * 8 =  64
9 * 1 =   9  9 * 2 =  18  9 * 3 =  27  9 * 4 =  36  9 * 5 =  45  9 * 6 =  54  9 * 7 =  63  9 * 8 =  72  9 * 9 =  81
```

## 3. f-string

### 3.1 f-string用法

f-string，亦称为格式化字符串常量（formatted string literals），是Python3.6新引入的一种字符串格式化方法。f-string让字符串的格式化更加简便，本质上f-string不是字符串常量，而是一个可以在运行时运算求值的表达式。

```python
color = 'red'
string = f"I like {color}"
print(string)
```

从形式看，f-string是以f或F装饰引领的字符串，内部使用大括号为真实值预留位置，那么大括号所标注的需要被替换的字段填充什么值呢？

填充什么值，要看f-string所在的作用域里与大括号内相对应的变量值。在本示例中，大括号里的字段名称是color，而当前所在作用域里恰好有一个变量的名称叫做color，那么字符串格式化时就会填充变量color的值，使用f-string的优势就在于它省略了这个指明填充值的过程。不仅如此，f-string还提供了更加强大的功能

```python
color = 'red'
string = f"I like {color.upper()}"
print(string)
```

大括号里甚至可以传入表达式和函数，在这个示例中，真实值是color变量转大写的结果。
