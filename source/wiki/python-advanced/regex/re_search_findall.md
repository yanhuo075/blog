---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 2.5 查找
order: 4
---

# python正则表达式精讲---search与findall

## 1. search的一般用法

search方法扫描整个字符串，返回第一个成功的匹配。从方法功能描述中，我们可以得出以下结论:

1. 即便有多个匹配，search方法也只会返回第一个
2. 匹配的位置可以是任何位置，这一点与match不同

假设目标字符串是这样的

```text
我有3个电话号，分别是13343454523， 13341154523，13341152223
```

使用正则表达式来提取电话号

```python
import re

text = '我有3个电话号，分别是13343454523， 13341154523，13341152223'
pattern = re.compile('(\d{11})')
res = pattern.search(text)

print(res.span())
print(res.groups())
```

程序输出结果

```text
(11, 22)
('13343454523',)
```

## 2. 搜索范围

search 只会返回第一个成功的匹配，那么问题来了，如果字符串里有多处匹配，就像上面的例子中，有3个电话号，想要全部提取，该怎么办呢？

search方法，允许你设置搜索的范围，提供一个开始的位置和一个结束的位置，默认是从索引0开始搜索，想要获取全部的匹配，则需要使用一个循环，上一次匹配的结束位置作为下一次匹配的开始位置，这样，就能返回全部的匹配,示例代码如下

```python
import re

text = '我有3个电话号，分别是13343454523， 13341154523，13341152223'
pattern = re.compile('(\d{11})')
res = pattern.search(text)

lst = []
while res:
    start, end = res.span()
    lst.append(res.group(1))
    res = pattern.search(text, start+1)

print(lst)
```

程序输出结果

```text
['13343454523', '13341154523', '13341152223']
```

## 3. findall

findall在字符串中找到所有正则表达式匹配的子串，并返回一个列表，如果没有找到，则返回空列表。

search方法可以通过设置搜索范围搜索所有匹配的子串，findall可以实现相同的功能，却不必一次次设置搜索范围

```python
import re

text = '我有3个电话号，分别是13343454523， 13341154523，13341152223'
pattern = re.compile('(\d{11})')
res = pattern.findall(text)
print(res)
```

程序输出结果

```text
['13343454523', '13341154523', '13341152223']
```

## 4. finditer

与findall的功能类似，不同之处在于，finditer返回的是一个迭代器

```python
import re

text = '我有3个电话号，分别是13343454523， 13341154523，13341152223'
pattern = re.compile('(\d{11})')
res = pattern.finditer(text)
for item in res:
    print(item.group())
```

程序输出结果

```text
13343454523
13341154523
13341152223
```
