---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL REPLACE()函数
order: 27
---

## SQL REPLACE()字符串替换函数

------

### 实例

把数据库表 article中的所有title字段里的 ngrok字符串替换成hello。

```
update `article` set title=replace(title,'ngrok','hello');
```

### replace函数定义

```
replace(original-string，search-string，replace-string)
```

### 参数

- original-string： 被搜索的字符串。可为任意长度。
- search-string： 要搜索并被 replace-string 替换的字符串。该字符串的长度不应超过 255 个字节。如果 search-string 是空字符串，则按原样返回原始字符串。
- replace-string： 该字符串用于替换 search-string。可为任意长度。如果 replace-string 是空字符串，则删除出现的所有 search-string。

### 说明

用字符串表达式3替换字符串表达式1中出现的所有字符串表达式2的匹配项。返回新的字符串。
　如果有某个参数为 NULL，此函数返回 NULL。
