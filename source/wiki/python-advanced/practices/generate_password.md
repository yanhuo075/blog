---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.4 制作密码生成器
order: 3
---

# python实战练手项目---制作密码生成器

本文将带你使用python来实现一个密码生成器，目的是通过程序来生成一个非常随机的密码，随机混乱的密码会更好的保护你的私密信息，增加破解的难度，当然，记忆他们也是一件困难的事情。

在动手写代码之前，首先要明确密码生成的规则，这玩意是比代码更重要的事情，如果密码生成规则都不清楚，又该如何编写代码呢？密码生成规则并没有什么成文的要求，结合大部分网站的密码规则总结如下：

1. 密码长度不能小于6
2. 密码里可以包含英文字母，数字，符号
3. 至少包含一个大写字母
4. 至少包含一个特殊符号

## 1. 随机生成若干个大写字母

规则要求至少包含一个大写字母，我实现一个函数，生成大写字母，至于一次生成几个大写字母，则使用random.randint(1, 3)来随机控制

```python
def get_upper():
    count = random.randint(1, 3)
    return random.choices('ABCDEFGHIJKLMNOPQRSTUVWXYZ', k=count)
```

## 2. 随机生成若干个特殊符号

与生成大写字母的方式相同，最多生成3个特殊符号，最少生成1个特殊符号

```python
def get_special_char():
    '''
    生成特殊符号
    :return:
    '''
    count = random.randint(1, 3)
    return random.choices('!@$%^&*()_+~', k=count)
```

## 3. 生成小写字母和数字

小写字母和数字在一起生成

```text
def get_lower(count):
    '''
    生成小写字母和数字
    :param count: 
    :return: 
    '''
    string = 'abcdefghijklmnopqrstuvwxyz0123456789'
    return random.choices(string, k=count)
```

## 4. 主函数generate_password

主函数负责调用前面已经实现的函数

```python
def generate_password(length):
    '''
    生成指定长度的密码
    :param length:
    :return:
    '''

    if length < 6:
        length = 6

    lst = []
    upper_lst = get_upper()     # 大写
    special_char = get_special_char()      # 特殊字符
    lst.extend(upper_lst)
    lst.extend(special_char)

    surplus_count = length - len(lst)
    lower_lst = get_lower(surplus_count)
    lst.extend(lower_lst)
    # 将顺序打乱
    random.shuffle(lst)
    return ''.join(lst)
```

## 5. 全部示例代码

```python
import random


def get_upper():
    '''
    生成大写字母
    :return:
    '''
    count = random.randint(1, 3)
    return random.choices('ABCDEFGHIJKLMNOPQRSTUVWXYZ', k=count)


def get_special_char():
    '''
    生成特殊符号
    :return:
    '''
    count = random.randint(1, 3)
    return random.choices('!@$%^&*()_+~', k=count)


def get_lower(count):
    '''
    生成小写字母和数字
    :param count:
    :return:
    '''
    string = 'abcdefghijklmnopqrstuvwxyz0123456789'
    return random.choices(string, k=count)


def generate_password(length):
    '''
    生成指定长度的密码
    :param length:
    :return:
    '''

    if length < 6:
        length = 6

    lst = []
    upper_lst = get_upper()     # 大写
    special_char = get_special_char()      # 特殊字符
    lst.extend(upper_lst)
    lst.extend(special_char)

    surplus_count = length - len(lst)
    lower_lst = get_lower(surplus_count)
    lst.extend(lower_lst)
    # 将顺序打乱
    random.shuffle(lst)
    return ''.join(lst)


if __name__ == '__main__':
    print(generate_password(8))
    print(generate_password(5))
    print(generate_password(12))
```
