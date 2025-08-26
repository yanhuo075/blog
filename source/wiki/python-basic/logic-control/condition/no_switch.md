---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.7 为什么python没有switch/case
order: 6
---

# 为什么python没有switch/case

不同于其他编程语言，python中是没有switch/case 这种语法的，如果你是从其他语言转到python的，期初，对于没有switch/case 是很不适应的，不过这并不影响你编程，因为if ... elif ... else 完全可以替代switch/case， 尽管写起来不那么舒服

## 1. if 条件语句替代 switch/case

```python
def get_score_by_course(course):
    """
    根据课程获取考试分数
    :param course:
    :return:
    """
    if course == 'mathematical':
        return 90
    elif course == 'english':
        return 95
    elif course == 'history':
        return 98
    else:
        return 0


print(get_score_by_course('english'))
```

## 2. 字典替代

除了使用if 条件语句，字典同样可以实现switch/case 的功能

```python
course_dict = {
    'mathematical': 90,
    'english': 95,
    'history': 98
}


def get_score_by_course(course):
    """
    根据课程获取考试分数
    :param course:
    :return:
    """
    return course_dict.get(course, 0)


print(get_score_by_course('english'))
```

使用字典，在课程和分数之间建立起映射关系，获取分数的函数一行代码就实现了if条件语句8行代码的功能，更加的简洁。

使用字典时，还可以用函数做value，使其扩展性更好。

```python
def get_mathematical_score():
    return 90


def get_english_score():
    return 95


def get_history_score():
    return 98

course_dict = {
    'mathematical': get_mathematical_score,
    'english': get_english_score,
    'history': get_history_score
}


def get_score_by_course(course):
    """
    根据课程获取考试分数
    :param course:
    :return:
    """
    func = course_dict.get(course, lambda : 0)
    return func()


print(get_score_by_course('english'))
```

## 3. 通过关键字寻找对应函数

这样的设计，当增加一个课程时，必须修改course_dict，否则将无法获得对应的获取分数的函数，面对这种业务场景，有一种可以免去字典映射的方法，通过globals()函数获得全局变量，然后通过函数名称找到对应的函数

```python
def get_mathematical_score():
    return 90


def get_english_score():
    return 95


def get_history_score():
    return 98


def get_score_by_course(course):
    """
    根据课程获取考试分数
    :param course:
    :return:
    """
    global_dict = globals()
    func_name = 'get_{course}_score'.format(course=course)
    func = global_dict.get(func_name, lambda : 0)
    return func()


print(get_score_by_course('history'))
```

## 4. 更加直观的映射关系

上面的方法虽然免去了字典映射，看上去简化了代码，但是却带来了新的问题，get_score_by_course函数里的代码总给人一种云里雾里的印象，if条件语句替换switch/case的方法虽然写起来繁琐，但course和处理方法之间的映射关系非常明确，使用字典时，同样可以表达这种明确的映射关系。

而通过关键字直接查找对应的函数这种方式下，course与处理函数之间的映射关系变得隐晦起来，除非你仔细分析代码，否则，很难找到这种映射关系。

我们需要一种无需维护字典，无需繁琐的if 条件语句，同时又能表达清晰简明的映射关系的方法

```python
def func_dispatch(func):
    registry = {}

    def dispatch(key_word):
        return registry.get(key_word, registry[object])

    def register(key_word, func=None):
        if func is None:
            return lambda f: register(key_word, f)

        registry[key_word] = func
        return func

    def wrapper(*args, **kw):
        return dispatch(args[0])(*args, **kw)

    registry[object] = func
    wrapper.register = register
    return wrapper


@func_dispatch
def score_dispath(course):
    return 0


@score_dispath.register('mathematical')
def get_mathematical_score(course):
    return 90


@score_dispath.register('english')
def get_english_score(course):
    return 95


@score_dispath.register('history')
def get_history_score(course):
    return 98


def get_score_by_course(course):
    """
    根据课程获取考试分数
    :param course:
    :return:
    """
    return score_dispath(course)


print(get_score_by_course('mathematical'))
```

func_dispatch 是一个装饰器，先用这个装饰器去装饰score_dispath，score_dispath就变成了可以分发函数的路由器，由它再去装饰实际处理业务的函数，注册的过程建立起关键字和函数之间的映射关系，这样，既没有if条件语句的繁琐，也没有维护字典的琐碎，却保留了映射关系的明确，至于这个func_dispatch装饰器，你实在理解不了也没有关系，只要会使用就可以了，使用时要注意，那些被注册的业务函数的参数必须和score_dispath保持一致，此外，你不需要再处理额外的事情。
