---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 7.7 序列化
order: 6
---

# python如何序列化数据

python 内置的pickle 和 json模块均可以实现对数据的序列化与反序列化，pickle 实现的是协议是二进制的，所以序列化以后的内容我们无法直接阅读，而json 序列后的数据仍具有可读性

## 1 什么是序列化

将数据转换为可以通过网络传输或者可以存储到本地磁盘的数据格式（xml,json或特定格式字符串）的过程称之为序列化，反之，称之为反序列化。

不同编程语言有各自的数据类型，同样是int类型，不同的编程语言里，其存储方式是不一样的。那么不同编程语言编写的系统如果想进行通信，就必须使用一种双方都认可的数据格式传输数据，当前，最为流行的就是json数据格式。

## 2 pickle

pickle是python的原生模块，它实现了对python对象的序列化和反序列化的二进制协议，由于采用的是二进制协议，因此，pickle序列化后的内容，是无法像json序列化后的内容一样正常查看的。

### 2.1 序列化示例

```python
import pickle


data = {
    'name': 'lilei',
    'age': 18
}

# 序列化到文本中
f = open('person', 'wb')
pickle.dump(data, f)
f.close()
```

虽然，pickle也提供了dumps方法，但返回的是byte类型数据，而不是字符串。经过实际测试，使用str函数将byte类型数据转成字符串的过程，可能会出错,这些byte数据不是从字符串转而来的，个别位置无法用utf-8进行编码，因此为了避免这类问题，不推荐你使用dumps方法。

```python
import pickle


data = {
    'name': 'lilei',
    'age': 18
}

byte_str = pickle.dumps(data)
print(byte_str)
string = str(byte_str, encoding='utf8')
```

### 2.2 pickle反序列化

```python
import pickle

f = open('person', 'rb')
data = pickle.load(f)
f.close()

print(data)
```

程序运行结果

```text
{'name': 'lilei', 'age': 18}
```

### 2.3 序列化对象

pickle不只能对标准数据类型进行序列化，就连用户自定义的对象也可以进行序列化

```python
import pickle

class Person(object):
    def __init__(self):
        self.name = 'lilei'
        self.age = 18


person = Person()
# 序列化
f = open('person', 'wb')
pickle.dump(person, f)
f.close()

# 反序列化
f = open('person', 'rb')
data = pickle.load(f)
f.close()

print(type(data))
print(data.name)
print(data.age)
```

程序输出结果

```text
<class '__main__.Person'>
lilei
18
```

这里有一个地方需要特别注意，使用pickle反序列化用户自定义对象时，必须保证可以访问到用户自定义的类，这样，pickle才能构造出这个类的对象。

## 3 json 模块

json是非常通用，非常流行的数据格式，几乎被所有语言所接纳，使用json模块序列化后的数据就是json格式，仍然可以被人们直接阅读。
下表，是python数据类型和json数据类型之间的映射关系

| python      | json   |
| ----------- | ------ |
| dict        | object |
| list, tuple | array  |
| str         | string |
| int, float  | number |
| True        | true   |
| False       | false  |
| None        | null   |

### 3.1 json序列化示例

```python
import json

data = {
    'name': 'lilei',
    'age': 18
}

f = open('person', 'w')
json.dump(data, f)
f.close()

string = json.dumps(data)
print(string)
```

不论是转成字符串，还是序列化存储到文件中，其内容都是可以看得懂的

```text
{"name": "lilei", "age": 18}
```

### 3.2 json反序列化

```python
import json

data = '{"name": "lilei", "age": 18}'
data = json.loads(data)
print(data)

with open('person', 'r') as f:
    data = json.load(f)
    print(data)
```

程序输出结果

```python
{'name': 'lilei', 'age': 18}
{'name': 'lilei', 'age': 18}
```

## 4 msgpack

msgpack 是一种和json很相似，但是速度更快，体积更小的二进制序列化数据格式，这是它的官方 https://msgpack.org/

想要使用这种数据格式进行序列化，需要安装msgpack-python模块

```text
pip3 install msgpack-python
```

### 4.1 使用示例

```python
    'name': 'lili',
    'age': 18,
    'score': 100
}

# 和json进行序列化数据大小对比
msg_str = msgpack.packb(stu)
print(msg_str)
print(len(msg_str))

json_str = json.dumps(stu)
json_str = bytes(json_str, encoding = "utf8")
print(json_str)
print(len(json_str))

# 反序列化
data = msgpack.unpackb(msg_str)
print(data)
```

程序输出结果

```python
b'\x83\xa4name\xa4lili\xa3age\x12\xa5scored'
23
b'{"name": "lili", "age": 18, "score": 100}'
41
{b'name': b'lili', b'age': 18, b'score': 100}
```

可以看到，msgpack序列化后的数据，其体积比json序列化后的要小很多，这样，在网络传输上就会更快。

### 4.2 json与msgpack性能对比

#### 4.2.1 体积对比

```python
import msgpack
import json

data = {
    'count': 100,
    'price': 29832.09,
    'desc': 'MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it’s faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves',
    'desc_cn': "MessagePack  是一个高效的二进制序列化格式。它让你像JSON一样可以在各种语言之间交换数据。但是它比JSON更快、更小。小的整数会被编码成一个字节，短的字符串仅仅只需要比它的长度多一字节的大小",
    'bool': True
}

data = [data for i in range(10)]
json_str = json.dumps(data)
print(len(json_str))

msg_str = msgpack.dumps(data)
print(len(msg_str))
```

程序执行结果

```text
8740
6001
```

msgpack节省了30%的存储空间

#### 4.2.2 序列化速度比较

```python
import msgpack
import json
import time

data = {
    'count': 100,
    'price': 29832.09,
    'desc': 'MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it’s faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves',
    'desc_cn': "MessagePack  是一个高效的二进制序列化格式。它让你像JSON一样可以在各种语言之间交换数据。但是它比JSON更快、更小。小的整数会被编码成一个字节，短的字符串仅仅只需要比它的长度多一字节的大小",
    'bool': True
}


data = [data for i in range(10)]
t1 = time.time()
for i in range(100000):
    json_str = json.dumps(data)

t2 = time.time()

for i in range(100000):
    msg_str = msgpack.dumps(data)
t3 = time.time()

print(t2-t1)
print(t3-t2)
```

程序执行结果

```text
6.320836067199707
0.9859249591827393
```

速度上，msgpack快了6倍，更快的速度，更小的体积。
