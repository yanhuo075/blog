---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 12.2 实用代码示例
order: 1
---

# python for循环实现一个无限循环

使用while True可以轻松的实现一个无限循环，也叫死循环，那么for循环能够实现类似的功能么？当然可以，借助itertools模块的cycle函数就可以

```python
import time
from itertools import cycle

cycle_iter = cycle([100, 200, 300])
for item in cycle_iter:
    print(item)
    time.sleep(1)
```

这个for循环会一直进行下去，循环输出100， 200， 300。函数cycle()仅仅是为了实现无限循环么？当然不是， 它提供的是一种循环遍历的方法，下面的示例向你展示这个函数的实际应用

```python
lst1 = ['red', 'blue', 'white']
lst2 = ['A', 'B', 'C', 'D', 'E', 'F', 'G']
```

这里有两个列表，现在要求你将两个列表里的元素进行拼接，拼接时按照顺序一对一的拼接在一起，最后的结果如下

```text
red_A, blue_B, white_C, red_D, blue_E, white_F, red_G
```

如果使用普通的方法，实现这个功能还是有一点麻烦，但使用cycle()就变得非常简单

```python
from itertools import cycle

lst1 = ['red', 'blue', 'white']
lst2 = ['A', 'B', 'C', 'D', 'E', 'F', 'G']

lst = []
color_cycle = cycle(['red', 'blue', 'white'])

for item in lst2:
    color = next(color_cycle)
    lst.append('{color}_{char}'.format(color=color, char=item))

print(lst)
```



# python继承字典实现一个特殊的类

我想实现一个特殊的字典，当我向字典中插入一个新的key-value对时，字典内部自动帮我生成一个反向的key-value对，比如我写入'一': 1 ， 那么字典里同时也存在1: '一', 因为这种映射关系也是成立的，但我不想写入两次。

下面是我的实现

```python
class DoubleDict(dict):

    def __delitem__(self, key):     # 删除一个key, 也必须删除value
        value = super().pop(key)
        super().pop(value, None)

    def __setitem__(self, key, value):      # 先设置key-value, 再设置value-key
        super().__setitem__(key, value)
        super().__setitem__(value, key)

    def update(self, another):
        for key, value in another.items():
            self.__setitem__(key, value)

    def __repr__(self):
        return f"{type(self).__name__}({super().__repr__()})"

dic = DoubleDict()
print(dic)
dic['一'] = 1
dic['二'] = 2

print(dic['一'])
print(dic[2])

dic.update({'三': 3})
print(dic[3])
```

设计一个类，继承dict, 然后重载__delitem__ , __setitem__, update，__repr__方法

1. __delitem__ 是删除给定键对应的值
2. __setitem__ 设置给定键的值
3. update是更新字典的方法
4. __repr__ 是显示对象信息的方法，由于字典的该方法显示的是字典的内容，因此我进行了重载，让其显示出类的名字



# python实现安全的int, float函数

类型转换总是会发生错误，尤其是当你的数据是由其他人通过API接口提供时，虽然文档上约定这个字段是int类型，但实际传给你时可能已经变成一个空的字符串。这种不按文档严格传输数据的情况时有发生，如果你对其他人传入的数据不信任而要亲自做类型转换的话，那么就得考虑编写一个安全的int函数和float函数。

为什么要编写专门的转换函数呢？简单说，我不希望在代码里看到一大堆的try ... except语句，毕竟这种类型转换几乎无处不在，所以，我需要一个专门的函数，为此我设计了如下两个函数

```python
import logging


logger = logging.getLogger('my_log')
logger.setLevel(logging.INFO)


def safe_int(value, default_return=0, is_log=False):
    '''
    提供安全的类型转换函数,将value转换为int类型
    :param value:需要被转换的数据
    :param default_return: 转换失败时的返回值
    :param is_log:是否记录日志
    :return:
    '''
    pass


def safe_float(value, default_return=0, is_log=False):
        '''
    提供安全的类型转换函数,将value转换为float类型
    :param value:需要被转换的数据
    :param default_return: 转换失败时的返回值
    :param is_log:是否记录日志
    :return:
    '''
```

