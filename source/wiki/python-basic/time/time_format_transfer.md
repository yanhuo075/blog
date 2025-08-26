---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 9.5 时间格式转换
order: 4
---

# datetime, 时间戳，字符串，time，各种时间格式的转换

本文介绍python当中表示时间的数据之间如何进行转换，涉及到的数据类型和模块包括datetime, 时间戳，字符串，time，在实践中，我们总能遇到这方面的需求。

## 1. 获得当前时间的时间戳

获得当前时间的时间戳有两种方法

```python
import time
from datetime import datetime

# 获得当前时间戳
current_time = int(time.time())
print(current_time)

now = datetime.now()
current_time = int(now.timestamp())
print(current_time)
```

程序输出结果

```text
1577761708
1577761708
```

## 2. 将时间戳转换为datetime类型数据

```python
from datetime import datetime

current_time = 1577761708
date_time = datetime.fromtimestamp(current_time)
print(date_time)
```

## 3. 时间戳与时间元组之间转换

```python
import time

current_time = 1577761708
print(time.ctime(current_time))         # 转换成 Tue Dec 31 11:08:28 2019
print(time.localtime(current_time))     # 转成时间元组,本地时间
print(time.gmtime(current_time))        # 转成时间元组,国际伦敦时间

localtime_tuple = time.localtime(current_time)      # 将时间元组转成时间戳
current_time = time.mktime(localtime_tuple)
print(current_time)
```

## 4. 字符串与datetime之间互相转换

```python
from datetime import datetime

now = datetime.now()
now_str = now.strftime("%Y-%m-%d %H:%M:%S")     # 将datetime转成字符串
print(now_str)

date_time = datetime.strptime(now_str, "%Y-%m-%d %H:%M:%S")  # 将字符串转成datetime类型
print(date_time, type(date_time))
```

## 5. 字符串与时间元组之间互相转换

```python
import time

time_str = '2019-12-31 11:22:23'
res = time.strptime(time_str, '%Y-%m-%d %H:%M:%S')      # 将字符串转成时间元组
print(res, type(res))

now = time.localtime()
now_str = time.strftime('%Y-%m-%d %H:%M:%S', now)       # 将时间元组转成字符串
print(now_str)
```

## 6. datetime 转成时间元组

```python
from datetime import datetime

now = datetime.now()
print(now.ctime())      # 返回ctime() 格式时间字符串
print(now.timetuple())  # 返回时间元组
```
