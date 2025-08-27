---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 9.3 封包与解包
order: 2
---

# python的封包与解包

## 1. python 封包

将多个值赋值给一个变量时，python会自动将这些值封装成元组，这个特性称之为封包

```python
a = 1, 2, 3
print(a, type(a))  # (1, 2, 3) <class 'tuple'>
```

当函数返回多个数值时，也会进行封包

```python
def test():
    return 1, 2, 3

a = test()
print(a, type(a))  # (1, 2, 3) <class 'tuple'>
```

实践中，很少主动使用封包操作

## 2. python解包

python解包可以将元组解包成可变参数，将字典解包成关键字参数，下面列列举几种使用python解包的场景

### 2.1 接收函数返回值

```python
def test():
    return 1, 2, 3

a, b, c = test()
print(a, b, c)      # 1 2 3
```

函数的返回值是一个元组，左侧是三个变量，这样就会发生解包，a, b, c依次等于元组里的元素，函数的返回值有3个，被封包成了元组， 赋值语句的左侧不一定非得是3个变量

```python
def test():
    return 1, 2, 3

a, *b = test()
print(a, b)     # 1 [2, 3]
```

变量a赋值为1， 变量b前面有一个星号，剩余的2， 3 将被解包为列表

### 2.2 遍历字典

```python
my_dic = {
    '一': 1,
    '二': 2,
    '三': 3
}

for item in my_dic.items():
    print(item)

# 解包
for key, value in my_dic.items():
    print(key, value)
```

### 2.3 传递参数

```python
def func(*args):
    print(sum(args))


a = (2, 4, 6)
func(*a)    # 将元组解包成可变参数

def func_2(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

b = {'一': 1, '二': 2}
func_2(**b)     # 将字典解包成关键字参数
```

解包技术在实践中大量应用，比如使用python操作redis时，如果你想一次性向集合中添加多个值，就必须使用解包结束传入参数

```python
import redis
from conf.redis_conf import RedisConfig, QueueConfig

r = redis.Redis(host=RedisConfig.host, port=RedisConfig.port,
                password=RedisConfig.password, db=RedisConfig.db)

tup = ('apple', '谷歌', '阿里', '腾讯')

r.sadd('my_set', *tup)
```

sadd的方法定义如下

```python
def sadd(self, name, *values):
    "Add ``value(s)`` to set ``name``"
    return self.execute_command('SADD', name, *values)
```

如果不使用解包技术，就只能在调用sadd方法时手动逐个写入参数，耗时又费力

### 2.4 合并两个字典

巧妙的利用解包技术，可以简单方便的将两个字典合并到一个新字典中

```python
dic_1 = {'一': 1}
dic_2 = {'二': 2}

dic_3 = {**dic_1, **dic_2}
print(dic_3)    # {'一': 1, '二': 2}
```