我的目标是设计实现两个安全的类型转换函数，我可以定义转换失败时的返回值，如果我想把失败原因记录下来，只需要将is_log设置为True。这两个函数看起来功能几乎相同，因此还需要一个公共的函数来完成这些操作

```python
def super_transfer(_type, value, default_return=0, is_log=False):
    try:
        result = _type(value)
    except:
        result = default_return
        if is_log:
            logger.error('将{value}转换为{_type}类型时失败'.format(value=value, _type=_type))

    return result

def safe_int(value, default_return=0, is_log=False):
    '''
    提供安全的类型转换函数,将value转换为int类型
    :param value:需要被转换的数据
    :param default_return: 转换失败时的返回值
    :param is_log:是否记录日志
    :return:
    '''
    return super_transfer(int, value, default_return, is_log)


def safe_float(value, default_return=0, is_log=False):
    '''
    提供安全的类型转换函数,将value转换为float类型
    :param value:需要被转换的数据
    :param default_return: 转换失败时的返回值
    :param is_log:是否记录日志
    :return:
    '''

    return super_transfer(float, value, default_return, is_log)
```

现在，来测试一下效果吧

```python
def test():
    print(safe_float(2))
    print(safe_float('3'))
    print(safe_float('3f', is_log=True))

    print(safe_int(98.8))
    print(safe_int('8i', is_log=True))

if __name__ == '__main__':
    test()
```

程序输出结果

```text
2.0
3.0
0
98
0
将3f转换为<class 'float'>类型时失败
将8i转换为<class 'int'>类型时失败
```

转换失败时，函数返回了默认值，同时在日志中记录了失败的原因，当然我写的这个日志对象很简单，你在实践中应当使用功能更加完善的log对象



# python利用生成器终止两层for循环

两层循环过程中，如果想通过break终止循环，是一件简单，但却很麻烦的事情,例如下面的这段代码

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]

stop = False
for i in lst1:
    for j in lst2:
        if i + j == 10:
            stop = True
            break
    
    if stop:
        break
```

两层for循环的目的非常简单，从两个列表中各找出一个数，使得他们的和等于10，而且只需找出一个组合即可。

找到满足要求的组合后，为避免不必要的循环，需要终止循环，而此时，if条件语句在for循环的最里层，此处执行break，只能跳出最里层的for循环，想要终止最外层的for循环，就必须传递终止信号给它，代码里，通过stop=True,告知外层for循环可以终止了。

这样的代码写起来，显然有些繁琐，最后的if stop判断总显得多余，面对这种情况，可以巧妙的利用生成器来避免这种复杂的写法。

```python
lst1 = [1, 2, 4, 5, 6]
lst2 = [1, 5, 6, 8, 9]


def num_generator(lst1, lst2):
    for i in lst1:
        for j in lst2:
            yield i, j


for i, j in num_generator(lst1, lst2):
    if i + j == 10:
        print(i, j)
        break
```

生成器num_generator里通过两层for循环对数据进行遍历，真正的业务逻辑使用一个for循环，这里就避免了跳出两层for循环的困境，这一次break，结束了for循环对num_generator的使用。



# python用一行代码判断列表里的元素都相同

给你一个列表

```python
lst = [1, 1, 1]
```

能你想出几种办法只用一行代码就能判断列表里的元素都是相同的呢？

以下是我的答案

```python
# 利用集合
print(len(set(lst)) == 1)
# 利用all函数
print(all(item == lst[0] for item in lst))
# 利用列表count方法
print(lst.count(lst[0]) == len(lst))
```



# python如何遍历文件夹及其子文件夹

## 1. os.listdir()

使用os.listdir方法可以简单快捷的列出一个文件夹里的所有文件，包括文件目录

```python
import os

lst = os.listdir('.')
print(lst)
```

如果想递归遍历，所有子文件夹也要遍历，os.listdir方法就无能为力了

## 2. os.walk()

os.walk()可以遍历文件夹中的所有子文件夹及子文件, walk函数返回一个元组(dirpath, dirnames, filenames)

1. dirpath是文件夹路径
2. dirnames是文件夹名称
3. filenames是文件名称

### 2.1 输出所有文件

```python
import os

for dirpath, dirnames, filenames in os.walk('.'):
    for filename in filenames:
        print(os.path.join(dirpath, filename))
```

### 2.2 输出所有文件夹

```python
import os

