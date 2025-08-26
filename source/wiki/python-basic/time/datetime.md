---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 9.3 datetime模块
order: 2
---

# python时间日期处理模块datetime详解

## 1. datetime模块

datetime是python中处理日期时间的标准库，datetime模块中常用的类包括date, time, datetime, timedelta，使用这些对象支持日期时间的数学运算和更有效的解析其属性用于格式化输出。

下面，逐个介绍这4个时间处理类

## 2. datetime.date

datetime.date 一个理想化的简单型日期是一个理想化的简单型日期，它假设当今的公历在过去和未来永远有效，主要用于处理年月日期时间。

使用指定的年月日，可以快速构造一个date对象

```python
from datetime import date

date_obj = date(2022, 5, 29)

print(date_obj.year)        # 2022
print(date_obj.month)       # 5
print(date_obj.day)         # 29
```

datetime.date只支持到年月日，不能处理小时分钟秒，下表是date类的常用方法和解释

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [date.today](http://www.coolpython.net/method_topic/date/today.html) | 返回当前的本地日期                                           |
| [date.fromtimestamp](http://www.coolpython.net/method_topic/date/fromtimestamp.html) | 返回对应于 POSIX 时间戳的当地时间，类型是datetime.date       |
| [date.fromisoformat](http://www.coolpython.net/method_topic/date/fromisoformat.html) | 根据传入的date_string参数返回对应的date对象，date_string字符串必须满足YYYY-MM-DD的时间格式 |
| [date.fromisocalendar](http://www.coolpython.net/method_topic/date/fromisocalendar.html) | 返回指定 year, week 和 day 所对应 ISO 历法日期的 date        |
| [date.isocalendar](http://www.coolpython.net/method_topic/date/isocalendar.html) | 返回一个由三部分组成的 named tuple 对象                      |
| [date.isoformat](http://www.coolpython.net/method_topic/date/isoformat.html) | 返回一个符合 YYYY-MM-DD 格式的字符串，是date.fromisoformat() 的逆操作 |
| [date.isoweekday](http://www.coolpython.net/method_topic/date/isoweekday.html) | 返回一个整数代表星期几，1代表星期一，7代表星期日             |
| [date.weekday](http://www.coolpython.net/method_topic/date/weekday.html) | 返回一个整数代表星期几，0表示星期一，6表示星期日             |
| [date.replace](http://www.coolpython.net/method_topic/date/replace.html) | 根据传入的实参返回一个新的date对象，如果没有传入任何实参，则返回一个原date对象相同的日期 |
| [date.strftime](http://www.coolpython.net/method_topic/date/strftime.html) | 根据format参数返回指定格式的日期字符串                       |
| [date.timetuple](http://www.coolpython.net/method_topic/date/timetuple.html) | 返回一个 time.struct_time对象，和time.localtime() 所返回的类型一致 |

## 3. datetime.datetime

date对象只能处理年月日，而datetime.datetime对象可以处理到时分秒，拥有更高的时间精度，datetime与date一样假定当前的格列高利历向前后两个方向无限延伸，同时像time对象一样假定每一天恰好有 3600*24 秒。

**datetime构造函数**

```python
datetime.datetime(year, month, day, hour=0, minute=0, 
second=0, microsecond=0, tzinfo=None, *, fold=0)
```

创建一个datetime对象，并输出它的各种属性

```python
from datetime import datetime

date_time = datetime(year=2022, month=6, day=2, hour=20, minute=23, second=30)
print(date_time.year)               # 年
print(date_time.month)              # 月
print(date_time.day)                # 日
print(date_time.hour)               # 小时
print(date_time.minute)             # 分钟
print(date_time.second)             # 秒
print(date_time.microsecond)        # 毫秒
```

下表是datetime对象常用的方法

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [datetime.now](http://www.coolpython.net/method_topic/datetime/now.html) | 返回当前时间                                                 |
| [datetime.utcnow](http://www.coolpython.net/method_topic/datetime/utcnow.html) | 返回当前UTC时间                                              |
| [datetime.fromtimestamp](http://www.coolpython.net/method_topic/datetime/fromtimestamp.html) | 返回POSIX 时间戳对应的本地日期和时间                         |
| [datetime.utcfromtimestamp](http://www.coolpython.net/method_topic/datetime/utcfromtimestamp.html) | 返回POSIX 时间戳的对应的UTC时间                              |
| [datetime.timestamp](http://www.coolpython.net/method_topic/datetime/timestamp.html) | 返回对应于datetime实例的 POSIX 时间戳                        |
| [datetime.timetuple](http://www.coolpython.net/method_topic/datetime/timetuple.html) | 返回一个 time.struct_time对象                                |
| [datetime.replace](http://www.coolpython.net/method_topic/datetime/replace.html) | 返回一个具有同样属性值的 datetime                            |
| [datetime.strftime](http://www.coolpython.net/method_topic/datetime/strftime.html) | 将datetime对象转为指定格式的字符串                           |
| [datetime.strptime](http://www.coolpython.net/method_topic/datetime/strptime.html) | 将字符串转为datetime对象，转换时要求字符串内容符合指定的格式 |
| [datetime.fromisocalendar](http://www.coolpython.net/method_topic/datetime/fromisocalendar.html) | 返回以 year, week 和 day值指明的ISO历法日期所对应的 datetime |
| [datetime.fromisoformat](http://www.coolpython.net/method_topic/datetime/fromisoformat.html) | 将符合isoformat时间格式的字符串转为datetime对象              |
