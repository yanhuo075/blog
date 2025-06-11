---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL TRIM()函数
order: 28
---

## SQL TRIM()函数去除字符串头尾空格

------

SQL 中的 TRIM 函数是用来移除掉一个字串中的字头或字尾。最常见的用途是移除字首或字尾的空白。这个函数在不同的资料库中有不同的名称：

- MySQL: TRIM( ), RTRIM( ), LTRIM( )
- Oracle: RTRIM( ), LTRIM( )
- SQL Server: RTRIM( ), LTRIM( )

各种 trim 函数的语法如下：

- **TRIM ( [ [位置] [要移除的字串] FROM ] 字串)** : [位置] 的可能值为 LEADING (起头), TRAILING (结尾), or BOTH (起头及结尾)。 这个函数将把 [要移除的字串] 从字串的起头、结尾，或是起头及结尾移除。如果我们没有列出 [要移除的字串] 是什么的话，那空白就会被移除。
- **LTRIM(字串)** : 将所有字串起头的空白移除。
- **RTRIM(字串)** : 将所有字串结尾的空白移除。

### 例1 TRIM()

```
SELECT TRIM('   Sample   ');
```

结果:

```
'Sample'
```

### 例2 LTRIM()

```
SELECT LTRIM('   Sample   ');
```

结果:

```
'Sample   '
```

### 例3 RTRIM()

```
SELECT RTRIM('   Sample   ');
```

结果:

```
'   Sample'
```
