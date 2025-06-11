---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 处理重复数据
order: 48
---

## SQL 处理重复数据

------

有时候，数据表中会存在相同的记录。在获取表中记录时，相较于取得重复记录来说，取得唯一的记录显然更有意义。

我们之前讨论过的 SQL **DISTINCT** 关键字，与 SELECT 语句一起使用可以时，可以达到消除所有重复记录，只返回唯一记录的目的。

## 语法

------

利用 DISTINCT 关键字来消除重复记录的基本语法如下所示：

```
SELECT DISTINCT column1, column2,.....columnN 
FROM table_name
WHERE [condition]
```

示例：

------

考虑 CUSTOMERS 表，表中记录如下所示：

```
+----+----------+-----+-----------+----------+
| ID | NAME     | AGE | ADDRESS   | SALARY   |
+----+----------+-----+-----------+----------+
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 |
|  2 | Khilan   |  25 | Delhi     |  1500.00 |
|  3 | kaushik  |  23 | Kota      |  2000.00 |
|  4 | Chaitali |  25 | Mumbai    |  6500.00 |
|  5 | Hardik   |  27 | Bhopal    |  8500.00 |
|  6 | Komal    |  22 | MP        |  4500.00 |
|  7 | Muffy    |  24 | Indore    | 10000.00 |
+----+----------+-----+-----------+----------+
```

首先，让我们先看一下 SELECT 语句是如何返回重复的薪水记录的：

```
SQL> SELECT SALARY FROM CUSTOMERS
     ORDER BY SALARY;
```

运行上述语句将会得到以下结果，其中 SALARY 为 2000 的记录出现了两次，即来自原始数据表的重复记录：

```
+----------+
| SALARY   |
+----------+
|  1500.00 |
|  2000.00 |
|  2000.00 |
|  4500.00 |
|  6500.00 |
|  8500.00 |
| 10000.00 |
+----------+
```

现在，让我们在上面的 SELECT 查询中使用 DISTINCT 关键字，然后观察将会得到什么结果：

```
SQL> SELECT DISTINCT SALARY FROM CUSTOMERS
     ORDER BY SALARY;
```

上述语句将会产生如下结果，这一再没有任何重复的条目了：

```
+----------+
| SALARY   |
+----------+
|  1500.00 |
|  2000.00 |
|  4500.00 |
|  6500.00 |
|  8500.00 |
| 10000.00 |
+----------+
```