for dirpath, dirnames, filenames in os.walk('.'):
    for dirname in dirnames:
        print(os.path.join(dirpath, dirname))
```



# python 循环使用for ... else 语句

for ... else是一种python特有的语法，当for循环正常结束时会进入到else语句块中执行代码，如果for循环是因为执行了break语句导致提前结束的，则不会进入到else语句块

下面是一个使用示例

```python
lst = [3, 6, 3, 6, 9, 10, 20]
tag = True

count = 0
for item in lst:
    if item % 2 == 0:
        count += 1

    if count > 3:
        break
else:
    tag = False

print(tag)
```

tag 为True, 表示列表里偶数的数量超过3个，如果for循环正常结束，没有遇到break语句，则执行else语句块，将tag设置为False



# python 异常处理中使用try ... else语句

使用try ....except...else 语句时，如果没有异常发生时，else中的语句将会被执行，这个特点以让我们根据是否发生了异常来做一些特殊的处理

```python
value = input('请输入一个整数')

b_except = True
try:
    value = int(value)
except:
    value = 0
else:
    b_except = False
    print('没有发生异常')

if value == 0:
    if not b_except:
        print(0)
    else:
        print('输入有异常')
else:
    print(value)
```

当输入的数据有问题时，会引发异常，在except语句中将value设置为0，但是请考虑，用户也可能正常输入0，因此需要一个标识位来标识是否引发了异常导致value为0，这种情况下，使用try ... else就很方便。



# python文件大小，创建日期， 修改日期， 最后访问日期

python 标准模块os下的path模块，提供了获取文件大小，创建日期，修改日期，最后访问日期的函数

```python
import os, datetime

filename = '/Users/kwsy/kwsy/coolpython/demo.py'
# 文件大小
filesize = os.path.getsize(filename)
print(filesize)     # 字节数

# 文件创建日期
create_time = os.path.getctime(filename)        # 返回的是时间戳
print(datetime.datetime.fromtimestamp(create_time))

# 文件最近访问时间
access_time = os.path.getatime(filename)
print(access_time)

# 文件最近修改时间
modify_time = os.path.getmtime(filename)
print(modify_time)
```



# python判断ip地址是否合法

ip地址由4段整数构成，每段范围从0到255，本文研究如何判断一个ip地址是否合法

## 1. 使用socket.getaddrinfo

socket.getaddrinfo方法的原型是

```python
def getaddrinfo(host, port, family=0, type=0, proto=0, flags=0):
```

该方法返回由元组组成的列表,下面的代码使用该方法获取百度域名的信息

```python
import socket
print(socket.getaddrinfo('www.baidu.com', None))
```

程序输出结果

```text
[(2, 1, 6, '', ('220.181.38.150', 0)), (2, 2, 17, '', ('220.181.38.150', 0)), (2, 3, 0, '', ('220.181.38.150', 0)), (2, 1, 6, '', ('220.181.38.149', 0)), (2, 2, 17, '', ('220.181.38.149', 0)), (2, 3, 0, '', ('220.181.38.149', 0))]
```

host既可以是域名，也可以是ip地址，如果方法能正常返回列表，则说明地址合法，反之地址不合法,在tornado源码里，有一个is_valid_ip函数就是利用该放方法进行判断

```python
import socket

def is_valid_ip(ip):
    """Returns true if the given string is a well-formed IP address.
    Supports IPv4 and IPv6.
    """
    if not ip or '\x00' in ip:
        # getaddrinfo resolves empty strings to localhost, and truncates
        # on zero bytes.
        return False
    try:
        res = socket.getaddrinfo(ip, 0, socket.AF_UNSPEC,
                                 socket.SOCK_STREAM,
                                 0, socket.AI_NUMERICHOST)
        return bool(res)
    except socket.gaierror as e:
        if e.args[0] == socket.EAI_NONAME:
            return False
        raise

print(is_valid_ip('192.168.0.1'))
print(is_valid_ip('1.1.1'))
```

令人感到诧异的是，1.1.1被识别为正常的ip，原来它会被理解为1.1.0.1。

## 2. 利用ip转整数

一个ip地址可以转换为int类型数据存储，这也是很多系统里喜欢使用的方法，相比于存储一个int类型数据，存储字符串ip占用的空间更多

```python
import socket
import struct

