---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 9.6 时间练习题
order: 5
---

[![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234101944.png)](https://item.taobao.com/item.htm?id=747102913485)

# 时间处理练习题

本篇教程的python时间处理练习题，包括输出日期信息, 统计时间间隔, 锻炼使用strftime, isoweekday等方法，通过练习题可以快速掌握time模块和datetime模块。

## 1. 输出今天的信息

### 1.1 题目分析

按照下面的格式，输出今天的时间信息

```text
今天是2019年4月18日，星期四，今年的第108天，这一年29.59%的时间已流逝
```

### 1.2 思路分析

对日期的操作，使用datetime模块

```python
today = datetime.datetime.now()
```

today存储了今天的日期信息，包括年月日，时分秒。

today.isoweekday() 返回的是数字1到7，对应周一到周日。

计算距离今年第一天的天数方法如下

```python
days = int(today.strftime('%j'))
```

计算时间流逝的百分比，需要计算出今年一共有多少天，如果是闰年，是366天，本练习题并不复杂，考察你对datetime模块的熟练程度

### 1.3 示例代码

```python
import datetime

out_put_str = "今天是{date_str}，{weekday}，今年的第{days}天，这一年{pass_ratio}%的时间已流逝"
year_days = 365   # 一年有365天

today = datetime.datetime.now()
date_str = '{year}年{month}月{day}日'.format(year=today.year, month=today.month, day=today.day)

year = today.year
# 判断是否为闰年，闰年的条件： 能被100乘除时，如果可以被400乘除，那么是闰年，不能被100乘除，能被4整除是闰年
b_runnian = False
if year % 100 == 0:
    if year % 400 == 0:
        b_runnian = True
elif year % 4 == 0:
    b_runnian = True

if b_runnian:
    year_days = 366


# 今年的第几天
days = int(today.strftime('%j'))



# 数字星期与中文星期的映射关系
week_map = {
    1: '星期一',
    2: '星期二',
    3: '星期三',
    4: '星期四',
    5: '星期五',
    6: '星期六',
    7: '星期日',
}

# 星期几
week_day = week_map[today.isoweekday()]
# 已经过去了多少
pass_ratio = round((days / year_days)*100, 2)
out_put = out_put_str.format(date_str=date_str, weekday=week_day, days=days, pass_ratio=pass_ratio)

print(out_put)
```

## 2. 统计日期间隔

### 2.1 题目要求

需要编写一个函数get_day_diff(date_lst, target)， 入参示例如下

```text
date_lst = [
                '2019-01-01', '2019-01-15',
                '2019-01-30', '2019-02-01',
                '2019-02-05', '2019-02-15',
                '2019-03-06', '2019-03-15',
                '2019-04-01', '2019-04-05',
                '2019-04-13', '2019-04-30',
                '2019-05-05', '2019-05-06'
                ]
target = '2019-05-08'
```

函数计算date_lst里的日期与target的间隔天数，然后统计这些天数信息，最后返回的结果示例如下

```text
{'7_days': 2, '30_days': 4, '90_days': 9, '180_days': 14}
```

7_days 表示时间间隔小于7天的日期个数

### 2.2 思路分析

不可能直接用字符串计算日期的间隔，需要将这些字符串转成datetime类型，这样才能计算两个日期的间隔

最终的结果需要用字典来保存，因此函数里需要初始化一个字典

```python
info = {
        '7_days': 0,
        '30_days': 0,
        '90_days': 0,
        '180_days':0
    }
```

用列表里的日期和target求间隔，然后做统计，如果间隔天数小于等于7天，则info['7_days'] += 1, 需要注意的地方是，一共有4个条件判断，而且这些条件判断之间不是互斥的关系，不能使用if else 这种逻辑判断，只需要4个if判断即可

### 2.3 示例代码

```python
import datetime


def get_day_diff(date_lst, target):
    info = {
        '7_days': 0,
        '30_days': 0,
        '90_days': 0,
        '180_days':0
    }

    target_date = datetime.datetime.strptime(target, '%Y-%m-%d')
    for date_str in date_lst:
        date = datetime.datetime.strptime(date_str, '%Y-%m-%d')
        day_diff = (target_date - date).days

        if day_diff <= 180:
            info['180_days'] += 1

        if day_diff <= 90:
            info['90_days'] += 1

        if day_diff <= 30:
            info['30_days'] += 1

        if day_diff <= 7:
            info['7_days'] += 1

    return info


if __name__ == '__main__':
    date_lst = [
                '2019-01-01', '2019-01-15',
                '2019-01-30', '2019-02-01',
                '2019-02-05', '2019-02-15',
                '2019-03-06', '2019-03-15',
                '2019-04-01', '2019-04-05',
                '2019-04-13', '2019-04-30',
                '2019-05-05', '2019-05-06'
                ]

    info = get_day_diff(date_lst, '2019-05-08')
    print(info)
```
