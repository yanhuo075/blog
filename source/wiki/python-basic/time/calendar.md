---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 9.2 calendar日历模块
order: 1
---

# 日历模块---calendar

calendar是python的日历模块，它提供了isleap方法可以判断某年是否为闰年, calendar.month()方法返回月历, calendar.weekday()返回日期的日期码，在一些操作计算上比datetime模块更加方便。

## 1. 判断某年是否为闰年

```python
import calendar

print(calendar.isleap(2019))
```

输出结果为False

## 2. 输出某个月的月历

calendar.month以多行字符串的方式返回某个月的月历

```python
import calendar

cal = calendar.month(2019, 11)
print(cal)
```

输出结果

```text
   November 2019
Mo Tu We Th Fr Sa Su
             1  2  3
 4  5  6  7  8  9 10
11 12 13 14 15 16 17
18 19 20 21 22 23 24
25 26 27 28 29 30
```

这样的结果虽然观察起来很直观，但由于返回的是字符串，想要进行一些数值计算则不行，需要使用monthcalendar方法

```python
import calendar

cal = calendar.monthcalendar(2019, 11)
print(cal)
```

输出结果

```text
[
[0, 0, 0, 0, 1, 2, 3],
[4, 5, 6, 7, 8, 9, 10],
[11, 12, 13, 14, 15, 16, 17],
[18, 19, 20, 21, 22, 23, 24],
[25, 26, 27, 28, 29, 30, 0]
]
```

## 3.获取指定日期的日期码

```python
import calendar

cal = calendar.weekday(2019, 11, 13)
print(cal)
```

输出结果是2

11月13日是周3，返回结果是2，是因为这个日期码是从0开始的，周一返回0，周日返回6
