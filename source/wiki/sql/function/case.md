---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 字母大小写转换函数
order: 13
---

## SQL 字母大小写转换函数

------

SQL 字母大小写转换函数包含了UPPER(s)、UCASE(s)、LOWER(s)和LCASE(s)函数。

### **1、LOWER(s)函数和LCASE(s)函数**

LOWER(s)或者LCASE(s)函数可以将字符串s中的字母字符全部转换成小写字母。

**　实例：**

使用LOWER函数或者LCASE函数将字符串中所有字母字符转换为小写。SQL语句如下：

```
SELECT LOWER('WWW.ngrok.cn'),LCASE('ngrok.CN');
```

执行结果如下：

### 2、UPPER(s)函数和UCASE(s)函数

UPPER(s)或UCASE(s)函数可以将字符串s中的字母字符全部转换成大写字母。
**　实例：**
　使用UPPER函数或者UCASE函数将字符串中的所有字母字符转换为大写。SQL语句如下：

```
SELECT UPPER('www.ngrok.cn'),UCASE('ngrok');
```
