---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 9.2 time模块
order: 1
---

# time

time模块是python的标准模块，提供了丰富的处理时间和日期的方法，例如time.time()方法返回当前的时间，单位精确到毫秒，time.strftime()可以返回格式化后便于人们理解的时间字符串。

## 1. 获取当前时间

```python
import time

curr_time = time.time()
print(curr_time)
```

程序输出结果

```text
1573637499.535866
```

这个时间你肯定不认识，这个叫时间戳，是从1970年1月1日午夜开始到现在所经过的时间，精确到了毫秒，但实际使用时，通常只用秒数。

使用在线时间转换网站 https://tool.lu/timestamp/ 可以对这个时间进行转换
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823233925288.jpeg)

## 2. 获取时间元组

前面讲的time()方法虽然可以获取当前时间，但是返回的结果对我们来说不友好，肯本看不出是哪年哪月，使用localtime方法，可以获取可以直接观察的日期信息

```python
import time

localtime = time.localtime(time.time())
print("当前时间是 :", localtime)
```

程序输出结果

```text
当前时间是 : time.struct_time(tm_year=2019, tm_mon=11, tm_mday=13, tm_hour=17, tm_min=39,
tm_sec=5, tm_wday=2, tm_yday=317, tm_isdst=0)
```

你想准确的获取年月日，则可以这样来操作

```python
import time

localtime = time.localtime(time.time())

print('年', localtime.tm_year)
print('月', localtime.tm_mon)
print('日', localtime.tm_mday)
```

程序输出结果

```text
年 2019
月 11
日 13
```

## 3. 格式化时间

虽然时间元组的形式已经相比time()方法直观了很多，但是仍然达不到我们想要的效果，我们平时用得较多的时间是这样的

```text
2019-11-13 17:43:11
```

实现这种时间格式并不难

```python
import time

localtime = time.localtime(time.time())
localtime = time.strftime("%Y-%m-%d %H:%M:%S", localtime)
print(localtime)
```

使用time模块的strftime方法，就可以将时间转换为指定的格式，这种格式使我们中国人喜欢用的，下面是外国人喜欢用的时间格式转换方法

```python
import time

localtime = time.localtime(time.time())
# 格式化成Wed Nov 13 17:45:58 2019形式
print(time.strftime("%a %b %d %H:%M:%S %Y", localtime))
```

strftime方法返回的是字符串，能将时间戳转换成字符串，也一定能够从字符串转换成时间戳

```python
import time

a = '2019-11-13 17:43:11'
a = time.mktime(time.strptime(a,"%Y-%m-%d %H:%M:%S"))
print(a)
```

程序输出结果为1573638191.0

## 4. 日期格式化符号

你应该已经注意到了，不论是从时间戳到字符串，还是字符串到时间戳，在日期转换过程中，都用到了类似"%Y-%m-%d %H:%M:%S" 这种的字符串，他们规定了以什么样的方式进行转换

- %y 两位数的年份表示（00-99）
- %Y 四位数的年份表示（000-9999）
- %m 月份（01-12）
- %d 月内中的一天（0-31）
- %H 24小时制小时数（0-23）
- %I 12小时制小时数（01-12）
- %M 分钟数（00=59）
- %S 秒（00-59）
- %a 本地简化星期名称
- %A 本地完整星期名称
- %b 本地简化的月份名称
- %B 本地完整的月份名称
- %c 本地相应的日期表示和时间表示
- %j 年内的一天（001-366）
- %p 本地A.M.或P.M.的等价符
- %U 一年中的星期数（00-53）星期天为星期的开始
- %w 星期（0-6），星期天为星期的开始
- %W 一年中的星期数（00-53）星期一为星期的开始
- %x 本地相应的日期表示
- %X 本地相应的时间表示
- %Z 当前时区的名称
- %% %号本身

工作中使用最频繁的，当属 "%Y-%m-%d %H:%M:%S"