ip = '127.0.0.1'
# 字符串ip转int类型
int_ip = socket.ntohl(struct.unpack("I",socket.inet_aton(str(ip)))[0])
print(int_ip)
# int类型ip转字符串ip
str_ip = socket.inet_ntoa(struct.pack('I',socket.htonl(int_ip)))
print(str_ip)
```

程序输出结果

```text
2130706433
127.0.0.1
```

如果字符串ip是非法的，那么在转int类型ip时一定会发生异常，只需要捕捉到这个异常就可以断定ip是非法的

```python
import socket
import struct

def is_valid_ip(ip):
    try:
        socket.ntohl(struct.unpack("I",socket.inet_aton(str(ip)))[0])
    except OSError:
        return False
    return True

print(is_valid_ip('192.168.0.1'))       # True
print(is_valid_ip('192.168.333.1'))  # False
```



# 用python判断ip是私有地址

判断一个IP地址是私有地址，其原理是A，B,C三类私有地址有明确的范围，他们有各自的前缀，A类地址有8位前缀，B类地址有12位前缀， C类地址有16位前缀。本文所指的IP地址，皆是IPV4。

一个IPV4地址，由四段组成，最大值为255，一个IP地址其实就是一个32位的bit串，每8位一段。所谓私有地址，就是非注册地址，只能做内网地址。私有地址有三类，分别是

1. A类 10.0.0.0 --10.255.255.255 　　
2. B类 172.16.0.0--172.31.255.255
3. C类192.168.0.0--192.168.255.255

划分规则如下：

1. A类有一个8位的前缀
2. B类的前缀是12位
3. C类的前缀是16位

啥意思呢。就拿A类地址来说，前八位已经定下来了，就是10，后面的22位，随意变化，那么范围自然就是10.0.0.0到10.255.255.255了，B类地址前12位已经定下来了，第一个八位定为172，第二个八位的前4位定为16，他们的二进制表示是这样的

- 第一段   10101100
- 第二段   00010000

第一段的8位加上第二段的前4位，这12位是固定下来的。那么后面的20位就是可以随便变化的，而第二段的后4位即便都是1，那么第二段的最大值也只能是00011111，也就是31，所以B类地址的第二段最大值就是31

- C类地址的前16位都固定了，就是192.168

明白了私有地址的范围，我们也就好计算了。一种方法，就是根据规则对字符串进行比较，用中间的点做分割，然后逐一对每段的值进行判断，然而本文要说的是另一个思路。

我们可以把一个IP地址转成二进制，然后呢，每一类的地址的前面8,12，或16位其实是固定的，只要前面的这些位对得上，那么这个地址就是私有地址，程序用python编写，代码如下：

```python
import socket
import struct

ip1 = 167772160
ip2 = 2886729728
ip3 = 3232235520


def isPrivateIp(ip):
    #将ip地址转换成二进制的形式
    binaryIp = socket.inet_aton(ip)
    #将二进制转成无符号long型
    numIp = struct.unpack('!L', binaryIp)[0]
    #32位都是1
    mark = 2**32-1
    #取numIP的前16位
    tag = (mark<<16) & numIp
    if ip3 ==tag:
        return True
    #取numIP的前12位
    tag = mark<<20 & numIp
    if ip2 == tag:
        return True
    #取numIP的前8位
    tag = (mark<<24) & numIp
    if ip1 == tag:
        return True
    return False

if __name__=='__main__':
    print(isPrivateIp('192.168.0.1'))
```



# 判断当前python版本

python的sys模块提供了许多有用的功能，这其中就包括获得python版本，sys.version_info可以获得python版本的详细信息，major是大版本号,如果major等于3，则说明是python3， 若major是2，则说明是python2

```python
import sys

print(sys.version_info)
version = sys.version_info.major    # 大版本号
print(version)
```

程序输出结果

```text
sys.version_info(major=3, minor=6, micro=6, releaselevel='final', serial=0)
3
```

我使用的python版本是3.6.6



# python将两个列表转换成一个字典

本文讨论如何将两个列表转换成一个字典，要求两个列表满足以下条件

1. 列表长度相同
2. 做字典key的列表里没有重复元素

```text
输入:
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

输出:
{
    'python': 90,
    'java': 99,
    'php': 95
}
```

### 方法1， 遍历列表创建字典

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = {}
for index, key in enumerate(name_lst):
    dic[key] = value_lst[index]

print(dic)
```

遍历大法好，用了忘不了，不管是什么问题，只要能遍历，就没有解决不了的

### 方法2， 结合zip和字典构造函数

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = dict(zip(name_lst, value_lst))
print(dic)
```

dict究竟是如何起作用的呢，可以参考下面的代码

```python
print(dict([('python', 90), ('java', 99)]))
```

列表里有连个元组，dict函数会将这个两个元组转换成key-value对，最终的结果是

```python
{'python': 90, 'java': 99}
```

### 方法3， 字典推导式

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = {key: value for key, value in zip(name_lst, value_lst)}
print(dic)
```



# python判断一个文件是否存在

本文旨在讨论在python当中判断一个文件是否存在的可行的方法，实践中，我们选择那个最简单的方法即可，现在的讨论更多的是一种技术探索

### 方法1， os.path.exists

```python
import os

print(os.path.exists('log.txt'))
```

这是我最喜欢使用的一种方法，它不仅能够判断一个文件是否存在，还能判断一个文件夹是否存在

### 方法2， os.path.isfile

```python
import os

print(os.path.isfile('log.txt'))
```

isfile方法可以判断所给的path是不是文件，如果是，就证明文件存在，反之，文件不存在

### 方法3， 尝试以读模式打开文件

```python
try: 
  with open('log.txt', 'r') as fh:
    pass
except FileNotFoundError: 
  print('不存在')
```

如果文件不存在，则会抛出FileNotFoundError异常，通过except捕捉该异常就可以判断文件是否存在，打开文件使用了with关键字，因此不需要主动关闭文件

### 方法4， 使用pathlib

```python
from pathlib import Path

config = Path('log.txt')
print(config.is_file())
```

原理同os.path.isfile一样



# 创建任意大小文件

不积跬步无以至千里，每天积累一点python知识，量变终究引起质变。

可能存在这样的应用要求：创建任意大小的文件。为了实现这个要求，你需要使用二进制的方式打开文件，并根据大小要求偏移文件指针

```python
def createfile(filename,size):
    with open(filename, 'wb') as f:
        f.seek(size-1)
        f.write(b'\x00')


if __name__ == '__main__':
    createfile('test.db', 53421)
```

打开文件后，进行偏移，然后写入一个字节的内容，最后的这个字节一定要写入，否则文件不会如期望的那样大。最大可以创建多大的文件呢？这个取决于操作系统的位数和文件系统的格式，这部分内容我没有考证过，也不敢乱说，在windows 64位环境下，如果文件大小超过了4G，那么open函数是无法以非二进制的方式打开的，此时一定要用二进制的方式打开。



# python翻转字典

翻转列表是一种常见的操作，很少要求你翻转字典，但实际工作中，这种情况并不是不存在，原字典内容是

```python
dic = {
    'python': 90,
    'java': 99,
    'php': 95
}
```

翻转以后的字典内容是

```python
dic = {
    90: 'python', 
    99: 'java', 
    95: 'php'
}
```

这里，我们只考虑value没有重复值且可hash的情况，接下来，我将尝试使用使用3种方法实现这个操作

### 方法1， 遍历字典

```python
inverted_dict = {}
for key, value in dic.items():
    inverted_dict[value] = key

print(inverted_dict)
```

这是最容易想到的一种方案，代码实现起来最普通

### 方法2， 字典推导式

```python
# 使用字典推导式
inverted_dict = {value: key for key, value in dic.items()}
print(inverted_dict)
```

使用字典推导式，让代码变得更加简单了

### 方法3， 结合函数map, reversed

```python
inverted_dict = dict(map(reversed, dic.items()))
print(inverted_dict)
```

方法3同样只需要一行代码就可以完成翻转，但理解起来相比于推导式稍有难度，reversed函数负责将key-value对翻转，map函数负责处理所有的key-value对，dict负责将map的结果转成字典



# 如何使用python 新建文件夹以及递归创建文件夹

## 1. os.mkdir

使用python创建文件夹，通常使用os.mkdir方法，在使用这个方法时有几个小的细节需要注意，假设你的代码是这样编写的

```python
import os

os.mkdir('/dir_1/dir_2/dir_3')
```

你需要保证/dir_1/dir_2 是存在的，否则将引发FileNotFoundError，如果/dir_1/dir_2/dir_3 已经存在，又会引发FileExistsError，通常，我会使用os.path.exists方法判断关键的目录是否已经存在，来决定是否新建文件夹。

## 2. os.makedirs

os.makedirs 可以视为os.mkdir的升级版本，它以递归的方式创建文件夹，如果dir_1不存在，就先创建dir_1，而后递归创建剩余的文件夹，这样就不存在FileNotFoundError；如果想要创建的目录已经存在，也没有关系，设置exist_ok = True， 就不会引发FileExistsError

```python
import os

os.makedirs('./1/2/3/4/5', exist_ok=True)
```

这两行代码你可以执行多次，不会有任何错误或异常



# python将两个列表转换成一个字典

本文讨论如何将两个列表转换成一个字典，要求两个列表满足以下条件

1. 列表长度相同
2. 做字典key的列表里没有重复元素

```text
输入:
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

输出:
{
    'python': 90,
    'java': 99,
    'php': 95
}
```

### 方法1， 遍历列表创建字典

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = {}
for index, key in enumerate(name_lst):
    dic[key] = value_lst[index]

print(dic)
```

遍历大法好，用了忘不了，不管是什么问题，只要能遍历，就没有解决不了的

### 方法2， 结合zip和字典构造函数

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = dict(zip(name_lst, value_lst))
print(dic)
```

dict究竟是如何起作用的呢，可以参考下面的代码

```python
print(dict([('python', 90), ('java', 99)]))
```

列表里有连个元组，dict函数会将这个两个元组转换成key-value对，最终的结果是

```python
{'python': 90, 'java': 99}
```

### 方法3， 字典推导式

```python
name_lst = ['python', 'java', 'php']
value_lst = [90, 99, 95]

dic = {key: value for key, value in zip(name_lst, value_lst)}
print(dic)
```



# python两个列表相加

本文讨论两个列表如何相加在一起，要求这两个列表满足以下要求

1. 列表长度相同
2. 列表里的元素都是int类型数据
3. 对应索引位置元素相加，生成新的列表

示例如下

```text
输入:
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

输出:
sum_lst = [3, 9, 10]
```

### 方法1， 遍历相加

```python
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

sum_lst = []
for index, item in enumerate(lst1):
    sum_lst.append(item + lst2[index])

print(sum_lst)
```

能想到这样的算法，就算是入门合格了，掌握了python基础技术，懂得如何遍历列表，如何向列表里新增元素

### 方法2， 使用列表推导式

使用列表推导式，并结合zip函数

```python
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

sum_lst = [x + y for x, y in zip(lst1, lst2)]
print(sum_lst)
```

方法2的核心在于正确使用zip函数，zip函数将多个列表视为一个整体，遍历时，同时从各个列表里取出相同索引位置的元素

### 方法3， 使用map

```python
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

import operator
sum_lst = list(map(operator.add, lst1, lst2))
print(sum_lst)
```

map函数从lst1和lst2中各取出一个数据然后使用operator.add相加, operator.add也可以替换成lambda表达式

```python
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

sum_lst = list(map(lambda x, y: x + y, lst1, lst2))
print(sum_lst)
```

### 方法4， 使用numpy

```python
lst1 = [1, 4, 7]
lst2 = [2, 5, 3]

import numpy as np
sum_lst = list(np.add(lst1, lst2))
print(sum_lst)
```



# python判断列表是空列表

本文将讨论如何判断一个列表是否为空列表，思路有3个

1. 判断列表长度是否等于0
2. 判断列表是否等于[]
3. 使用not

示例代码

```python
lst = list()

print(len(lst) == 0)       # 长度
print(lst == [])           # 等于空列表
print(not lst)             # 使用not 构成表达式
```

输出结果为

```text
True
True
True
```

我在工作中最喜欢使用的是第3种方法

```python
lst = list()

if not lst:
    print('空列表')
```

不只是空列表，空字典，空集合都可以使用not来进行判断
